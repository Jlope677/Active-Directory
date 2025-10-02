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

### **Step 1 – Create Azure Virtual Machines**    

 **Create a Resource Group**
<img width="1071" height="930" alt="rg1" src="https://github.com/user-attachments/assets/c8e5f6cb-5250-41c1-a3ac-cec1bb3dbf52" />

**Create a Virtual Network and Subnet**
<img width="1266" height="890" alt="vnet" src="https://github.com/user-attachments/assets/075b0eb4-89b5-44c9-8041-45f44d12f723" />

**Create the Domain Controller VM (Windows Server 2022) named “DC-1”**

<img width="1105" height="862" alt="dc-1(1)" src="https://github.com/user-attachments/assets/1f073bf0-edb5-40a1-ab7d-65906a4cf81b" />
<img width="1339" height="887" alt="dc-1(2)" src="https://github.com/user-attachments/assets/21bb7f0c-d27f-45a8-9f9c-3f6c041e84a7" />
<img width="1050" height="646" alt="dc-1(3)" src="https://github.com/user-attachments/assets/ef2e1745-3ce9-4e5c-9579-19c5d7bc2c88" />
<img width="1304" height="945" alt="dc-1(4)" src="https://github.com/user-attachments/assets/1320a6df-810c-4cad-ab58-4fa1200fecca" />








 

---

### **Step 2 – Install Active Directory Domain Services (AD DS)**  
- Installed **AD DS** on DC-1.  
- Promoted server to a **Domain Controller** for domain: `mydomain.com`.  

📸 *Screenshots: Installing AD DS & domain setup wizard*  

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
