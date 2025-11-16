# **2 – Securing Switch Ports**

<br><br>

## **2.1 Introduction**

This chapter focuses on securing switch access ports to protect the network from unauthorized or fake devices. Each access port accepts only one MAC address, which is automatically learned and stored as a sticky entry. If a violation occurs, the port restricts traffic from unknown devices. VLAN 99 is used for management access, while VLAN 10 and VLAN 20 serve user and guest networks. The chapter also includes a demonstration of disabling unused ports as part of good security practice.


![](images/Pasted%20image%2020251111184750.png)

<br><br>

## **2.2 Topology**

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

<br><br>

## **2.3 Steps**


1. Enable port security on access ports and allow only one device per port.
    
2. Test the configuration by generating traffic and verifying learned MAC addresses.
    
3. Demonstrate how unused ports are administratively disabled to enhance network security.

<br><br>

## **2.4 Port Security Configuration (SW1)**

Port security helps prevent unauthorized or fake devices from connecting by limiting the number of devices that can connect to a switch port.

- Enable port security on access ports.
    
- Limit the number of allowed MAC addresses per port.
    
- Configure the action to take if a violation happens.
    

### **Switch (SW1)**

```
enable
configure terminal

interface range gi0/1 - 3
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 no shutdown
 exit

end
write memory
```
![](images/Pasted%20image%2020251111172344.png)

### **Diagnostics – Port Security**

* Verify port security status and learned MAC addresses.
    

```plaintext
show port-security
```
![](images/Pasted%20image%2020251111172637.png)

### **Verification after Traffic Test**

After performing a ping from **User** (VLAN 10) to **Admin** (VLAN 99), the switch learned the MAC addresses dynamically through port security. 

```plaintext
show port-security address
```
![](images/Pasted%20image%2020251111174215.png)

The command output confirms that each VLAN has one secure sticky MAC address assigned to its respective interface.

<br><br>

## **2.5 Administrative Shutdown of Unused Ports**

In this lab, the switch **has only four GigabitEthernet interfaces (Gi0/0–Gi0/3)**, all of which are already in use.  
However, for demonstration purposes, the following example shows **how unused ports are administratively disabled** in a larger switch environment to prevent unauthorized access.

### **Switch (SW1)**

 ```
enable
configure terminal
interface range gi0/4-gi0/24
shutdown
description UNUSED_PORT
exit
end
wr
 ```

<br><br>

## 2.6 **Conclusion**

This chapter presents the basic concept of switch port protection. Port security limits device access, applies network segmentation, and provides stable and controlled switch operation.


---

Next part:  [Access Control Lists](03-access-control-lists.md)
