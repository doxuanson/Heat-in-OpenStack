#Chýõng 1 : Gi?i thi?u v? HEAT
***
#M?C L?C  
[1. Khái ni?m](#1)  
[2. Các thành ph?n c?a HEAT](#2)  
[3. Các khái ni?m c?n bi?t](#3)  
[4. Template of Heat](#4)
[5. Cài ð?t Heat trong OpenStack b?n mitaka](#5)

***  
<a name="1"></a>
##1. Khái ni?m 
Heat là s? ph?i h?p gi?a các project c?a OpenStack . Heat cung c?p kh? nãng cho phép ngý?i dùng t? ð?nh ngh?a ?ng d?ng c?a m?nh thông qua template ( VD : t?o network , router , instance , ... )

<a name="2"></a>
## 2.Các thành ph?n c?a HEAT
The Orchestration service g?m các thành ph?n :  
 
- `heat` command-line client 
- `heat-apt` component
- `heat-apt-cfn` component
- `heat-engine`

<a name="3"></a>
##3.Các khái ni?m c?n bi?t

- **Stack** : Trong cách nói c?a Heat , stack là 1 t?p h?p các objects , 1 t?p h?p các resource , s? ðý?c taoj b?i Heat . Ði?u này bao g?m các instance (VMs) , networks , subnets , routers , ports , router interface , security group , ecurity group rules, auto-scaling rules, etc...
- **Template** : Heat s? d?ng 1 template ð? ð?nh ngh?a stack . 
- **1 Heat template có 3 sections chính** :
  - **Parameters** : Ðây là thông tin nhý image ID , network ID ,…
  - **Resource** : là các objects c? th? mà Heat s? t?o ra .
  - **Output** : thông tin ðý?c chuy?n qua cho ngý?i s? d?ng , thông qua OpenStack DardBoard ho?c `heat stack-list` and `heat stack-show` commands .  

<a name="4"></a>
##4.Template of HEAT
- **Heat h? tr?  2 lo?i template là HOT và CFN** :
  - **CFN** (CloudFormation-compatible format ) : là ð?nh d?ng ðý?c h? tr?i b?i Heat trong th?i gian qua . CFN ðý?c vi?t dý?i d?ng JSON .
VD : AWS dùng ð?nh d?ng CFN ( AWS CloudFormation template ) ð? cho phép ngý?i dùng t?o và qu?n lí các tài nguyên .
  - **HOT** (Heat Orchestration Template ) : là 1 ð?nh d?ng template m?i thay th? cho CFN . Heat input in the format native to OpenStack . 
HOT ðý?c vi?t dý?i d?ng YAML .
HOT ðý?c h? tr? t? phiên b?n OPS Icehouse .  

<a name="5"></a>
##5.Cài ð?t Heat trong OpenStack b?n mitaka
Làm theo hý?ng d?n trong link sau :   
<http://docs.openstack.org/mitaka/install-guide-ubuntu/heat-install.html>
