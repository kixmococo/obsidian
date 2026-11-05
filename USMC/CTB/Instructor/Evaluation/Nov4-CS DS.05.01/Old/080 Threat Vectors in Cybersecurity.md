# 080 Threat Vectors in Cybersecurity

![Course cover](images/cover.png)

*Introduction to Cyber Security*

## Contents

1. [Threat Types](#1-threat-types)
2. [Cyber Attack Examples](#2-cyber-attack-examples)

---

## 1. Threat Types

These threats collectively target **confidentiality, integrity, and availability (CIA)** — the core pillars of cybersecurity. Understanding both the technical method and human factor behind each type is essential for building strong defenses.  
These are the most common threat types.

### Social Engineering

Social engineering is the use of deception and psychological manipulation to trick people into revealing sensitive information or performing actions that compromise security. It targets human behavior rather than technical systems.  
The "human element" is typically considered the weakest link in cybersecurity. You can think of it as the most **direct** way of compromising security as this is the primary method in **breaching trust**. It is often attributed or associated with all other threat vectors.  
**Example**: "I am from IT support, and I need to fix an urgent issue on your computer!"

### Common Threat Vectors

#### Malware

Software made to harm (viruses, worms, ransomware).

#### Phishing

Tricking people into revealing info.

#### Man-in-the-Middle (MitM)

Intercepting communication between two parties.

#### Distributed Denial-of-Service (DDoS)

Overloading systems to knock them offline.

#### Zero-Day Exploits

Attacks on vulnerabilities not yet patched.

### Best Defenses Against Threats

- **Strong Passwords** → Avoid names/birthdays, use length + complexity.
- **Multi-Factor Authentication (MFA)** → Password + extra verification.
- **Software Updates** → Patch vulnerabilities quickly.
- **Antivirus & Firewalls** → Detect/block threats.
- **Data Backups** → Recover after an attack.
- **Network Segmentation** → Separate critical systems from public ones.

### Safe Habits

Consistency and mindfulness are key!

- **Think before you click** → Links, attachments, QR codes.
- **Use HTTPS** → Encrypted website connections.
- **Check senders** → Phishing emails often look “off” if you check closely.
- **Limit permissions** → Apps shouldn’t have access they don’t need.
- **Log out** → Especially on public/shared devices.

---

## 2. Cyber Attack Examples

### 1. Malware – SolarWinds Cyberattack (2020)

**Description:** Russian state-sponsored hackers compromised SolarWinds' Orion software updates, embedding malware known as SUNBURST. This malware provided backdoor access to over 18,000 organizations, including U.S. government agencies and defense contractors.  
**Why It Matches:** The attack involved malicious software (malware) inserted into legitimate software updates, allowing unauthorized access to sensitive systems.

**Discussion questions:**

- How can malware hide itself in software that people and organizations already trust?
- What makes government networks especially vulnerable to supply chain attacks?
- If you were responsible for cybersecurity at the DoD, what layered defenses would you implement to prevent this type of attack?

### 2. Phishing – Operation Shady RAT (2006–2011)

**Description:** A prolonged cyber-espionage campaign targeted at least 71 organizations, including U.S. defense contractors and government agencies. Attackers used spear-phishing emails to gain unauthorized access to sensitive information.  
**Why It Matches:** The campaign relied on deceptive emails to trick recipients into revealing credentials, a hallmark of phishing attacks.

**Discussion questions:**

- Why are humans often considered the weakest link in cybersecurity?
- How could this campaign have been prevented if employees had been trained differently?
- How do attackers use social engineering to bypass technical defenses?

### 3. Man-in-the-Middle (MitM) – BGP Hijacking Incident (2013)

**Description:** In 2013, a series of Border Gateway Protocol (BGP) hijacking incidents redirected internet traffic intended for U.S. government agencies and corporations to Belarus and Iceland. This redirection allowed malicious actors to intercept and potentially manipulate sensitive data. Approximately 1,500 IP blocks were affected, and the hijacks occurred at least 38 times, often going unnoticed.  
**Why It Matches:** BGP hijacking is a form of MitM attack where attackers exploit vulnerabilities in the internet's routing infrastructure to intercept data between two parties. In this case, the attackers inserted themselves into the communication path between the DoD and external entities, allowing them to eavesdrop on or alter the data being transmitted.

**Discussion questions:**

- Why is it difficult to detect when someone is intercepting communications in transit?
- How does MitM differ from malware or phishing in terms of the attack vector?
- How might encryption or network monitoring have mitigated this attack?

### 4. DDoS – GitHub DDoS Attack (2015)

**Description:** In 2015, GitHub, a widely used code hosting platform, experienced a massive DDoS attack attributed to Chinese cyber actors. The attack utilized a technique known as "reflected DNS amplification," where traffic from Baidu, a Chinese search engine, was redirected to GitHub, overwhelming its servers. This attack was politically motivated, aiming to disrupt access to certain content.  
**Why It Matches:** DDoS attacks involve overwhelming a target's systems with excessive traffic, rendering services unavailable. This incident exemplifies a large-scale DDoS attack impacting a major platform.

**Relation to DoD:** While the attack targeted GitHub, a platform used by many developers, including those in the defense sector, it underscores the vulnerabilities in critical infrastructure and the potential impact on defense-related operations.

**Discussion questions:**

- Why is it important for critical infrastructure to plan for attacks even if they don’t target them directly?
- How could such an attack impact military operations if it targeted a DoD network?
- What strategies could organizations use to “absorb” or mitigate overwhelming traffic?

### 5. Zero-Day Exploits – CVE-2017-8570 in Ukraine (2017)

**Description:** A remote code execution vulnerability in Microsoft Office (CVE-2017-8570) was exploited in cyberattacks against Ukrainian government entities. The exploit allowed attackers to execute arbitrary code via specially crafted Office documents.  
**Why It Matches:** Zero-day exploits target previously unknown vulnerabilities, allowing attackers to execute malicious actions before patches are available.

**Discussion questions:**

- Why are zero-day vulnerabilities so valuable to attackers?
- How do organizations defend against attacks that exploit unknown vulnerabilities?
- What is the ethical responsibility of software companies and governments when a zero-day is discovered?

---

### ✍️ Activity: Cyber Threats Reflection

Write your reflection on the threats and case studies above.

> *Your response:*
>
> _______________________________________________
