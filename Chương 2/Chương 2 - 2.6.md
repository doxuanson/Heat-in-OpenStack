#Chương 2 : HOT Template Guide 
***
#Mục lục  
[2.6.Template](#2.6)  
- [2.6.1.Create a network and a subnet](#2.6.1)  
- [2.6.2.Create a port](#2.6.2)
- [2.6.3.Create a router and add subnet to router](2.6.3)  
- [2.6.4.Create a Floating IP](2.6.4)  
- [2.6.5. Associate a floating IP to an instance](2.6.5)  
- [2.6.6.Create a private-network hoàn chỉnh](2.6.6)  
- [2.6.7.Create a instance](2.6.7)  
- [2.6.8.Create a instance with network and floating_ip](2.6.8)  



***  

<a name="2.6"></a>  
##2.6.Template  
<a name="2.6.1"></a>  
###2.6.1.Create a network and a subnet  

```sh
heat_template_version: 2016-04-08

description: >
  Create a new network with subnet 
  subnet address is "10.10.10.0/24"

parameters:
  cidr:
    type: string
    label: Network CIDR
    description: The CIDR of the private network.
    default: '10.10.10.0/24'
  dns:
    type: comma_delimited_list
    label: DNS nameservers
    description: Comma separated list of DNS nameservers for the private network.
    default: '8.8.8.8'
  
resources:
  new_network:
    type: OS::Neutron::Net 
  new_subnet:
    type: OS::Neutron::Subnet 
    properties:
      network: { get_resource: new_network }
      cidr: { get_param: cidr}
      dns_nameservers: { get_param: dns}  

outputs:
  new_network_show:
    description: Info network
    value: {get_attr: [new_network , show]}
  new_subnet_show:
    description: Info subnet
    value: {get_attr: [new_subnet , show]}      

```  

<a name="2.6.2"></a>  
###2.6.2.Create a port  

```sh
heat_template_version: 2016-04-08

description: >
  Create a port

resources:
  new_port:
    type: OS::Neutron::Port
    properties:
      network: private-test
      fixed_ips:
        - subnet_id: "private-test-subnet"

outputs:
  new_port_show:
    description: Info port
    value: {get_attr: [new_port , show]}
```  

<a name="2.6.3"></a>
###2.6.3.Create a router and add subnet to router  
####1.Create a router  

```sh
heat_template_version: 2016-04-08

description: >
  Create a new router 

parameters:
  public_network:
    type: string
    label: Public network name or ID
    description: Public network with floating IP addresses.
    default: external

resources:
	new_router:   
    type: OS::Neutron::Router
    properties:
      external_gateway_info: { network: {get_param: public_network} }

outputs:
  new_router_show:
    description: Info router
    value: {get_attr: [new_router , show] }
```  

####2.Add subnet to router  

```sh
heat_template_version: 2016-04-08

description: >
  Create a new router with name : newrouter 
  add subnet to router

parameters:
  public_network:
    type: string
    label: Public network name or ID
    description: Public network with floating IP addresses.
    default: external

resources:
  new_router:   
    type: OS::Neutron::Router
    properties:
      external_gateway_info: { network: {get_param: public_network} }

  subnet_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router: { get_resource: new_router }
      subnet: private-test-subnet   

outputs:
  new_router_show:
    description: Info router
    value: {get_attr: [new_router , show] }

 ```  


<a name="2.6.4"></a>  
###2.6.4.Create a Floating IP  


 ```sh
 heat_template_version: 2015-04-30

resources:
  new_FloatingIP:
    type: OS::Nova::FloatingIP
    properties:
      pool: external

outputs:
  new_FloatingIP_show:
    description: Info FloatingIP
    value: {get_attr: [new_FloatingIP , show]}

```  

<a name="2.6.5"></a>  
###2.6.5. Associate a floating IP to an instance  

```sh
heat_template_version: 2015-04-30

resources:

  the_resource:
    type: OS::Nova::FloatingIPAssociation
    properties:
      floating_ip: 100.100.100.10
      server_id: id_VM  

```  


<a name="2.6.6"></a>
###2.6.6.Create a private-network hoàn chỉnh  

```sh
heat_template_version: 2016-04-08

description: Simple template to create a private network

parameters:
  public_network:
    type: string
    label: Public network name or ID
    description: Public network with floating IP addresses.
    default: external
  cidr:
    type: string
    label: Network CIDR
    description: The CIDR of the private network.
    default: '10.10.10.0/24'
  dns:
    type: comma_delimited_list
    label: DNS nameservers
    description: Comma separated list of DNS nameservers for the private network.
    default: '8.8.8.8'

resources:
  new_network:
    type: OS::Neutron::Net

  new_subnet:
    type: OS::Neutron::Subnet
    properties:
      network: { get_resource: new_network }
      cidr: { get_param: cidr }
      dns_nameservers: { get_param: dns }
  new_port:
    type: OS::Neutron::Port
    properties:
      network: { get_resource: new_network }
      fixed_ips:
        - subnet_id: { get_resource: new_subnet }
  new_router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info:
        network: { get_param: public_network }

  router_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router_id: { get_resource: new_router }
      port: {get_resource: new_port}

outputs:
  network_name: 
    description: The new network
    value: {get_attr: [new_network,name]}

```  


<a name="2.6.7"></a>  
###2.6.7.Create a instance  

```sh
heat_template_version: 2016-04-08
description: Create a instance

parameters:
  image:
    type: string
    label: Image name or ID
    description: Image to be used for compute instance
    default: cirros
  flavor:
    type: string
    label: Flavor
    description: Type of instance (flavor) to be used
    default: m1.small
  private_network:
    type: string
    label: Private network name or ID
    description: Network to attach instance to.
    default: private


resources:
  instance:
    type: OS::Nova::Server
    properties:
      flavor: { get_param: flavor }
      image: { get_param: image }
      networks: 
        - network: { get_param: private_network }

outputs:
  instance_floating_ip:
    description: Ip of instance
    value: {get_attr: [instance , first_address]}

```  

<a name="2.6.8"></a>  
###2.6.8.Create a instance with network and floating_ip  

```sh
heat_template_version: 2016-04-08

description: |
  Create a instance with network and floating_ip


parameters:
  public_network:
    type: string
    label: Public network name or ID
    description: Public network with floating IP addresses.
    default: external
  cidr:
    type: string
    label: Network CIDR
    description: The CIDR of the private network.
    default: '10.10.10.0/24'
  dns:
    type: comma_delimited_list
    label: DNS nameservers
    description: Comma separated list of DNS nameservers for the private network.
    default: '8.8.8.8'
  image:
    type: string
    label: Image name or ID
    description: Image to be used for compute instance
    default: ubuntu14.04
  flavor:
    type: string
    label: Flavor
    description: Type of instance (flavor) to be used
    default: m1.small
  key:
    type: string
    label: Key name
    description: Name of key-pair to be used for compute instance
    default: test


resources:
  new_network:
    type: OS::Neutron::Net

  new_subnet:
    type: OS::Neutron::Subnet
    properties:
      network_id: { get_resource: new_network }
      cidr: { get_param: cidr }
      dns_nameservers: { get_param: dns }
  new_port:
    type: OS::Neutron::Port
    properties:
      network: { get_resource: new_network }
      fixed_ips:
        - subnet_id: { get_resource: new_subnet }
  new_router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info:
        network: { get_param: public_network }

  router_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router: { get_resource: new_router }
      subnet: { get_resource: new_subnet }
  new_FloatingIP:
    type: OS::Nova::FloatingIP
    properties:
      pool: {get_param: public_network}

  instance:
    type: OS::Nova::Server
    properties:
      flavor: { get_param: flavor }
      image: { get_param: image }
      key_name: { get_param: key }   
      networks: 
        - port: { get_resource: new_port }

  add_floating_ip:
    type: OS::Nova::FloatingIPAssociation
    properties:
      floating_ip: {get_resource: new_FloatingIP}
      server_id: {get_resource: instance}

outputs:
  instance_name: 
    description: Name of instance
    value: { get_attr: [instance , name]}
  instance_ip_private:
    description: Ip private address of the instance
    value: { get_attr [instance , networks , {get_resource: new_network} , 0]} 
  instance_ip_floating:
    description: Floating Ip of instance
    value: { get_attr: {new_FloatingIP , ip}}

```  

