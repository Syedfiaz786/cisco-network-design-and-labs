# Lab 01: Basic Switch Security and Management
## 🧠 Core Networking Concepts (Understand the Logic)

### Concept 1: Why We Change the Hostname & Disable DNS Lookup
* **Hostname:** Default switches display `Switch>`, which makes identification difficult in large networks. Changing the hostname reveals the actual location and identity of the device.
* **No IP Domain-Lookup:** When a command is mistyped on the CLI, the switch starts searching for it on the internet/DNS, causing the switch to completely hang (lockout) for 1–2 minutes. This command eliminates this hanging issue permanently.

### Concept 2: Security Boundaries (Enable Secret & Console Password)
* **Enable Secret:** This applies an encrypted password using MD5/SHA algorithms to lock the switch's privileged mode, ensuring regular users cannot modify configurations.
* **Console Password with Local Login:** This enforces physical terminal security. When someone directly connects a console cable, the switch will first demand a username and password.

### Concept 3: The Purpose of a Banner (MOTD)
* **Message of the Day (MOTD):** This displays a warning on the screen immediately when the device session opens. Legally, this is necessary to warn any unauthorized hacker that this is a private property network.

### Concept 4: Layer 2 Management SVI (VLAN 99 vs VLAN 1)
* **Why not VLAN 1?** Out-of-the-box, all switch ports belong to VLAN 1. If the management IP is also on it, anyone connecting a laptop can open the switch's management system.
* **VLAN 99 (Isolated Management):** We created a separate room (VLAN 99) for management. Until a specific port (like Fa0/24) is manually assigned to VLAN 99 and receives a physical terminal signal, the management path (SVI Protocol) will not come up.

### Concept 5: Why SSHv2 over Telnet?
* **Telnet:** Its data travels in clear-text (plain words), which can easily be stolen using packet sniffers (like Wireshark).
* **SSH version 2:** It utilizes the RSA asymmetric encryption algorithm. This encrypts the entire terminal session, keeping passwords and commands completely secure over the network.
---
```text
## 🛠️ Step-by-Step Configuration Script (Office_Switch)

### Step 1: Set the switch hostname
Switch> enable
Switch# configure terminal
Switch(config)# hostname Office_Switch

### Step 2: Run the command no ip domain-lookup
Office_Switch(config)# no ip domain-lookup

### Step 3: Set privilege secret
Office_Switch(config)# enable secret class

### Step 4: Set banner on the opening of device
Office_Switch(config)# banner motd # Unauthorized Access Prohibited#

### Step 5: Set Console Password
Office_Switch(config)# username admin privilege 15 secret admin123
Office_Switch(config)# line console 0
Office_Switch(config-line)# login local
Office_Switch(config-line)# exit

### Step 6: Make Management Switch Virtual Interface (SVI)
! Create management VLAN database entry
Office_Switch(config)# vlan 99
Office_Switch(config-vlan)# name Management
Office_Switch(config-vlan)# exit

! Assign IP address to the Virtual Interface
Office_Switch(config)# interface vlan 99
Office_Switch(config-if)# ip address 192.168.1.10 255.255.255.0
Office_Switch(config-if)# no shutdown
Office_Switch(config-if)# exit

! Assign physical host access port to activate SVI Line Protocol
Office_Switch(config)# interface fastEthernet 0/24
Office_Switch(config-if)# switchport mode access
Office_Switch(config-if)# switchport access vlan 99
Office_Switch(config-if)# no shutdown
Office_Switch(config-if)# exit

! Map Default Gateway for the switch infrastructure
Office_Switch(config)# ip default-gateway 192.168.1.1

! Hardening: Disable default VLAN 1 IP interface
Office_Switch(config)# interface vlan 1
Office_Switch(config-if)# no ip address
Office_Switch(config-if)# shutdown
Office_Switch(config-if)# exit

### Step 7: Configure SSH
Office_Switch(config)# ip domain-name thenetworkarchitect.local
Office_Switch(config)# crypto key generate rsa
! Note: Enter 1024 or 2048 modulus bits when prompted by the CLI

Office_Switch(config)# ip ssh version 2
Office_Switch(config)# line vty 0 4
Office_Switch(config-line)# login local
Office_Switch(config-line)# transport input ssh
Office_Switch(config-line)# exit

### Step 8: Save all in nvram
Office_Switch(config)# exit
Office_Switch# copy running-config startup-config
