# Lab 11 (Annex D, Lab 2) – Exchange 2019 Post-Deployment Configuration: CLI Edition

> CLI rewrite of `ocr-Lab-11-gui.pdf`. It reaches the same end state using PowerShell and the Exchange Management Shell (EMS) in place of EAC and DNS Manager clicks.

## Gist / End Goal
When you finish, the environment has:
- **Two Exchange 2019 mailbox servers** (`NXXMEUEX01`, `NXXMEUEX02`) with databases renamed to **MBDB01 / MBDB02**. The DB and log files are moved off the system drive to **F:**.
- An **Internet Send Connector** (`*`, scoped) that uses both servers as the source.
- **DNS**: conditional forwarders and reverse zones for distant sites, plus A records (autodiscover / webmail / owa / mail), **MX** and **autodiscover SRV** records with round robin across both servers.
- **Outlook Anywhere** and **OWA/ECP virtual directory URLs** set on both servers.
- A **DAG** (`XXMEUDAG`) with `NXXMEUUTIL` as the witness and both servers as members. Each database has a passive copy on the other server.

## Conventions
> New to PowerShell variables, pipelines or loops? Read [[concepts-primer]] first.

- Replace `XX` with your unit number and `XXX.YYY` with real IP octets.
- Run Exchange cmdlets in **EMS as Administrator**. Your account must be in **Organization Management**.
- Run DNS cmdlets on the **DC** (or remotely with `-ComputerName XXMEUDC01`).

Paste this block first in every shell you open, so the commands below work unchanged:
```powershell
$Dom   = 'XXMEU.USMC.MIL'
$EX1   = 'NXXMEUEX01'; $EX1ip = '10.154.XXX.YYY'
$EX2   = 'NXXMEUEX02'; $EX2ip = '10.152.XXX.YYY'
$DC    = 'XXMEUDC01'
$WIT   = 'NXXMEUUTIL'
$DAG   = 'XXMEUDAG'
$Exch  = $EX1, $EX2
```

> **Speed tip:** Installing Exchange on EX02 (Task 2) is the slowest step. Start it **first**, then do the DNS work (Task 4) while it runs.

---

## Task 1 + 2: Finish EX02, Rename Databases, Move DB and Logs to F:

**1. Install Exchange on EX02 unattended.** Run this on EX02 from the mounted Exchange ISO, after the same prerequisites as Lab 1:
```cmd
D:\Setup.exe /Mode:Install /Roles:Mailbox /IAcceptExchangeServerLicenseTerms_DiagnosticDataOFF
```
On older CUs (before CU11), use `/IAcceptExchangeServerLicenseTerms` instead. Restart when setup finishes. You can ignore the asterisk warning the GUI mentions.

**2. Create the folder structure on both servers in one shot.** DAG copies need **identical paths on every server**, so create all four folders on both:
```powershell
Invoke-Command -ComputerName $Exch {
  'F:\Exchange Database Files\MBDB01','F:\Exchange Database Files\MBDB02',
  'F:\Exchange Log Files\MBDB01','F:\Exchange Log Files\MBDB02' |
    ForEach-Object { New-Item -ItemType Directory -Path $_ -Force | Out-Null }
}
```

**3. Rename the default databases:**
```powershell
Get-MailboxDatabase -Server $EX1 | Set-MailboxDatabase -Name MBDB01
Get-MailboxDatabase -Server $EX2 | Set-MailboxDatabase -Name MBDB02
Get-MailboxDatabase | ft Name,Server,EdbFilePath,LogFolderPath   # confirm
```

**4. Move the DB and logs together** (one command per DB, no prompts). Each DB is briefly dismounted, so outside a lab, do this only during setup or a maintenance window.
```powershell
foreach ($db in 'MBDB01','MBDB02') {
  Move-DatabasePath $db `
    -EdbFilePath   "F:\Exchange Database Files\$db\$db.edb" `
    -LogFolderPath "F:\Exchange Log Files\$db" `
    -Confirm:$false -Force
}
Get-MailboxDatabase -Status | ft Name,Mounted,EdbFilePath,LogFolderPath
```
Paths that contain spaces must be quoted.

---

## Task 3: Internet Send Connector
```powershell
New-SendConnector -Name 'To Internet' -Internet -AddressSpaces '*' `
  -SourceTransportServer $EX1,$EX2 -IsScopedConnector $true
Get-SendConnector 'To Internet' | fl Name,AddressSpaces,SourceTransportServers,IsScopedConnector
```
The PDF's example omits the quotes around `To Internet`, so it fails as written.

**Capstone variant** (the connector already exists, so only add the new server as a source):
```powershell
Set-SendConnector 'To Internet' -SourceTransportServers @{Add='NEWEXSERVER'}
```

---

## Task 4: DNS (run on XXMEUDC01)

**1. Conditional forwarders and reverse zones** for each distant site (neighbors and instructor). You can't have both a stub zone and a conditional forwarder for the same domain.
```powershell
# One line per distant domain
Add-DnsServerConditionalForwarderZone -Name 'YYMEU.USMC.MIL' -MasterServers 10.x.x.x -ReplicationScope Forest
# Reverse lookup zone for each distant network
Add-DnsServerPrimaryZone -NetworkId '10.x.x.0/24' -ReplicationScope Forest
Get-DnsServerZone | ft ZoneName,ZoneType,IsReverseLookupZone
```

