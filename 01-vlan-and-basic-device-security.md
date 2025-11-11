# **01 – Vlan and Basic Device Security**


## 1.1 **Introduction**  

This lab focuses on securing Cisco network devices at the fundamental level. VLAN segmentation is created at the beginning to separate users, guests, and management areas. These VLANs will later be used to demonstrate Access Control Lists (ACLs) and controlled access between different parts of the network. The main goal of this phase is to establish basic device protection such as password encryption, secure login, and administrative management through a dedicated VLAN.



![](images/Pasted%20image%2020251111181956.png)

### **Objectives**

- Create VLANs to separate users, guests, and management traffic.
    
- Configure console and privileged access security.
    
- Enable password encryption on all stored credentials.
    
- Display a login banner with access warning.
    
- Disable unused switch ports administratively.
    
- Set up a management VLAN (VLAN 99) for administrative access.


### **VLAN Overview**

|Device|VLAN ID|Name|Subnet|Purpose|
|---|---|---|---|---|
|Admin (VPCS)|99|Management|192.168.99.0/24|Administrative access and network management|
|User (VPCS)|10|Users|192.168.10.0/24|Internal user workstation network|
|Guest (VPCS)|20|Guests|192.168.20.0/24|Isolated guest network|
|SW1|99|Management|192.168.99.0/24|Switch management interface|

> **Note:**  
> VLAN 99 is used exclusively for administrative access.  
> The switch has a management IP address within this VLAN, and the router provides the default gateway (192.168.99.1) for all management devices.


## **1.2 Topology**

| Device           | Type         | Interface | Connected to → | Peer Interface | IP Address    | Subnet Mask   | Gateway      |
| ---------------- | ------------ | --------- | -------------- | -------------- | ------------- | ------------- | ------------ |
| **R1**           | Subinterface | G0/0.10   | SW1            | Gi0/0          | 192.168.10.1  | 255.255.255.0 | –            |
| **R1**           | Subinterface | G0/0.20   | SW1            | Gi0/0          | 192.168.20.1  | 255.255.255.0 | –            |
| **R1**           | Subinterface | G0/0.99   | SW1            | Gi0/0          | 192.168.99.1  | 255.255.255.0 | –            |
| **SW1**          | Access Port  | Gi0/1     | User (VPCS)    | e0             | –             | –             | 192.168.10.1 |
| **SW1**          | Access Port  | Gi0/2     | Guest (VPCS)   | e0             | –             | –             | 192.168.20.1 |
| **SW1**          | Access Port  | Gi0/3     | Admin (VPCS)   | e0             | 192.168.99.2  | 255.255.255.0 | 192.168.99.1 |
| **User (VPCS)**  | Host         | e0        | SW1            | Gi0/2          | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| **Guest (VPCS)** | Host         | e0        | SW1            | Gi0/3          | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| **Admin (VPCS)** | Host         | e0        | SW1            | Gi0/1          | 192.168.99.10 | 255.255.255.0 | 192.168.99.1 |


## 1.3 **Steps**

1. Configure static IP addresses on all VPCS devices (Admin, User, Guest) according to the addressing plan. **This step is not documented** as it follows the predefined topology table.
    
2. Create VLANs on the switch, assign access ports for user, guest, and management devices, and configure VLAN 99 for administrative access.
    
3. Configure trunk connection between the router and switch.
    
4. Set up router subinterfaces for inter-VLAN communication.
    
5. Secure privileged and console access with passwords and encryption.
    
6. Display a login banner for authorized access.
    

## **1.4 VLAN Configuration and Port Assignment (SW1)****

VLANs segment the network and assign devices to specific broadcast domains. Each VLAN corresponds to its access port, and a dedicated management VLAN provides administrative access to the switch.

- Create VLANs with names.
    
- Assign access ports to the appropriate VLANs.
    
- Configure management VLAN for switch administration.
    
- Save configuration.
    

### **Switch (SW1)**

