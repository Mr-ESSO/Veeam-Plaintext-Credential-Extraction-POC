# Veeam-Plaintext-Credential-Extraction-POC
A Proof of Concept (PoC) demonstrating the risk of plaintext credential extraction in Veeam Backup &amp; Replication.
├── README.md
├── Veeam-Credential-Extraction-POC.exe (EXE) 
├── screenshots/             (POC images & examples)
├── LICENSE                  (Optional: MIT or CC0)

# Overview
This repository provides a Proof of Concept (PoC) demonstrating how plaintext credentials can be extracted from the Veeam Backup & Replication PostgreSQL database.
The purpose of sharing this information is to help administrators and security professionals understand the associated risks and implement appropriate security measures to protect their backup infrastructure.

# Risk Description
Local users on the Veeam Backup & Replication server can extract plaintext credentials that are stored in the PostgreSQL database.

These credentials include:
- VMware vCenter and ESXi hosts
- Windows and Linux servers that are configured as backup targets
  
This process leverages:
- PowerShell and psql.exe
- Windows Data Protection API (DPAPI)
- Veeam EncryptionSalt stored in the Windows Registry

# Impact
- Theft of credentials stored in the Veeam Backup & Replication configuration and database
- Potential compromise of connected hypervisors and backup targets
- Lateral movement within the environment

# Proof of Concept (POC)
This PoC demonstrates the following steps

STEP1: Go to Directory PostgreSQL and check version veeam backup & replication
- [C:\Program Files\PostgreSQL\15\bin]
<img width="640" alt="2025-05-31_184233" src="https://github.com/user-attachments/assets/af6497e1-0595-4853-9830-5fc83f8413c0" />

STEP2: Check Information in the Windows Registry
- Command: PS C:\Program Files\PostgreSQL\15\bin> Get-ItemProperty -Path "HKLM:\SOFTWARE\Veeam\Veeam Backup and Replication\Data"
<img width="610" alt="1 Check Information-Registry" src="https://github.com/user-attachments/assets/4234cfd2-b1f2-410e-bc8e-3c0e5e037f6f" />

STEP3: Check DB-PostgreSQL[Credentials Include(VMware vCenter, ESXi hosts Windows and Linux servers configured as backup targets)]
- Command: PS C:\Program Files\PostgreSQL\15\bin> .\psql.exe -d VeeamBackup -U postgres -c "SELECT user_name,password FROM credentials"
<img width="962" alt="2 Check DB-Postgres SQL" src="https://github.com/user-attachments/assets/029f6daf-73f5-42b4-a77f-eabf4d14f029" />

Explanation
* This command connects to the PostgreSQL database used by Veeam Backup & Replication.
* It queries the credentials table, which stores credentials for backup targets (e.g., ESXi hosts, repositories). These passwords are stored encrypted by Veeam.
* The passwords configured during the server addition process for backup purposes are stored in the PostgreSQL database on the Veeam Backup & Replication server. These passwords are encrypted, which is the intended and secure behavior to prevent unauthorized access or misuse of the credentials.

STEP4: Run File Veeam-Credential-Extraction-POC.exe
Download the file Veeam-Credential-Extraction-POC.exe from the GitHub repository and move it to the directory: C:\Program Files\PostgreSQL\15\bin\
- Command: PS C:\Program Files\PostgreSQL\15\bin> .\Veeam-Extract-Credentials.exe
<img width="954" alt="3 Run File Veeam-Credential-Extraction-POC" src="https://github.com/user-attachments/assets/8c405190-50c8-4b86-84c7-5a64e328566b" />

This screenshot shows the full process of extracting and decrypting credentials from a Veeam Backup & Replication server using:
- A PostgreSQL database query to dump encrypted credentials from the credentials table.
- A custom tool called Veeam-Credential-Extraction-POC.exe, used to decrypt those credentials and display them in plaintext.

# Why is this Critical?
This demonstrates full credential compromise on a Veeam Backup & Replication server:
- Once an attacker gains local administrator access, they can extract and decrypt all credentials stored in the Veeam application.
- These credentials can be used to compromise the backup infrastructure, ESXi hypervisors, and domain controllers.
- This activity bypasses AV/EDR because it uses native system tools (PowerShell, psql) and known cryptographic APIs (DPAPI).
