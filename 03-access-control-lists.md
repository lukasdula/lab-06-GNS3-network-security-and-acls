
# **3 - Access Control List**

## **3.1 Introduction**

This chapter introduces **Access Control Lists (ACLs)** as a method to control communication between VLANs and enhance network security. ACLs define which devices or networks are allowed or denied access to certain resources.

The existing topology includes three VLANs: VLAN 10 for **Users**, VLAN 20 for **Guests**, and VLAN 99 for **Management**. The router (R1) performs inter-VLAN routing, and ACLs will be applied to manage access between these segments.

**The access policy for this lab is as follows:**

* **Admin (VLAN 99)** has access to all networks.
    
* **User (VLAN 10)** can access the Guest network but not the Management network.
    
* **Guest (VLAN 20)** has no access to other networks.
    

*This configuration demonstrates how ACLs can enforce network segmentation and protect critical management resources.*


![](images/Pasted%20image%2020251111212921.png)

## **3.2 Topology**

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


## **3.3 Steps Overview**

1. Create and define ACL on R1 according to the access policy. Include rules for Admin, User, and Guest VLANs, and allow ICMP for testing.
    
2. Apply ACL to the router subinterfaces that handle inter-VLAN routing.
    
3. Verify connectivity using ping and diagnostic commands.

## **3.4 ACL Configuration (R1)**

This step defines and configures Access Control Lists on the router R1 according to the network policy. The ACL will control access between VLANs and allow diagnostics while restricting who can initiate pings.

**Access Policy Summary:**

- Admin (VLAN 99) has access to all VLANs.
    
- User (VLAN 10) can access Guest (VLAN 20) but not Management (VLAN 99).
    
- Guest (VLAN 20) cannot access any other VLAN.
    
- ICMP (ping): Only Admin can initiate pings to others; others cannot initiate pings. Replies to Admin are allowed.
    

**Configuration:**

```prikazy
enable
configure terminal
ip access-list extended VLAN_CONTROL
permit ip 192.168.99.0 0.0.0.255 any
permit icmp 192.168.99.0 0.0.0.255 any echo
permit icmp any 192.168.99.0 0.0.0.255 echo-reply
permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
deny icmp 192.168.10.0 0.0.0.255 any echo
deny ip 192.168.10.0 0.0.0.255 192.168.99.0 0.0.0.255
deny icmp 192.168.20.0 0.0.0.255 any echo
deny ip 192.168.20.0 0.0.0.255 192.168.99.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 any
permit ip any any
exit
```
![](images/Pasted%20image%2020251111201022.png)

## **3.5 Apply ACL to Subinterfaces**

After the ACL is created, it is applied to the router interfaces that handle inter-VLAN routing. The ACL is applied in the inbound direction so that traffic is filtered when it enters the router.

### **Router (R1)**


```prikazy
interface gi0/0.10
 ip access-group VLAN_CONTROL in
exit

interface gi0/0.20
 ip access-group VLAN_CONTROL in
exit

interface gi0/0.99
 ip access-group VLAN_CONTROL in
exit
```
![](images/Pasted%20image%2020251111202359.png)


This setup keeps VLAN 10 and VLAN 20 traffic filtered according to the ACL rules as it enters the router. VLAN 99 keeps full management access and can use ping for testing. The ICMP rules allow only the Admin network to start pings, while other VLANs can only reply.


## **3.6 Troubleshooting**

**When testing ACLs, Admin (VLAN 99) could not ping any devices.** We found that ACL was not the problem -> the issue came from **port-security** on SW1.  

The command `show port-security` showed **36 violations** on Gi0/1, meaning the port was dropping frames from the Admin PC.  

 ```
 show port-security
 ```
![](images/Pasted%20image%2020251111210251.png)

The cause was an old sticky MAC (`0050.7966.6803`) saved during previous testing in chapter two. 

 ```
 SW1#show port-security address
 ```
![](images/Pasted%20image%2020251111210634.png)

The **MAIN** Admin VPC had a different original MAC (`0050.7966.6801`). 

```
show ip
```
![](images/Pasted%20image%2020251111210731.png)

After removing the old sticky MAC entry, the correct address was automatically re-learned once the Admin PC sent traffic, restoring full connectivity and proper ACL function.

**Command used to fix:**

```prikazy
interface gi0/1
shutdown
no switchport port-security mac-address sticky 0050.7966.6803
no shutdown
end
```

**Verification:**

```prikazy
show port-security address
```
![](images/Pasted%20image%2020251111211205.png)

```
show port-security 
```
![](images/Pasted%20image%2020251111211249.png)

After the correction, the ACL policy operated exactly as intended -> the Admin network regained full access, while User and Guest VLANs followed their restricted rules.


## **3.7 Verification – ACL Connectivity Tests**

The following ping tests confirm that ACL rules are applied correctly and communication between VLANs follows the intended access policy.

**1. Admin (VLAN 99) → User (VLAN 10) and Guest (VLAN 20)**  

* Ping from Admin to User and Guest works successfully, confirming that the Admin network has full access to all VLANs.

```prikazy
Admin> 192.168.10.10
Admin> 192.168.20.10
```
![](images/Pasted%20image%2020251111212044.png)

**2. Guest (VLAN 20) → User (VLAN 10) and Admin (VLAN 99)**  

* Ping from Guest to User and Admin fails, proving that the Guest network is fully restricted and cannot initiate communication with other VLANs.

```prikazy
Guest> ping 192.168.10.10
Guest> ping 192.168.99.10
```
![](images/Pasted%20image%2020251111212419.png)

**3. User (VLAN 10) → Guest (VLAN 20) and Admin (VLAN 99)**  

* Ping from User to Guest works, but ping to Admin fails. This confirms that the User VLAN can reach Guest but is blocked from accessing the Management VLAN.

```prikazy
User> ping 192.168.20.10
User> ping 192.168.99.10
```
![](images/Pasted%20image%2020251111212249.png)

## 3.8 **Conclusion**

Access Control Lists control communication between VLANs as designed.  
The Admin VLAN (99) has full access, the User VLAN (10) is limited to the Guest VLAN (20), and the Guest VLAN (20) remains fully isolated.  
After fixing the port-security issue on SW1, all connectivity tests confirm that the ACL policy works correctly.

