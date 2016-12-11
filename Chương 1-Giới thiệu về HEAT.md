#Chương 1 : Giới thiệu về HEAT
***
#MỤC LỤC  
[1. Khái niệm](#1)  
[2. Các thành phần của HEAT](#2)  
[3. Các khái niệm cần biết](#3)  
[4. Template of Heat](#4)  
[5. Cài đặt Heat trong OpenStack bản mitaka](#5)

***  

<a name="1"></a>
##1. Khái niệm 
Heat là sự phối hợp giữa các project của OpenStack . 


Heat cung cấp khả năng cho phép người dùng tự định nghĩa ứng dụng của mình thông qua template ( VD : tạo network , router , instance , ... )

<a name="2"></a>
## 2.Các thành phần của HEAT
The Orchestration service gồm các thành phần :  
 
- `heat` command-line client 
- `heat-apt` component
- `heat-apt-cfn` component
- `heat-engine`

<a name="3"></a>
##3.Các khái niệm cần biết

- **Stack** : Trong cách nói của Heat , stack là 1 tập hợp các objects , 1 tập hợp các resource , sẽ được taoj bởi Heat . Điều này bao gồm các instance (VMs) , networks , subnets , routers , ports , router interface , security group , ecurity group rules, auto-scaling rules, etc...
- **Template** : Heat sử dụng 1 template để định nghĩa stack . 
- **1 Heat template có 3 sections chính** :
  - **Parameters** : Đây là thông tin như image ID , network ID ,…
  - **Resource** : là các objects cụ thể mà Heat sẽ tạo ra .
  - **Output** : thông tin được chuyển qua cho người sử dụng , thông qua OpenStack DardBoard hoặc `heat stack-list` and `heat stack-show` commands .  

<a name="4"></a>
##4.Template of HEAT
- **Heat hỗ trợ  2 loại template là HOT và CFN** :
  - **CFN** (CloudFormation-compatible format ) : là định dạng được hỗ trợi bởi Heat trong thời gian qua . CFN được viết dưới dạng JSON .
VD : AWS dùng định dạng CFN ( AWS CloudFormation template ) để cho phép người dùng tạo và quản lí các tài nguyên .
  - **HOT** (Heat Orchestration Template ) : là 1 định dạng template mới thay thế cho CFN . Heat input in the format native to OpenStack . 
HOT được viết dưới dạng YAML .
HOT được hỗ trợ từ phiên bản OPS Icehouse .  

<a name="5"></a>
##5.Cài đặt Heat trong OpenStack bản mitaka
Làm theo hướng dẫn trong link sau :   
<http://docs.openstack.org/mitaka/install-guide-ubuntu/heat-install.html>