```plaintext
enable
configure terminal

vlan 10
name User
exit
vlan 20
name Guest
exit
vlan 99
name Admin
exit

interface gi0/2
switchport mode access
switchport access vlan 10
no shutdown
exit

interface gi0/3
switchport mode access
switchport access vlan 20
no shutdown
exit

interface gi0/1
switchport mode access
switchport access vlan 99
no shutdown
exit

interface vlan 99
ip address 192.168.99.2 255.255.255.0
no shutdown
exit 
ip default-gateway 192.168.99.1
end
write memory
```
![](images/Pasted%20image%2020251111153346.png)


## **1.5 Trunk Configuration

Trunk carries multiple VLANs between the switch and the router. The router uses subinterfaces to route between VLANs; the physical interface stays without an IP.

- Set the SW1 uplink as an 802.1Q trunk and allow required VLANs.
    

### **Switch (SW1)**

```prikazy
configure terminal
interface gi0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,99
no shutdown
end
write memory
```
![](images/Pasted%20image%2020251111154959.png)

### **Diagnostics – SW1**

1. Verify trunk configuration and allowed VLANs.

```plaintext
show interfaces trunk
```
![](images/Pasted%20image%2020251111155139.png)

2. Confirm VLAN assignment and port status.

```plaintext
show vlan brief
```
![](images/Pasted%20image%2020251111155153.png)

## **1.6 Router Subinterface Configuration (R1)**

Subinterfaces on R1 enable inter-VLAN routing across the trunk link connected to SW1. Each VLAN has its own virtual interface with an IP address serving as the default gateway for that VLAN.

    
- Create one subinterface per VLAN and configure 802.1Q encapsulation.
    
- Assign IP addresses according to the topology plan.

### **Router (R1)**

```prikazy
enable
configure terminal

interface gi0/0
no shutdown
exit

interface gi0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit

interface gi0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit

interface gi0/0.99
encapsulation dot1Q 99
ip address 192.168.99.1 255.255.255.0
exit

end
write memory
```
![](images/Pasted%20image%2020251111160319.png)

### **Diagnostics – Subinterfaces**

* Verify that all subinterfaces are up and correctly configured.

```plaintext
show ip interface brief
```
![](images/Pasted%20image%2020251111160359.png)


## **1.7 Securing Console and Privileged Access**

This configuration secures device access by setting privileged mode and console passwords, creating a local admin user, enabling password encryption, and applying basic login protection.

- Configure an encrypted enable secret password.
    
- Create a local user account for administrative access.
    
- Set console password protection with login requirement.
    
### **Switch (SW1)**

```prikazy
enable
configure terminal

service password-encryption

enable secret cisco

username admin privilege 15 secret admin12345

line console 0
 password 12345
 login local
 exit

end
write memory
```
![](images/Pasted%20image%2020251111161536.png)

### **Router (R1)**

```prikazy
enable
configure terminal

service password-encryption

enable secret cisco

username admin privilege 15 secret admin12345

line console 0
 password 12345
 login local
 exit

end
write memory
```
![](images/Pasted%20image%2020251111161802.png)


> **Notes.:** The configuration includes both `password` and `secret` for demonstration purposes. In practice, `secret` always takes precedence over `password` and stores the value in encrypted form.


## **1.8 Login Banner Configuration**

A Message of the Day (MOTD) banner displays a legal or administrative notice before login. It helps identify authorized use policies and discourages unauthorized access attempts.

- Configure a message of the day banner on all devices.
    
- Include a standard warning message.

### **Switch (SW1)**

```prikazy
enable
configure terminal
banner motd #
Unauthorized access is prohibited.
#
end
write memory
```
![](images/Pasted%20image%2020251111165734.png)
### **Router (R1)**

```prikazy
enable
configure terminal
banner motd #
Unauthorized access is prohibited.
#
end
write memory
```
![](images/Pasted%20image%2020251111165932.png)
### **Diagnostics – MOTD Banner**

* Verify the banner message configuration.

```plaintext
show running-config
```
![](images/Pasted%20image%2020251111170108.png)


### **1.9 Conclusion**

This chapter demonstrates the configuration of VLAN segmentation and basic device security on Cisco network equipment. VLANs separate user, guest, and management traffic, while router subinterfaces provide inter-VLAN routing through an 802.1Q trunk connection. Device access is secured with encrypted passwords, a local administrator account, and a login banner to identify authorized access. All configurations are verified to ensure correct VLAN operation, trunk connectivity, and reliable communication between network segments.


---

Next part:















