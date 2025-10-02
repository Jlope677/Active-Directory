# Active Directory in Azure: From Zero to Admin 🚀

> **Personal portfolio lab** — I built a full AD lab in Microsoft Azure (Domain Controller + Windows 10 client), joined the client to the domain, configured Remote Desktop for Domain Users, bulk‑created employees via PowerShell, and implemented account lockout/enable/disable workflows and log review.
>
> This repo showcases the exact steps I took, my screenshots, and what I learned — so hiring managers can quickly see my hands‑on Windows/AD/Azure skills.

---

## 🧭 Table of Contents
- [Project Summary](#project-summary)
- [Architecture](#architecture)
- [Environments & Tech](#environments--tech)
- [Prerequisites](#prerequisites)
- [Part 1 — Prepare AD Infrastructure in Azure](#part-1--prepare-ad-infrastructure-in-azure)
- [Part 2 — Deploy Active Directory & Join Client](#part-2--deploy-active-directory--join-client)
- [Part 3 — Remote Desktop for Domain Users](#part-3--remote-desktop-for-domain-users)
- [Part 4 — Bulk User Creation via PowerShell](#part-4--bulk-user-creation-via-powershell)
- [Part 5 — Account Lockout / Enable / Disable / Reset](#part-5--account-lockout--enable--disable--reset)
- [Demonstration](#demonstration)
- [What I Learned](#what-i-learned)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## Project Summary

**What this is:** A guided, repeatable lab that provisions a Windows Server 2022 Domain Controller and a Windows 10 client in Azure, implements core AD administration tasks, and demonstrates operational awareness (logs, policy, RDP access).

**Why it matters:** This lab mirrors **real entry‑level sysadmin/SOC workflows**: building a domain, joining clients, creating users at scale, and handling account lockouts/credential hygiene while validating via Windows event logs.

**My role:** I did the architecture, provisioning, configuration, and testing end‑to‑end, and documented evidence (screenshots and notes) after each milestone.

---

## Architecture

```
Azure Subscription
└── Resource Group: AD-Lab
    ├── VNet: AD-VNet (10.0.0.0/16)
    │   └── Subnet: default (10.0.0.0/24)
    ├── VM: DC-1 (Windows Server 2022)
    │   └── Static Private IP; DNS for VNet
    └── VM: Client-1 (Windows 10)
        └── DNS → DC-1 private IP
```

*📸 **Insert diagram/screenshot of your Azure resources here***

---

## Environments & Tech

- **Cloud:** Microsoft Azure (Resource Group, VNets, NICs, VMs)
- **OS:** Windows Server 2022 (Domain Controller), Windows 10 Client
- **Directory:** Active Directory Domain Services (AD DS), ADUC
- **Policy:** Group Policy (Account Lockout Policy)
- **Remote Access:** RDP for Domain Users
- **Scripting:** PowerShell (bulk user creation)
- **Logging:** Event Viewer on DC and Client

---

## Prerequisites

- Azure subscription
- RDP client
- Basic Windows admin familiarity
- (Optional) Your PowerShell script for user creation: `Generate-Names-Create-Users.ps1`

---

## Part 1 — Prepare AD Infrastructure in Azure

1. **Create** Resource Group, VNet, and Subnet.
2. **Provision** `DC-1` (Windows Server 2022) with a **static private IP**.
3. **Provision** `Client-1` (Windows 10) in the **same VNet/region** as `DC-1`.
4. On `Client-1`, set **DNS** to `DC-1`’s **private IP**, then **Restart** the VM.
5. From `Client-1`, verify connectivity:
   ```powershell
   ping <DC-1_PRIVATE_IP>
   ipconfig /all   # DNS should show DC-1’s IP
   ```

*📸 **Insert: Azure VNet + NIC settings, DC-1 static IP, Client-1 DNS, ping/ipconfig output***

---

## Part 2 — Deploy Active Directory & Join Client

1. On `DC-1`, **Install AD DS** and **Promote** to a new forest, e.g. `mydomain.com`.
2. Create OUs: `_EMPLOYEES` and `_ADMINS` in **ADUC**.
3. Create **Domain Admin**: user `jane_admin`, add to **Domain Admins**, sign in as `mydomain\jane_admin`.
4. **Join Client-1** to the domain (sign in as local `labuser`, join domain, auto‑restart).
5. In ADUC, create `_CLIENTS` OU and **move Client-1** into it.

*📸 **Insert: AD DS promotion wizard, OUs, Domain Admin creation, Client-1 shown in ADUC***

---

## Part 3 — Remote Desktop for Domain Users

On `Client-1` (signed in as `mydomain\jane_admin`):

1. Open **System Properties → Remote Desktop**.
2. **Allow** `Domain Users` to Remote Desktop.

*📸 **Insert: RDP settings showing Domain Users allowed***

> In production, I would make this a **GPO** for scale and consistency.

---

## Part 4 — Bulk User Creation via PowerShell

On `DC-1` (signed in as `jane_admin`):

1. Open **PowerShell ISE as Administrator**.
2. Create/open a new script file and paste your contents (e.g., `Generate-Names-Create-Users.ps1`).
3. Run it to create users in `_EMPLOYEES` (confirm in ADUC).
4. Attempt to sign in to `Client-1` as one of the newly created users.

*📸 **Insert: Script, ISE run output, users visible in ADUC***

> Example skeleton to place users into an OU (adapt to your script):
```powershell
Import-Module ActiveDirectory

$ou = "OU=_EMPLOYEES,DC=mydomain,DC=com"
$users = @(
  @{ GivenName="Ava"; Surname="Lopez"; Sam="ava.lopez"; Password="P@ssw0rd123!" },
  @{ GivenName="Noah"; Surname="Nguyen"; Sam="noah.nguyen"; Password="P@ssw0rd123!" }
)

foreach ($u in $users) {
  New-ADUser -Name "$($u.GivenName) $($u.Surname)" -GivenName $u.GivenName -Surname $u.Surname `
             -SamAccountName $u.Sam -UserPrincipalName "$($u.Sam)@mydomain.com" `
             -AccountPassword (ConvertTo-SecureString $u.Password -AsPlainText -Force) `
             -ChangePasswordAtLogon $true -Enabled $true -Path $ou
}
```

---

## Part 5 — Account Lockout / Enable / Disable / Reset

1. Pick a test user and **enter wrong passwords** repeatedly to trigger lockout (after policy is set).
2. Configure **Account Lockout Policy** via **GPMC** on `DC-1` (example values):
   - Threshold: **5** invalid attempts
   - Duration: **30 minutes**
   - Reset counter after: **15 minutes**
3. **Unlock** account, **Reset** password, **Re‑enable/Disable** as needed.
4. **Observe logs** on DC and Client (Event Viewer → Security) to validate lockout and sign‑in results.

*📸 **Insert: GPO editor for Account Lockout Policy, Event Viewer log screenshots***

---

## Demonstration

- **Join domain:** Show Client-1 domain join + ADUC computer object.
- **RDP access:** Sign in to Client-1 as a **non‑admin domain user**.
- **Bulk users:** Show ADUC populated with scripted users.
- **Lockout flow:** Show failed logons → lockout → unlock/reset → successful logon.
- **Logs:** Show Event IDs confirming activity on DC and Client.

*🎥 **Insert: short screen recording(s) or GIFs of the above***

---

## What I Learned

- Standing up **core Windows infrastructure** in Azure (networking + VMs + DNS).
- **AD DS** provisioning and secure administrative practices (separate admin account).
- **OU design** and **client domain joins** for device governance.
- **RDP configuration** for standard users and when to scale with **Group Policy**.
- **Bulk identity operations** with **PowerShell** and verifying outcomes in ADUC.
- Implementing **Account Lockout Policy** and validating through **Windows event logs**.

---

## Troubleshooting

- **Client can’t join domain:** Verify Client DNS points to **DC-1 private IP**; `ipconfig /flushdns` and restart.
- **Ping blocked:** If testing, temporarily **disable Windows Defender Firewall** on DC/Client; re‑enable after validation.
- **Script errors:** Ensure **ActiveDirectory** module is available and OU **distinguished name** is correct.
- **Policy not applying:** Run `gpupdate /force` or wait for replication; verify GPO link scope and precedence.
- **RDP denied:** Confirm `Domain Users` are in Remote Desktop Users and `Allow connections` is enabled.

---

## Next Steps

- Convert manual RDP configuration into a **GPO**.
- Add **fine‑grained password policies** and **delegation** for Help Desk.
- Enable **auditing** and forward logs to **Azure Monitor / Sentinel**.
- Add a **PowerShell Just‑Enough‑Administration (JEA)** sandbox for safer admin ops.

---

## Media Placeholders

- 📸 `/media/01-azure-overview.png`
- 📸 `/media/02-dc1-static-ip.png`
- 📸 `/media/03-client1-dns.png`
- 📸 `/media/04-adds-promotion.png`
- 📸 `/media/05-ous-and-admin.png`
- 📸 `/media/06-client-in-aduc.png`
- 📸 `/media/07-rdp-domain-users.png`
- 📸 `/media/08-bulk-users.png`
- 📸 `/media/09-lockout-gpo.png`
- 📸 `/media/10-event-logs.png`

> Replace the placeholders above with your actual screenshots/gifs.

---

## Repo Structure

```
.
├── README.md
├── scripts/
│   └── Generate-Names-Create-Users.ps1     # (add your script here)
└── media/                                   # (add screenshots/gifs here)
```

---

## How to Reproduce (Quick Start)

1. Deploy `DC-1` (Win Server 2022) and `Client-1` (Win10) in the same VNet.
2. Set `Client-1` DNS → `DC-1` private IP and restart.
3. Promote `DC-1` to a new forest `mydomain.com` and create `_EMPLOYEES`, `_ADMINS`, `_CLIENTS` OUs.
4. Create `jane_admin`, add to **Domain Admins**, and sign in as `mydomain\jane_admin`.
5. Join `Client-1` to the domain; move its object into `_CLIENTS` OU.
6. Allow **Domain Users** to RDP into `Client-1`.
7. Run the bulk user creation PowerShell script and validate in ADUC.
8. Configure **Account Lockout Policy**, test lockout/unlock/reset, and verify logs.

---

### License

MIT (or your preferred license)
