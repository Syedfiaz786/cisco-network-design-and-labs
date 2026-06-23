# Lab 01: Basic Switch Security and Management
## 🧠 Core Networking Concepts (Understand the Logic)

### Concept 1: Why We Change the Hostname & Disable DNS Lookup
* **Hostname:** Default switches par `Switch>` likha aata hai, jisse bade networks mein pehchan mushkil hoti hai. Hostname badalne se device ki actual location aur identity ka pata chalta hai.
* **No IP Domain-Lookup:** CLI par jab koi command galat type ho jaye, to switch use internet par dhoondne lagta hai aur switch 1-2 minute ke liye bilkul hang (lockout) ho jata hai. Is command se yeh hang hone wala masla hamesha ke liye khatam ho jata hai.

### Concept 2: Security Boundaries (Enable Secret & Console Password)
* **Enable Secret:** Yeh switch ke privileged mode ko lock karne ke liye MD5/SHA algorithm ke zariye encrypted password lagata hai taake koi aam user configurations na badal sake.
* **Console Password with Local Login:** Yeh physical terminal security hai. Jab koi direct console cable lagaye ga, to switch pehle username aur password maangega.

### Concept 3: The Purpose of a Banner (MOTD)
* **Message of the Day (MOTD):** Yeh device open hote hi sab se pehle screen par warning show karta hai. Legal taur par yeh zaroori hota hai taake kisi bhi unauthorized hacker ko warning di ja sakay ke yeh ek private property network hai.

### Concept 4: Layer 2 Management SVI (VLAN 99 vs VLAN 1)
* **Why not VLAN 1?** Out-of-the-box switch ke saare ports VLAN 1 mein hote hain. Agar management IP bhi isi par ho, to koi bhi laptop connect karke switch ka system open kar sakta hai.
* **VLAN 99 (Isolated Management):** Hum ne management ke liye ek alag kamra (VLAN 99) banaya. Jab tak kisi specific port (jaise Fa0/24) ko manually VLAN 99 mein nahi dala jaye ga aur wahan physical terminal signal nahi milega, tab tak management rasta (SVI Protocol) up nahi hoga.

### Concept 5: Why SSHv2 over Telnet?
* **Telnet:** Iska data clear-text (sada lafzon) mein jata hai jise packet sniffers (jaise Wireshark) se asani se chori kiya ja sakta hai.
* **SSH version 2:** RSA asymmetric encryption algorithm ka use karta hai. Yeh poore terminal session ko encrypt kar deta hai, jisse passwords aur commands network par mukammal secure rehti hain.

---

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
