# 🛠️ Active Directory

This project demonstrates my ability to **deploy, configure, and manage Active Directory (AD)** in a simulated enterprise environment. The lab was built using **Azure Virtual Machines**, a **Windows Server 2022 Domain Controller (DC-1)**, and a **Windows 10 Client (Client-1)** joined to the domain.  

The lab covers:  
1. **Deploying Active Directory**  
2. **Creating & Managing User Accounts with PowerShell**  
3. **Configuring Remote Desktop Access**  
4. **Enabling, Disabling, Unlocking, and Resetting Accounts**  
5. **Configuring Account Lockout Policies via Group Policy**  

---

## 🌎 Environments and Technologies Used  
- Microsoft Azure (Virtual Machines / Compute)  
- Active Directory Domain Services (AD DS)  
- Windows Server 2022 (Domain Controller: DC-1)  
- Windows 10 Pro (Client-1)  
- Remote Desktop (RDP)  
- PowerShell ISE  
- Group Policy Management Console (GPMC)  
- Event Viewer  

---

## 💻 Operating Systems Used  
- Windows Server 2022  
- Windows 10 Pro (22H2)  

---

# Part 1 – Deploying Active Directory  

## **Step 1 – Create Azure Virtual Machines**    

### **Create a Resource Group**
![Resource Group](https://github.com/user-attachments/assets/c8e5f6cb-5250-41c1-a3ac-cec1bb3dbf52)

### **Create a Virtual Network and Subnet**
![Virtual Network](https://github.com/user-attachments/assets/075b0eb4-89b5-44c9-8041-45f44d12f723)

### **Create the Domain Controller VM (Windows Server 2022) – `DC-1`**

- **VM Name:** `DC-1`  
- **Operating System:** Windows Server 2022  
- **Username:** `labuser`  
- **Password:** `Cyberlab123!`

![DC-1 Setup 1](https://github.com/user-attachments/assets/1f073bf0-edb5-40a1-ab7d-65906a4cf81b) 
![DC-1 Setup 2](https://github.com/user-attachments/assets/21bb7f0c-d27f-45a8-9f9c-3f6c041e84a7) 
![DC-1 Setup 3](https://github.com/user-attachments/assets/ef2e1745-3ce9-4e5c-9579-19c5d7bc2c88) 
![DC-1 Setup 4](https://github.com/user-attachments/assets/1320a6df-810c-4cad-ab58-4fa1200fecca) 

### **Configure Domain Controller’s NIC Private IP (Set to Static)**

To ensure stability in the environment, the Domain Controller’s private IP address must be set to **static**.  


| Step | Screenshot |
|------|------------|
| Verify NIC Private IP | <img width="1174" height="602" alt="dc-1 private ip" src="https://github.com/user-attachments/assets/ba2b79d5-1226-4a4b-b6fe-de90b56b1cb1" /> |
| Open NIC Settings | <img width="1904" height="772" alt="dc-1 nic(1)" src="https://github.com/user-attachments/assets/ae15ad6f-48b8-4653-8d0a-ed61485b1a22" /> |
| Configure Static IP | <img width="1601" height="752" alt="dc-1 nic(2)" src="https://github.com/user-attachments/assets/94ee9222-9556-4ef8-bf91-e401b1d0961e" /> |

### **Log into DC-1 and Disable Windows Firewall (for testing connectivity)**

> ⚠️ *This is only for testing purposes in the lab. In a production environment, firewall rules should be properly configured instead of disabling the firewall.*


<img width="505" height="305" alt="dc-1 firewall(1)" src="https://github.com/user-attachments/assets/d2cef197-133e-473e-a9b9-bb890b359dcd" />  
<img width="729" height="568" alt="dc-1 firewall(2)" src="https://github.com/user-attachments/assets/f5746cd0-0165-448f-be2b-86d154d60da8" />  
<img width="510" height="579" alt="dc-1 firewall(3)" src="https://github.com/user-attachments/assets/8933ebd7-c96f-4678-8fb8-4d9b5e6bf73a" />  
<img width="537" height="596" alt="dc-1 firewall(4)" src="https://github.com/user-attachments/assets/6d6a0d32-bafc-40a7-b468-995f7082ef86" />  
<img width="525" height="600" alt="dc-1 firewall(5)" src="https://github.com/user-attachments/assets/56af524c-0b19-48f0-8518-e8288abe4a0b" />  


### **Create the Client VM (Windows 10) – `Client-1`**

- **VM Name:** `Client-1`  
- **Operating System:** Windows 10  
- **Username:** `labuser`  
- **Password:** `Cyberlab123!`  


<img width="1184" height="714" alt="Client-1(1)" src="https://github.com/user-attachments/assets/b07ce59a-f49d-4899-a42f-d1d1f2bf915c" />  
<img width="1231" height="591" alt="Client-1(2)" src="https://github.com/user-attachments/assets/1ea982cf-dc28-4b4a-b1bc-b54d7e5df518" />  
<img width="1263" height="774" alt="Client-1(3)" src="https://github.com/user-attachments/assets/a9a229a1-db20-4936-ad22-cbf1c516633d" />  
<img width="1212" height="833" alt="Client-1(4)" src="https://github.com/user-attachments/assets/0d222a70-0296-4e0b-a963-357742910911" />  


### ** Configure Client-1’s DNS Settings to Use DC-1’s Private IP**

Once Client-1 is created:  
1. Set its **DNS server** to DC-1’s **private IP**.  
2. Restart the VM.  
3. Ping DC-1’s private IP to confirm connectivity.  
4. Run `ipconfig /all` to verify that DC-1 is listed as the DNS server.  


<img width="1902" height="891" alt="client-1 dns(1)" src="https://github.com/user-attachments/assets/375e3451-5c3d-426d-a63f-11112ab40761" />  
<img width="1343" height="786" alt="client-1 dns(2)" src="https://github.com/user-attachments/assets/16be22fa-cb79-4fda-b34f-b60553a6fb71" />  
<img width="1581" height="615" alt="restart client-1" src="https://github.com/user-attachments/assets/2f861a03-27b7-4b8c-9cc7-2ef8cc7abe25" />  
<img width="1030" height="958" alt="ping dc-1 private ip" src="https://github.com/user-attachments/assets/6301adc4-6ae6-4434-85a5-2dbf3d37a85d" />  

---

### **Step 2 – Install Active Directory Domain Services (AD DS)**  
- Installed **AD DS** on DC-1.  
- Promoted server to a **Domain Controller** for domain: `mydomain.com`.  
<img width="589" height="614" alt="login in to domain controller dc-1" src="https://github.com/user-attachments/assets/bfdf70d2-8b00-4093-8510-abb3092b169c" />
<img width="1363" height="954" alt="active_directory1" src="https://github.com/user-attachments/assets/ffabe006-84b1-4c6f-8098-74838e898b50" />
<img width="1309" height="969" alt="active_directory2" src="https://github.com/user-attachments/assets/13796227-2fec-4ae7-b772-5ca436be9fc0" />
<img width="627" height="530" alt="active_directory3" src="https://github.com/user-attachments/assets/1a7586d1-c2ad-4ad0-a72f-068718a80e12" />
<img width="1022" height="711" alt="active_directory4" src="https://github.com/user-attachments/assets/dedd5743-3e69-4510-82e1-7802c8c96aea" />





---

### **Step 3 – Join Client-1 to the Domain**  
- Logged into Client-1 → updated system settings → joined to domain `mydomain.com`.  
- Verified in **Active Directory Users and Computers (ADUC)** that Client-1 appeared under **Computers**.  

📸 *Screenshots: Domain join + verification in ADUC*  

---

### **Step 4 – Create Organizational Units (OUs)**  
- Created OUs: `_ADMINS`, `_CLIENTS`, `_EMPLOYEES`.  
- These OUs help organize domain objects logically.  

📸 *Screenshot: OUs visible in ADUC*  

---

### **Step 5 – Create a Domain Admin Account**  
- Created `jane_admin` inside `_ADMINS`.  
- Added to **Domain Admins** group.  
- Verified login with domain admin credentials.  

📸 *Screenshot: jane_admin account creation + RDP login test*  

---

### **Step 6 – Automate Bulk User Creation with PowerShell**  
- Wrote a **PowerShell script** (`create-users.ps1`) to generate thousands of test users with randomized names in `_EMPLOYEES`.  

```powershell
$PASSWORD_FOR_USERS = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if (($count % 2) -eq 0) {
            $name += $consonants[(Get-Random -Minimum 0 -Maximum $($consonants.Count))]
        } else {
            $name += $vowels[(Get-Random -Minimum 0 -Maximum $($vowels.Count))]
        }
        $count++
    }
    return $name
}

for ($i=0; $i -lt $NUMBER_OF_ACCOUNTS_TO_CREATE; $i++) {
    $username = (generate-random-name()) + "." + (generate-random-name())
    Write-Output "Creating user: $username"
}
```

📸 *Screenshots: PowerShell script + ADUC populated with thousands of new users*  

---

# Part 2 – Enabling, Unlocking Accounts & Resetting Passwords  

### **Step 1 – Configure Remote Desktop for Domain Users**  
- Enabled RDP on **Client-1**.  
- Allowed **Domain Users** to log in.  

📸 *Screenshots: Remote Desktop settings + adding Domain Users group*  

---

### **Step 2 – Disable and Enable Accounts**  
- Disabled a user account (`gef.hoc`).  
- Attempted login via RDP (got **“Account disabled”** error).  
- Re-enabled the account and confirmed successful login.  

📸 *Screenshots: Disable → Error → Enable → Login restored*  

---

### **Step 3 – Simulate Account Lockouts**  
- Entered wrong passwords multiple times.  
- Triggered **account lockout**.  
- Verified events in **Event Viewer → Security Logs (Event ID 4625: Failed Logon)**.  

📸 *Screenshots: Lockout messages + Event Viewer logs*  

---

### **Step 4 – Unlock a Locked-Out Account**  
- Used **ADUC Properties** to unlock the account.  
- Confirmed login restored.  

📸 *Screenshots: Unlock checkbox in ADUC + Apply*  

---

### **Step 5 – Reset Passwords**  
- Reset password of a locked account in ADUC.  
- Selected **“Unlock account”** during reset.  
- Successfully logged in with the new password.  

📸 *Screenshots: Reset password + Successful login*  

---

### **Step 6 – Configure Account Lockout Policy**  
- Used **Group Policy Management Console (gpmc.msc)**.  
- Edited **Default Domain Policy → Account Lockout Policy**.  
- Applied:  
  - Account lockout threshold: **5 attempts**  
  - Lockout duration: **30 minutes**  
  - Reset counter after: **10 minutes**  

📸 *Screenshots: GPMC settings + gpupdate /force in PowerShell*  

---

## 📊 Results & Observations  
- **Disabled accounts** block logins instantly.  
- **Account lockouts** are logged in Event Viewer with detailed audit trails.  
- **Unlocking and resetting passwords** restore access without recreating accounts.  
- **Group Policy** provides centralized control over lockout rules.  

---

## 🤔 What I Learned  
- How to fully deploy Active Directory in a lab setting.  
- How to manage accounts: enable, disable, unlock, and reset passwords.  
- How to detect and troubleshoot login failures with Event Viewer.  
- How to enforce account lockout policies for stronger security.  
- How to automate account creation with PowerShell.  

---


✅ This project demonstrates **end-to-end Active Directory administration**: deployment, user management, account security, and Group Policy configuration. It reflects **core skills of a System Administrator** in real enterprise environments.  
