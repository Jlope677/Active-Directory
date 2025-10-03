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


### **Configure Client-1’s DNS Settings to Use DC-1’s Private IP**

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

## **Step 2 – Install Active Directory Domain Services (AD DS)**  
### Installed **AD DS** on DC-1.
  <img width="1512" height="1008" alt="dc-1(add roles and features)" src="https://github.com/user-attachments/assets/eb79881d-44a5-4b32-85f2-2016f1f57ce3" />
  <img width="1059" height="773" alt="dc-1(add roles and features)2" src="https://github.com/user-attachments/assets/5c862e42-dcdd-46dd-bb6d-cfd79b841574" />
  <img width="1010" height="718" alt="dc-1(add roles and features)3" src="https://github.com/user-attachments/assets/b522ccee-bbab-457e-9c4c-e35ea3c6372e" />
  <img width="1071" height="724" alt="dc-1(add roles and features)4" src="https://github.com/user-attachments/assets/8d53c83d-3203-42d0-bb52-1d325cf69c69" />


### Promoted server to a **Domain Controller** for domain: `mydomain.com`.  
<img width="1323" height="642" alt="dc-1 promote" src="https://github.com/user-attachments/assets/e11c3317-6629-4768-a455-1ea6c5a9ce4f" />
<img width="1050" height="796" alt="dc-1 promote2" src="https://github.com/user-attachments/assets/2d401cdd-1476-48d1-8647-1eed16d6003f" />
<img width="997" height="767" alt="dc-1 promote3(Password1)" src="https://github.com/user-attachments/assets/ef5ecaa5-322a-4f14-85bd-0de6c1260a5d" />
<img width="989" height="758" alt="dc-1 promote4" src="https://github.com/user-attachments/assets/efb816eb-f724-475b-8e71-a75ed24fd4ee" />
<img width="589" height="614" alt="login in to domain controller dc-1" src="https://github.com/user-attachments/assets/0b686fdf-1a09-4a54-9937-e2c0c973ccb4" />

---

## **Step 3 – Create Organizational Units (OUs)**  
- Created OUs: `_ADMINS`, `_CLIENTS`, `_EMPLOYEES`.  
- These OUs help organize domain objects logically.  
<img width="1363" height="954" alt="active_directory1" src="https://github.com/user-attachments/assets/78fdcda9-a3bd-4f56-b705-5e7559638b09" />
<img width="1309" height="969" alt="active_directory2" src="https://github.com/user-attachments/assets/df209517-cc19-4e85-b9e2-161c1e7b9d82" />
<img width="627" height="530" alt="active_directory3" src="https://github.com/user-attachments/assets/a04bcab2-e260-4915-99d2-9c0a0648ace5" />
<img width="1022" height="711" alt="active_directory4" src="https://github.com/user-attachments/assets/c2ee783e-b0a1-4ba7-979c-0d5c3e2d73ae" />
<img width="1025" height="721" alt="new OU named “_CLIENTS”" src="https://github.com/user-attachments/assets/83ab161a-226b-4b14-a71f-0c7babe33d38" />


---

## **Step 4 – Create a Domain Admin Account**  
- Created `jane_admin` inside `_ADMINS`.  
- Added to **Domain Admins** group.  
- Verified login with domain admin credentials.  
<img width="1101" height="867" alt="jane_admin1" src="https://github.com/user-attachments/assets/916fd85f-8b29-4c63-959c-c42f21c86c24" />
<img width="1022" height="765" alt="jane_admin2" src="https://github.com/user-attachments/assets/ec6c3fce-0931-447d-ae0a-7dd118347d74" />
<img width="927" height="640" alt="jane_admin3" src="https://github.com/user-attachments/assets/08bf1905-ed27-4bc9-8477-efc7379696d0" />
<img width="613" height="530" alt="jane_admin4" src="https://github.com/user-attachments/assets/9d368035-1fc1-4e42-8336-e1f0f7bd4fcf" />
<img width="990" height="719" alt="jane_admin5" src="https://github.com/user-attachments/assets/f1401389-be38-43e3-b5f1-3e8a2067a285" />
<img width="1143" height="758" alt="jane_admin6" src="https://github.com/user-attachments/assets/6d637d6e-b001-45a5-99f3-58b78a0fc80c" />
<img width="577" height="687" alt="jane_admin7" src="https://github.com/user-attachments/assets/8207f0eb-a0ae-4487-bb11-bf631c745731" />
<img width="606" height="623" alt="login into jane_admin" src="https://github.com/user-attachments/assets/47989bf4-7eff-4d06-812d-ef300b2e778e" />







 

---
## **Step 5 – Join Client-1 to the Domain**

