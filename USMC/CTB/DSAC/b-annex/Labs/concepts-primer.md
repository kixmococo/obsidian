# Concepts Primer: PowerShell for the CLI Labs

> Companion to [[cli-Lab-11-gui]]. It explains the PowerShell concepts the CLI guide uses that the GUI guide never needed.

---

## 1. What "Conventions" Means
The Conventions section sets the ground rules the rest of the guide assumes, so every command reads and behaves the same way.

1. **Placeholders:** `XX` is your unit number and `XXX.YYY` stands for real IP octets. The original PDF works the same way (`XXMEU`, `10.154.XXX.YYY`).
2. **Where to run things:** Exchange commands go in the **Exchange Management Shell (EMS)**, a PowerShell window with the Exchange commands loaded. DNS commands go on the **domain controller**. A command run in the wrong place gives "command not found" or acts on the wrong server.
3. **Permissions:** your account must be in **Organization Management**, or the DAG steps will fail.

---

## 2. Variables (`$Dom`, `$EX1`, etc.)
In PowerShell, anything starting with `$` is a **variable**: a named box that holds a value.

```powershell
$Dom = 'XXMEU.USMC.MIL'
```
This means: store the text `XXMEU.USMC.MIL` in a box called `$Dom`. From then on, wherever you write `$Dom`, PowerShell substitutes that text.

| Variable | Holds | Used for |
|---|---|---|
| `$Dom` | your domain name | DNS zone names, FQDNs, URLs |
| `$EX1` / `$EX2` | Exchange server names | targeting each server |
| `$EX1ip` / `$EX2ip` | their IP addresses | DNS A records |
| `$DC` | domain controller name | DNS work |
| `$WIT` | witness server name | DAG creation |
| `$DAG` | DAG name | DAG commands |
| `$Exch` | **both** Exchange servers as a list | running one command against both |

### Why use variables?
- **Type it once, use it everywhere.** Your domain appears dozens of times in this lab. Without variables, you'd retype `XXMEU.USMC.MIL` in every command, and one typo (like the PDF's own "AUTODOSCOVER") breaks DNS or a URL.
- **Easy to reuse.** A student in a different unit changes only the top block, and every command below it still works.
- **Readable.** `"https://owa.$Dom/owa"` reads as "the OWA URL for my domain" and expands to `https://owa.XXMEU.USMC.MIL/owa`.

### Catches
- **Variables only live in the window you set them in.** Close the shell or open a new one, and they're gone. That's why the guide says to paste the block into every shell you open.
- **Quote type matters.** Double quotes (`"owa.$Dom"`) fill the variable in. Single quotes (`'owa.$Dom'`) keep the text literally as `owa.$Dom`. That's why URLs use double quotes and fixed text uses single quotes.

### Lists
`$Exch = $EX1, $EX2` uses a comma to make a **list** of two names. Commands that accept several servers, like `Invoke-Command -ComputerName $Exch`, then hit both at once.

---

## 3. The Pipeline `|`
The pipeline passes the output of one command into the next.

```powershell
Get-OwaVirtualDirectory | Set-OwaVirtualDirectory -InternalUrl ...
```
The first command finds the OWA directories on **both** servers, and the second changes all of them. This is how one line replaces clicking through each server in the EAC.

---

## 4. `foreach` Loops
A `foreach` loop repeats a block for each item in a list.

```powershell
foreach ($s in $Exch) { Add-DatabaseAvailabilityGroupServer -Identity $DAG -MailboxServer $s }
```
This runs once with `$s = NXXMEUEX01` and again with `$s = NXXMEUEX02`. `$s` is a temporary variable for "the current item".

---

## 5. `Invoke-Command -ComputerName`
`Invoke-Command` runs the commands in the `{ }` block **on another machine** over the network (PowerShell Remoting).

```powershell
Invoke-Command -ComputerName $Exch { New-Item -ItemType Directory -Path 'F:\Example' -Force }
```
This is how the guide creates folders on both Exchange servers, or sets up the witness server, without logging into them.

---

## 6. Line Continuation: the Backtick `` ` ``
A backtick at the end of a line means "this command continues on the next line". It only splits long commands for readability.

```powershell
Move-DatabasePath MBDB01 `
  -EdbFilePath 'F:\Exchange Database Files\MBDB01\MBDB01.edb'
```
Nothing can follow the backtick on that line, **not even a space**, or the command breaks.

---

## 7. `$true`, `$false`, `$null`
These are PowerShell's yes, no and empty.

| Value | Meaning | GUI equivalent |
|---|---|---|
| `$true` | yes / on | ticking a box |
| `$false` | no / off | unticking a box |
| `$null` | empty / nothing | leaving a field blank |

For example, `-FormsAuthentication $false` is the same as unticking that box in the GUI, and `-ExternalUrl $null` is the same as leaving the field blank.

---

## 8. `-Confirm:$false` and `-Force`
These switches skip the "Are you sure? [Y/N]" prompts. The GUI lab has you press **Y** twice for each move, and these switches answer yes automatically.

> Only use them when you're sure of the command, because nothing stops you if it's wrong.

---

## 9. `-EA SilentlyContinue`
This is short for `-ErrorAction SilentlyContinue`. It means "if this fails, don't show an error, just keep going". The guide uses it where an item **may already exist** (like the server A records), so a "record already exists" error doesn't stop you.

---

## 10. Hashtables: `@{Add='...'}`
```powershell
Set-SendConnector 'To Internet' -SourceTransportServers @{Add='NEWEXSERVER'}
```
`@{ }` is a **hashtable**, a set of key/value pairs. Here it tells Exchange to **add** this server to the existing list rather than replace the list. Without it, `Set-SendConnector` would wipe out the servers already listed. You can use `@{Remove='SERVER'}` the same way to take one off.

---

## 11. `ft` and `fl`: Formatting Output
| Alias | Full name | Shows results as |
|---|---|---|
| `ft` | `Format-Table` | a compact table |
| `fl` | `Format-List` | a detailed list, one property per line |

```powershell
Get-MailboxDatabase | ft Name,Server,EdbFilePath
```
They only change how results are **displayed**. The guide uses them in the verify steps so you can see at a glance that settings took.

---

## 12. Command Verbs: `Get-` vs `Set-` / `New-` / `Add-`
PowerShell commands follow a **Verb-Noun** pattern, and the verb tells you what the command does:

| Verb | Does | Safe to run anytime? |
|---|---|---|
| `Get-` | **reads** and displays | ✅ yes, it changes nothing |
| `Set-` | **changes** something that exists | ⚠️ makes a change |
| `New-` | **creates** something new | ⚠️ makes a change |
| `Add-` | **adds** to something that exists | ⚠️ makes a change |
| `Remove-` | **deletes** | ❌ be careful |

Each section of the guide follows the same pattern: make the change with `Set-`, `New-` or `Add-`, then run a `Get-` to confirm it worked.

> **Tip:** if you're unsure what a command does, run `Get-Help <CommandName> -Examples`.
