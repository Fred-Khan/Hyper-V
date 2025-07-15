## 🧑‍🏫 **Course Title**: Hyper-V on Windows 10: Practical VM Management for VMWare Users

### 🎯 Audience Profile
- Comfortable with **VMWare Workstation** workflows.
- Experienced in basic **Windows 10** configuration and powershell commands.
- Knowledge in configuring networking adapters.

---

## 🧩 **Module 1: Getting Acquainted with Hyper-V on Windows 10**
### 💡 Conceptual Shift from VMWare Workstation
| Feature                     | VMWare Workstation            | Hyper-V on Windows 10                   |
|----------------------------|-------------------------------|-----------------------------------------|
| Installation Mode          | Desktop App                   | Windows Feature (via "Turn Windows features on or off") |
| Hypervisor Type            | Type 2                        | Type 1 (client-side native via Windows OS kernel) |
| Graphical Interface        | Full GUI + hardware emulation | Hyper-V Manager + PowerShell            |
| Networking Configuration   | Virtual Network Editor        | Virtual Switch Manager                  |
| Snapshots / Restore Points| VMWare Snapshots              | Hyper-V Checkpoints                     |

### ⚙️ Enable Hyper-V on Windows 10 Professional (If not already done):
- Go to `Control Panel → Programs → Turn Windows features on or off` (For Windows 10 Home you will need to run a batch file in command prompt as Admin)
- Tick ✅ **Hyper-V** and its subcomponents
- Restart your PC

---

## 🌐 **Module 2: Networking and NAT with Hyper-V on Windows 10**
### 🛜 Understanding Switch Types
- **External**: Connects VMs to your real network — useful for DHCP/remote access
- **Internal**: VMs communicate with the host only
- **Private**: Isolated VM-only communication (no host)

### 🏗️ NAT Subnet Setup for Internet Access
Windows 10 Hyper-V doesn’t have GUI options for NAT—so we use PowerShell:

#### 🔄 PowerShell Configuration (Start a PowerShell console as Administrator)
```powershell
# View current adapters with "Get-NetAdapter" and pipe to Format table to size all columns to be visible
Get-NetAdapter | Format-Table -AutoSize

# Create Internal Switch
New-VMSwitch -SwitchName "InternalNATSwitch" -SwitchType Internal

# Confirm that the Internal Switch was created successfully
Get-NetAdapter | Format-Table -AutoSize

# Assign IP to virtual switch
New-NetIPAddress -IPAddress 192.168.50.1 -PrefixLength 24 -InterfaceAlias "vEthernet (InternalNATSwitch)"

# Create NAT Gateway
New-NetNat -Name "InternalNAT" -InternalIPInterfaceAddressPrefix 192.168.50.0/24
```

This allows all connected VMs to use your host's internet via NAT, like VMWare’s built-in NAT adapter.

#### 💡 NOTE: Hyper-V does not provide DHCP by default. This is it’s expected behavior. If you want automatic IP assignment like the Default Switch, you can run a DHCP server inside a VM or use a third-party DHCP service on your host.

---

## 🖥️ **Module 3: Creating and Configuring a VM**
### 🔧 Steps via Hyper-V Manager GUI
1. **Launch Hyper-V Manager** (search from Start menu)
2. Click **New → Virtual Machine**
3. Set a name like `Win10LabVM`
4. Generation: Choose **Generation 2** (for modern OS)
5. Assign RAM (e.g. 4096MB) — ensure “dynamic memory” is ticked for efficiency
6. Connect to your **InternalNATSwitch**
7. Load your ISO image (use a local file path for a Windows 10 ISO)
8. Complete setup and start the VM
9. Inside VM: configure static IP or DHCP under `192.168.50.x` range

---

## 🧪 **Exercise: Create a Windows 10 VM with NAT Internet Access**
### 🎓 Goal
Reinforce your Hyper-V workflow by building a VM and connecting it to the internet via custom NAT.

### ✅ Task List
1. Create Internal NAT Switch using PowerShell
2. Configure a VM using Hyper-V Manager with ISO
3. Set VM to use NAT switch
4. Boot VM and test internet access (e.g. browser ping to external site)

---

## 💡 Tips & Recommendations
- ⚙️ Enable **Enhanced Session Mode** for better UI and device sharing
- 🧼 Use **Checkpoints** during experimentation to revert easily
- 📁 Keep ISO images and VM folders organized in a dedicated directory (e.g., `C:\HyperV\VMs`)
- 🚀 Use **PowerShell** for automating repetitive VM deployments if scaling a lab
- 🎯 For Windows 10 clients, avoid running VMWare Workstation and Hyper-V simultaneously — they conflict due to virtualization backend (Hyper-V takes control of VT-x)

---
You should be able to delete the switches you created via the Virtual Switch Manager. If not, use the commands below via Powershell. 
---

## 🧹 Delete Internal NAT Switch and NAT Gateway via PowerShell

### 🔌 1. Remove the NAT Gateway
```powershell
Remove-NetNat -Name "InternalNAT"
```
> This removes the NAT translation rules associated with the subnet.

---

### 🔧 2. Remove the Internal Virtual Switch
```powershell
Remove-VMSwitch -Name "InternalNATSwitch" -Force
```
> The `-Force` flag ensures removal even if the switch is in use.

---

### 🧼 3. (Optional) Clean Up IP Configuration
If you manually assigned an IP to the virtual adapter:
```powershell
Get-NetIPAddress -InterfaceAlias "vEthernet (InternalNATSwitch)" | Remove-NetIPAddress -Confirm:$false
```

---

## 🧠 Further Reading

<a href="https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/hyper-v-overview?pivots=windows" target="_blank">Hyper-V Overview - Microsoft Learn</a>