### Joining Client-1 to `mydomain.com`
I logged into **Client-1**, opened **System Settings**, and updated the computer to join the domain `mydomain.com`.

<p align="center">
  <img width="1275" alt="Client-1 join to domain (step 1)" src="https://github.com/user-attachments/assets/edd3fa14-e9c0-4b60-9758-fa865a57329d" />
</p>
<p align="center">
  <img width="777" alt="Client-1 join to domain (step 2)" src="https://github.com/user-attachments/assets/c53c55e4-c270-433f-99e5-8401ef9568d6" />
</p>
<p align="center">
  <img width="1286" alt="Client-1 join to domain (step 3)" src="https://github.com/user-attachments/assets/75a54e14-779a-48a3-a6cc-2e59164148fb" />
</p>


### Verification in ADUC
After joining, I opened **Active Directory Users and Computers (ADUC)** and verified that `Client-1` appeared under **Computers**.

<p align="center">
  <img width="1190" alt="Client-1 verified in ADUC" src="https://github.com/user-attachments/assets/93fa9b43-9e7f-4021-b0b7-c57fece916ea" />
</p>


### Organizing in OU
To keep the domain structured, I moved `Client-1` into the **_CLIENTS** Organizational Unit (OU).

<p align="center">
  <img width="1024" alt="Drag Client-1 into _CLIENTS OU" src="https://github.com/user-attachments/assets/5e76ad2d-887c-4302-8d24-d20a66f95575" />
</p>
---

## **Step 6 – Automate Bulk User Creation with PowerShell**  
- Wrote a **PowerShell script** (`create-users.ps1`) to generate thousands of test users with randomized names in `_EMPLOYEES`.  

```powershell
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}
```

<img width="1700" height="838" alt="script1" src="https://github.com/user-attachments/assets/fbe4e4cb-e358-48a1-b5a8-e5a7e4b693cf" />
<img width="1145" height="480" alt="script2" src="https://github.com/user-attachments/assets/727160c6-7dcc-41db-b4f5-d17fdb94fd32" />
<img width="1819" height="943" alt="results of script" src="https://github.com/user-attachments/assets/cd224e97-e27c-4cb6-93d5-5ce0fd6922a1" />

---

# Part 2 – Enabling, Unlocking Accounts & Resetting Passwords  

## **Step 1 – Configure Remote Desktop for Domain Users**  
- Enabled RDP on **Client-1**.  
- Allowed **Domain Users** to log in.
- Attempt to log into Client-1 with one of the accounts

<img width="948" height="910" alt="Remote Desktop" src="https://github.com/user-attachments/assets/d03955a9-298d-4940-bf95-16f55f8e8d3a" />
<img width="905" height="896" alt="Remote Desktop2" src="https://github.com/user-attachments/assets/548682a0-e7b4-4c9d-ab3e-ad7a818fcbe5" />
<img width="1189" height="721" alt="Remote Desktop3" src="https://github.com/user-attachments/assets/48b0d308-5078-4aed-996e-8b59da5ec972" />
<img width="557" height="617" alt="attempt to log into Client-1 with one of the accounts" src="https://github.com/user-attachments/assets/cd11e02e-0b5c-4331-94ca-c131c8c4ba93" />
<img width="824" height="691" alt="attempt to log into Client-1 with one of the accounts2" src="https://github.com/user-attachments/assets/4d3ef683-fdd8-40d3-9309-fbeca497cfb2" />


---

## **Step 2 – Disable and Enable Accounts**

In this step, I simulated disabling and re-enabling a user account in Active Directory.

- Disabled the user account **`gef.hoc`**.  
- Attempted to log in via RDP → received **“Account disabled”** error.  
- Re-enabled the account in **Active Directory Users and Computers (ADUC)**.  
- Confirmed login was restored and the user was able to access again.  

**Screenshots of the Process:**

| Action | Screenshot |
|--------|------------|
| Disable the account in ADUC | <img width="1033" height="766" alt="disable account" src="https://github.com/user-attachments/assets/0f4b1e41-7479-4efe-bc45-31d0d4c92723" /> |
| Confirmation of disabled account | <img width="1083" height="701" alt="disable account2" src="https://github.com/user-attachments/assets/20581854-23b0-41e9-849c-db7e6b452af9" /> |
| RDP login attempt → **Account disabled error** | <img width="772" height="492" alt="disable account3" src="https://github.com/user-attachments/assets/9a07f82c-6804-43f4-99b0-8f977953472c" /> |
| Re-enabled account in ADUC | <img width="1069" height="821" alt="enable account" src="https://github.com/user-attachments/assets/2faa6974-96f0-4bd7-9a20-58bf0eb136d9" /> |
| Successful login after re-enabling | <img width="671" height="633" alt="try logging in again" src="https://github.com/user-attachments/assets/2e7840aa-2cea-40e8-8a20-eb247369f688" /> |