**2. Host records.** Each name gets both server IPs, which gives you round robin:
```powershell
# Server host records (normally already exist): add with PTRs only if missing
Add-DnsServerResourceRecordA -ZoneName $Dom -Name $EX1 -IPv4Address $EX1ip -CreatePtr -EA SilentlyContinue
Add-DnsServerResourceRecordA -ZoneName $Dom -Name $EX2 -IPv4Address $EX2ip -CreatePtr -EA SilentlyContinue

foreach ($n in 'autodiscover','webmail','owa','mail') {
  Add-DnsServerResourceRecordA -ZoneName $Dom -Name $n -IPv4Address $EX1ip,$EX2ip
}
```

**3. MX and autodiscover SRV records** (one per mailbox server):
```powershell
foreach ($s in $Exch) {
  Add-DnsServerResourceRecordMX -ZoneName $Dom -Name '.' -MailExchange "$s.$Dom" -Preference 10
  Add-DnsServerResourceRecord -Srv -ZoneName $Dom -Name '_autodiscover._tcp' `
    -DomainName "$s.$Dom" -Priority 0 -Weight 0 -Port 443
}
```

**4. Verify round robin.** The order of the returned IPs should rotate on each lookup:
```powershell
1..3 | ForEach-Object { nslookup "mail.$Dom" }
Resolve-DnsName -Type MX $Dom; Resolve-DnsName -Type SRV "_autodiscover._tcp.$Dom"
```
For mail to flow between sites, **both** sides need conditional forwarders pointing at each other and their own MX record.

---

## Task 5: Outlook Anywhere and Virtual Directory URLs
Each `Get-*` cmdlet below returns objects for **both** servers, so one pipeline configures both.

**1. Outlook Anywhere.** Set the external host to OWA and the internal host to WEBMAIL:
```powershell
Get-OutlookAnywhere | Set-OutlookAnywhere `
  -ExternalHostname "owa.$Dom"     -ExternalClientsRequireSsl $true `
  -InternalHostname "webmail.$Dom" -InternalClientsRequireSsl $true
```

**2. ECP and OWA URLs and authentication.** ECP gets no external URL. ECP and OWA must use **matching** authentication, which is the "relationship" pop-up in the GUI:
```powershell
Get-EcpVirtualDirectory | Set-EcpVirtualDirectory `
  -InternalUrl "https://webmail.$Dom/ecp" -ExternalUrl $null `
  -FormsAuthentication $false -WindowsAuthentication $true

Get-OwaVirtualDirectory | Set-OwaVirtualDirectory `
  -InternalUrl "https://webmail.$Dom/owa" -ExternalUrl "https://owa.$Dom/owa" `
  -FormsAuthentication $false -WindowsAuthentication $true

Invoke-Command -ComputerName $Exch { iisreset /noforce }
```

**3. Verify:**
```powershell
Get-OutlookAnywhere     | ft Server,InternalHostname,ExternalHostname
Get-EcpVirtualDirectory | ft Server,InternalUrl,ExternalUrl,FormsAuthentication,WindowsAuthentication
Get-OwaVirtualDirectory | ft Server,InternalUrl,ExternalUrl,FormsAuthentication,WindowsAuthentication
```

---

## Task 6: Database Availability Group

**1. Confirm your permissions:**
```powershell
Get-RoleGroupMember 'Organization Management'
```

**2. Prepare the witness (NXXMEUUTIL)** remotely, with no local login needed:
```powershell
Invoke-Command -ComputerName $WIT {
  Install-WindowsFeature FS-FileServer | Out-Null
  Add-LocalGroupMember -Group Administrators -Member 'XXMEU\Exchange Trusted Subsystem','XXMEU\<YourAdminAccount>' -EA SilentlyContinue
  Get-LocalGroupMember Administrators
}
```

**3. Create the DAG and add the members.** The witness must be a hostname or FQDN (not an IP) and can't be a DAG member. If you omit the witness directory, it defaults to `%SystemDrive%\DAGFileShareWitnesses\<DAG FQDN>`.
```powershell
New-DatabaseAvailabilityGroup -Name $DAG -WitnessServer $WIT
foreach ($s in $Exch) { Add-DatabaseAvailabilityGroupServer -Identity $DAG -MailboxServer $s }
Get-DatabaseAvailabilityGroup $DAG -Status | fl Name,Servers,WitnessServer,OperationalServers
```
Add the servers one at a time, which is what the loop does; this takes a few minutes while the failover cluster forms. If it errors, restart both Exchange servers and rerun the `foreach` line.

**4. Add the passive database copies.** Each DB gets a copy on the other server, and seeding starts automatically:
```powershell
Add-MailboxDatabaseCopy -Identity MBDB01 -MailboxServer $EX2 -ActivationPreference 2
Add-MailboxDatabaseCopy -Identity MBDB02 -MailboxServer $EX1 -ActivationPreference 2
```
On low-bandwidth links, add `-SeedingPostponed` and seed later with `Update-MailboxDatabaseCopy`.

**5. Verify the copies, and fix a bad one** (this replaces the GUI's "Activate"):
```powershell
Get-MailboxDatabaseCopyStatus * | ft Name,Status,CopyQueueLength,ReplayQueueLength,ContentIndexState
Get-MailboxDatabase | ft Name,Servers                     # both servers should be listed

# A copy that is Suspended or Failed:
Resume-MailboxDatabaseCopy  -Identity "MBDB01\$EX2"
# Still bad? Reseed it from scratch:
Update-MailboxDatabaseCopy  -Identity "MBDB01\$EX2" -DeleteExistingFiles -Confirm:$false
```
Healthy end state: the active copies show **Mounted**, the passive copies show **Healthy**, and the queues are at or near 0.

---

**Lab complete.** Proceed to Lab 3.
