# Escalate-my-privileges_LAB

Objective

The escalate_my_privilege lab is a small virtual machine designed with web vulnerabilities to be detected by the penetration testers. The goal of this excercise was to identify the vulnerabilities on the server, exploit the vulnerabilities to access the root id, capture the flag stored in the root directory.

Skills Learned

- Vulnerability assessment using Nmap.
- Remote code Execution via the upload of bash reverse shells.
- Privilege Escalatio techniqe in order to gain unauthenticated access.
- Problem solving and critical thinking in finding and exploitng vulnerabilites.
- Directory bruteforcing.
- Exploitation of sudo binaries for privilege escalation.
- Documentation and reporting of vulnerabilites along with mitigation strategies.

Tools Used

Nmap: Vulnerability analysis tool.
VmWare workstation - virtual machine environment
Kali Linux - Os used to carry out attacks and hosted on vmware.

Step 1: Install and configure Escalate_my_privileges machine. The compressed target VM file was downloaded from the vulnhub platform, after the file was extracted and added to the VMware workstation as a virtual machine.

Ref 1: <img width="480" height="270" alt="Screenshot (47)" src="https://github.com/user-attachments/assets/aa72ab35-2f37-4530-88a4-5d18a4267b5b" />

- Step 2:  Vulnerability assessment.
The target machine was scanned using nmap, were the IP address of the machine was specified along with option -sV. The -sV directs nmap to check for the version of the current services run on any open port found. The purpose of this scan is to identify any open port and oudated service on the open port that can be exploited in order to get a foothold on the target machine. From the conducted scan, the TCP ports 80,22, and 111 were found open with apache web server, ssh, and rpcbind running respectively. The main point of intrest in on the port 80 as a directory /phpbash.php is found in the robots.txt entry 

<img width="480" height="270" alt="Screenshot (35)" src="https://github.com/user-attachments/assets/c57f44c7-3305-40a3-8f9a-bfb750ac84fe" />

- Step 3: Initial Foothold

After navigating to the /phpbash.php directory on the web server it yielded an apache bash environment which indicates potential for running scripts that could yield access to root. 

<img width="480" height="270" alt="Screenshot (37)" src="https://github.com/user-attachments/assets/81c29f13-d12a-449b-9df3-0a308c175168" />

From navigating through the bash environment on apache, on the home directory a user armour was found, after entering the directory three interesting files were found which were Credentials.txt, in that file contained a possible password md5(rootroot1) which from the looks is meant to be md5 hash format. Also, a runme.sh file was found, this signifies possibility of appending it with a reverse shell to gain remote access to the machine. 

<img width="480" height="270" alt="Screenshot (38)" src="https://github.com/user-attachments/assets/f412a721-d162-49d7-8a89-9372c0d1848f" />

After appending to the script and running it a shell was generated indicating access has been gained to the machine. Now the next approach is to gain access to the armour user, to do this the password obtained earlier was converted to md5 format and used to authenticate the armour user in order to gain access.

<img width="480" height="270" alt="Screenshot (39)" src="https://github.com/user-attachments/assets/a95af1b3-7ad7-433b-900e-4465a4be4bd9" />

Step 4: Privilege escalation: Lateral Movement:

After successfully gaining access to the machine via the armour user, we can now try to escalate privileges using sudo binary "sudo -l" to see files that can be executed, the result showed that
armour user has rights to execute /bin/bash meaning root access can be gained. After gaining access to the root account the root directory was inspected and a file proof.txt was found which likely contains the flag. After using cat to read the contents of the file the flag was obtained.

<img width="480" height="270" alt="Screenshot (40)" src="https://github.com/user-attachments/assets/c7a8a057-a308-412c-a9cc-df90a257f9bb" />
<img width="480" height="270" alt="Screenshot (41)" src="https://github.com/user-attachments/assets/284923e8-976c-42cf-a190-a765c7e42362" />

Mitigation strategies to secure this machine from the penetration test carried out:

Reconnaissance and Service Enumeration (Nmap Scan)
Issue:
Open ports (22, 80, 111) exposed unnecessary services to the attacker. /phpbash.php was discoverable through robots.txt, leading directly to remote command execution.

Mitigation:

Minimize attack surface: close unused ports, restrict SSH access (22) with firewall rules or VPN access.

- Do not leave sensitive entries like /phpbash.php inside robots.txt (attackers always check it).
- Regularly audit exposed services with Nmap, Nessus, OpenVAS.
- Disable rpcbind unless strictly required.

Weak Web Application Security (phpbash.php)
Issue: /phpbash.php allowed attackers to interact with an unrestricted shell on the webserver.

Mitigation:

- Never deploy debugging/administrative backdoors (like phpbash) in production.
- Use web application firewalls (WAFs) to detect and block suspicious uploads or shell-like behavior.
- Employ file integrity monitoring (e.g., Wazuh, Tripwire) to detect unauthorized files.
- Enforce least privilege for the web server user (no unnecessary read/write access).

Insecure Home Directory Artifacts (Credentials.txt, runme.sh)
Issue: Credentials were stored in plaintext (rootroot1) and weakly hashed (MD5). A writable script (runme.sh) allowed attackers to append a reverse shell and execute arbitrary commands.

Mitigation:

- Never store plaintext passwords or weak hashes (MD5 is deprecated). Use bcrypt, Argon2, or PBKDF2.
- Store secrets in environment variables or secure vaults (e.g., HashiCorp Vault).
- Restrict file permissions (chmod 600) and ownership.
- Scripts in home directories should never be world-writable.
- Perform regular audits of user home directories for weak configurations.

Privilege Escalation via Sudo Misconfiguration
Issue:
The user armour had sudo rights to execute /bin/bash without restrictions, granting instant root access.

Mitigation:

- Apply principle of least privilege: restrict sudo access to only necessary administrative binaries (e.g., systemctl, service).
- Remove NOPASSWD rules unless absolutely required.
- Regularly review /etc/sudoers and enforce stricter access control.
- Monitor sudo usage with centralized logging (e.g., SIEM like Splunk or Wazuh)