---
## **Step 3 – Configure Account Lockout Policy**  
- 📖 [Reference: How To Configure Account Lockout Threshold in Group Policy](https://docs.google.com/document/d/1msUMWaPDMR1hPYxzGOlgN4KpUjnyyYEv3vvOQXkSpLQ/edit)

- Used **Group Policy Management Console (gpmc.msc)**.  
- Edited **Default Domain Policy → Account Lockout Policy**.  
- Applied:  
  - Account lockout threshold: **5 attempts**  
  - Lockout duration: **30 minutes**  
  - Reset counter after: **10 minutes**  
<img width="657" height="412" alt="grouppolicy1" src="https://github.com/user-attachments/assets/3f5d7ad7-c867-4ab0-943d-ae2cbdc7b5c9" />
<img width="973" height="709" alt="grouppolicy2" src="https://github.com/user-attachments/assets/017c1b0f-818e-48e0-9009-2d66d365aaa2" />
<img width="1654" height="832" alt="grouppolicy3" src="https://github.com/user-attachments/assets/bea019aa-ba18-44f8-91c4-72ee0210c75a" />
<img width="1225" height="747" alt="grouppolicy4" src="https://github.com/user-attachments/assets/4cd53a4c-1fcc-4865-8023-dfef81642528" />
<img width="1244" height="697" alt="forcing policy" src="https://github.com/user-attachments/assets/c684b7c1-0dae-4f03-b594-b7a23b751f15" />


## **Step 4 – Simulate Account Lockouts**  

To simulate an account lockout:  

- Entered the wrong password multiple times.  
- Triggered an **account lockout**.  
- Verified the lockout event in **Event Viewer → Security Logs**:  
 
<img width="628" height="544" alt="effects of putting a bad password" src="https://github.com/user-attachments/assets/ffc6059a-4670-437b-ab81-521013fe8c33" />
<img width="758" height="656" alt="failed password results" src="https://github.com/user-attachments/assets/ab711591-0189-4144-89d1-e792c0416028" />
<img width="1087" height="752" alt="failed password results2" src="https://github.com/user-attachments/assets/673c534e-6621-4d1b-98fb-c7d0abf9e4d7" />
<img width="1578" height="674" alt="event viewer in dc-1" src="https://github.com/user-attachments/assets/bd390f45-25fe-4de7-9e7e-9aa04a4b4ee7" />
<img width="1061" height="861" alt="evenviewer client-1" src="https://github.com/user-attachments/assets/b02b4286-41c6-475a-98f3-133dd691e45e" />
<img width="764" height="903" alt="evenviewer client-1(2)" src="https://github.com/user-attachments/assets/af609fb5-0121-400f-a216-629c5c6caeeb" />
<img width="1572" height="971" alt="evenviewer client-1(3)" src="https://github.com/user-attachments/assets/84ac6430-ada1-4bf7-b874-20ae6dde7da9" />
<img width="1015" height="794" alt="unlock account" src="https://github.com/user-attachments/assets/e5561ff2-90e3-415e-a991-2313c30bd90c" />

 

---


## **Step 5 – Reset Passwords**  
- Reset password of a locked account in ADUC.  
- Selected **“Unlock account”** during reset.  
- Successfully logged in with the new password.  
<img width="1023" height="697" alt="resetpassword" src="https://github.com/user-attachments/assets/e51901cc-bdd0-4ae4-98bd-6e5bc53be2bb" />
<img width="1075" height="772" alt="resetpassword2" src="https://github.com/user-attachments/assets/1ca561bc-24e4-4f3c-b452-76a504b49f10" />


---



### 📊 Results & Observations  
- **Disabled accounts** block logins instantly.  
- **Account lockouts** are logged in Event Viewer with detailed audit trails.  
- **Unlocking and resetting passwords** restore access without recreating accounts.  
- **Group Policy** provides centralized control over lockout rules.  

---

### 🤔 What I Learned  
- How to fully deploy Active Directory in a lab setting.  
- How to manage accounts: enable, disable, unlock, and reset passwords.  
- How to detect and troubleshoot login failures with Event Viewer.  
- How to enforce account lockout policies for stronger security.  
- How to automate account creation with PowerShell.  

---


✅ This project demonstrates **end-to-end Active Directory administration**: deployment, user management, account security, and Group Policy configuration. It reflects **core skills of a System Administrator** in real enterprise environments.  
