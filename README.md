# Enterprise Active Directory Home Lab (VirtualBox)

## Goal
Build a small enterprise-style Windows environment to practice:
- Active Directory (users, OUs, GPO)
- DHCP + NAT routing for client internet access
- Domain join + login with domain accounts
- PowerShell automation (bulk user creation)
- Troubleshooting (DHCP/DNS/domain join issues)

---

## Lab Architecture (High-Level)
- **DC01** (Windows Server 2019/2022)
  - NIC 1: NAT (Internet)
  - NIC 2: Internal Network (Lab LAN)
  - Roles: AD DS, DNS, DHCP, RRAS (NAT)
- **CLIENT01** (Windows 10/11 Pro)
  - NIC: Internal Network (Lab LAN)
  - Joined to domain

Diagram: `diagrams/network-diagram.png`

---

## Network Plan
Internal Lab LAN:
- DC01 (Internal NIC): `172.16.0.1/24`
- DHCP Scope: `172.16.0.100 - 172.16.0.200`
- Default Gateway (DHCP option): `172.16.0.1`
- DNS (DHCP option): `172.16.0.1` (or `127.0.0.1` on DC)

Details: `configs/ip-plan.md`

---

## Step-by-Step Build (Concise)

### 1) Install VirtualBox + Extension Pack
- Install VirtualBox
- Install VirtualBox Extension Pack

### 2) Download ISOs
- Windows Server 2019/2022 ISO
- Windows 10/11 Pro ISO

### 3) Create DC01 VM (Domain Controller)
- Name: `DC01`
- RAM: 2–4 GB (as available)
- CPU: 2–4 cores (as available)
- Network:
  - Adapter 1: **NAT**
  - Adapter 2: **Internal Network** (name it: `intnet`)
- Install Windows Server

### 4) Install Guest Additions (quality of life)
- Devices → Insert Guest Additions CD Image
- Install and reboot

### 5) Configure NICs + Static IP (Internal)
- Rename NICs:
  - NAT NIC: `Internet`
  - Internal NIC: `Internal`
- Set static IPv4 on **Internal** NIC:
  - IP: `172.16.0.1`
  - Mask: `255.255.255.0`
  - Gateway: *(blank)*
  - DNS: `127.0.0.1` *(or `172.16.0.1`)*

### 6) Rename Server + Promote to Domain Controller
- Rename computer to `DC01`
- Add Roles: **AD DS**
- Promote to Domain Controller:
  - New forest: `mydomain.local` *(example)*
- Reboot

### 7) Configure RRAS (NAT)
- Add Roles: **Remote Access** → **Routing**
- Tools → Routing and Remote Access
- Configure NAT:
  - Public interface: `Internet` NIC
  - Private interface: `Internal` NIC

### 8) Configure DHCP
- Add Role: **DHCP Server**
- Create DHCP Scope:
  - Range: `172.16.0.100 - 172.16.0.200`
  - Router (Option 003): `172.16.0.1`
  - DNS (Option 006): `172.16.0.1`
- Authorize DHCP server

### 9) Create CLIENT01 VM
- Name: `CLIENT01`
- Network:
  - Adapter 1: **Internal Network** (`intnet`)
- Install Windows 10/11 **Pro** (Home cannot domain join)

### 10) Verify DHCP + Internet on Client
On CLIENT01:
- `ipconfig /all`
- Confirm:
  - IP in scope (`172.16.0.x`)
  - Default gateway `172.16.0.1`
  - DNS `172.16.0.1`
- Test:
  - `ping 8.8.8.8`
  - `ping google.com`
  - `ping mydomain.local`

### 11) Join CLIENT01 to Domain
- Rename to `CLIENT01`
- Join domain: `mydomain.local`
- Reboot
- Login using domain account

### 12) Bulk User Creation (PowerShell)
- Scripts in: `scripts/`
- Run:
  - `Create-Employees.ps1`
  - Uses `names.txt`

---

## Evidence / Screenshots
Stored in: `docs/screenshots/`
- DC NIC config (internal static IP)
- AD DS role installed + domain created
- RRAS NAT config
- DHCP scope options
- CLIENT01 IP config
- Domain join success
- Domain login proof

---

## Scripts
- `scripts/Create-Employees.ps1` — creates OU + users from `names.txt`
- `scripts/names.txt` — list of names used

---

## Troubleshooting Notes
See: `docs/troubleshooting.md`
Common issues:
- Client gets IP but no gateway (DHCP option 003 missing)
- DNS not resolving (client not using DC DNS)
- Domain join fails (wrong DNS / using Windows Home)
