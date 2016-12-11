#Chương 2 : HOT Template Guide 
***
#Mục lục  
[2.3.OpenStack Resource Types](#2.3)  
[2.4.Hướng dẫn tạo Networks , Instances](#2.4)  
- [2.4.1.Manage networks](#2.4.1)  
- [2.4.2.Manage instance](#2.4.2)  
 
[2.5.Sử dụng section parameters and section output in Stack](#2.5)  
- [2.5.1.A most basic template](#2.5.1)  
- [2.5.2.Template input parameters](#2.5.2)  
- [2.5.3.Providing template outputs](#2.5.3)  


***  

<a name="2.3"></a>  
##2.3.OpenStack Resource Types  
- Tham khảo :  
<http://docs.openstack.org/developer/heat/template_guide/openstack.html>  

<a name="2.4"></a>  
##2.4.Hướng dẫn tạo Networks , Instances  
<a name="2.4.1"></a>  
###2.4.1.Manage networks  
####1.Create a network and a subnet  
>Note : The Networking service (neutron) must be enabled on your OpenStack deployment to create and manage networks and subnets. Networks and subnets cannot be created if your deployment uses legacy networking (nova-network).  

- Use the `OS::Neutron::Net` resource to create a network, and the `OS::Neutron::Subnet` resource to provide a subnet for this network:  
```sh
resources:
  new_net:
    type: OS::Neutron::Net

  new_subnet:
    type: OS::Neutron::Subnet
    properties:
      network_id: { get_resource: new_net }
      cidr: "10.8.1.0/24"
      dns_nameservers: [ "8.8.8.8", "8.8.4.4" ]
      ip_version: 4
```  

####2.Create and manage a router  
- Use the `OS::Neutron::Router` resource to create a router. You can define its gateway with the `external_gateway_info` property:  
```sh
resources:
  router1:
    type: OS::Neutron::Router
    properties:
      external_gateway_info: { network: public }
```  
- You can connect subnets to routers with the `OS::Neutron::RouterInterface` resource:  
```sh
resources:
  subnet1_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router_id: { get_resource: router1 }
      subnet: private-subnet
```  

####3.Complete network example  
- The following example creates a network stack:  
  - A network and an associated subnet.  
  - A router with an external gateway.  
  - An interface to the new subnet for the new router.  
- In this example, the public network is an existing shared network: 

```sh
resources:
  internal_net:
    type: OS::Neutron::Net

  internal_subnet:
    type: OS::Neutron::Subnet
    properties:
      network_id: { get_resource: internal_net }
      cidr: "10.8.1.0/24"
      dns_nameservers: [ "8.8.8.8", "8.8.4.4" ]
      ip_version: 4

  internal_router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info: { network: public }

  internal_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router_id: { get_resource: internal_router }
      subnet: { get_resource: internal_subnet }
```  

<a name="2.4.2"></a>
###2.4.2.Manage instance  
####1.Create an instance  
- Use the `OS::Nova::Server` resource to create a Compute instance. The flavor property is the only mandatory one, but you need to define a boot source using one of the `image` or `block_device_mapping` properties.  
- You also need to define the `networks` property to indicate to which networks your instance must connect if multiple networks are available in your tenant.  
- The following example creates a simple instance, booted from an image, and connecting to the `private` network:  

```sh
resources:
  instance:
    type: OS::Nova::Server
    properties:
      flavor: m1.small
      image: ubuntu-trusty-x86_64
      networks:
        - network: private
```  

####2.Connect an instance to a network  
- Use the networks property of an `OS::Nova::Server` resource to define which networks an instance should connect to. Define each network as a YAML map, containing one of the following keys:  
  - `port`  
The ID of an existing Networking port. You usually create this port in the same template using an OS::Neutron::Port resource. You will be able to associate a floating IP to this port, and the port to your Compute instance.  
  - `network`
The name or ID of an existing network. You don’t need to create an OS::Neutron::Port resource if you use this property. But you will not be able to use neutron floating IP association for this instance because there will be no specified port for server.
- The following example demonstrates the use of the `port` and `network`properties:  

```sh 
resources:
  instance_port:
    type: OS::Neutron::Port
    properties:
      network: private
      fixed_ips:
        - subnet_id: "private-subnet"

  instance1:
    type: OS::Nova::Server
    properties:
      flavor: m1.small
      image: ubuntu-trusty-x86_64
      networks:
        - port: { get_resource: instance_port }

  instance2:
    type: OS::Nova::Server
    properties:
      flavor: m1.small
      image: ubuntu-trusty-x86_64
      networks:
        - network: private
```  

####3.Create and associate a floating IP to an instance  
You can use two sets of resources to create and associate floating IPs to instances.  
**OS::Nova resources**  
***  
Use the OS::Nova::FloatingIP resource to create a floating IP, and the OS::Nova::FloatingIPAssociation resource to associate the floating IP to an instance.  
The following example creates an instance and a floating IP, and associate the floating IP to the instance:  
```sh
resources:
  floating_ip:
    type: OS::Nova::FloatingIP
    properties:
      pool: public

  inst1:
    type: OS::Nova::Server
    properties:
      flavor: m1.small
      image: ubuntu-trusty-x86_64
      networks:
        - network: private

  association:
    type: OS::Nova::FloatingIPAssociation
    properties:
      floating_ip: { get_resource: floating_ip }
      server_id: { get_resource: inst1 }
```  

**OS::Neutron resources**  

>Note : The Networking service (neutron) must be enabled on your OpenStack deployment to use these resources.  

Use the OS::Neutron::FloatingIP resource to create a floating IP, and the OS::Neutron::FloatingIPAssociation resource to associate the floating IP to a port:  
```sh
parameters:
  net:
    description: name of network used to launch instance.
    type: string
    default: private

resources:
  inst1:
    type: OS::Nova::Server
    properties:
      flavor: m1.small
      image: ubuntu-trusty-x86_64
      networks:
        - network: {get_param: net}

  floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: public

  association:
    type: OS::Neutron::FloatingIPAssociation
    properties:
      floatingip_id: { get_resource: floating_ip }
      port_id: {get_attr: [inst1, addresses, {get_param: net}, 0, port]}
```  
You can also create an OS::Neutron::Port and associate that with the server and the floating IP. However the approach mentioned above will work better with stack updates.  
```sh
resources:
  instance_port:
    type: OS::Neutron::Port
    properties:
      network: private
      fixed_ips:
        - subnet_id: "private-subnet"

  floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: public

  association:
    type: OS::Neutron::FloatingIPAssociation
    properties:
      floatingip_id: { get_resource: floating_ip }
      port_id: { get_resource: instance_port }
```  

<a name="2.5"></a>  
##2.5.Sử dụng section parameters and section output in Stack  
<a name="2.5.1."></a>
###2.5.1.A most basic template  
The most basic template you can think of may contain only a single resource definition using only predefined properties (along with the mandatory Heat template version tag). For example, the template below could be used to simply deploy a single compute instance.  
```sh
heat_template_version: 2015-04-30

description: Simple template to deploy a single compute instance

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      key_name: my_key
      image: F18-x86_64-cfntools
      flavor: m1.small
```  

Each HOT template has to include the `heat_template_version` key with a valid version of HOT, e.g. 2015-10-15 (see Heat template version for a list of all versions). While the description is optional, it is good practice to include some useful text that describes what users can do with the template. In case you want to provide a longer description that does not fit on a single line, you can provide multi-line text in YAML, for example:  
```sh
description: >
  This is how you can provide a longer description
  of your template that goes over several lines.
```  

The `resources` section is required and must contain at least one resource definition. In the example above, a compute instance is defined with fixed values for the ‘key_name’, ‘image’ and ‘flavor’ parameters.  
Note that all those elements, i.e. a key-pair with the given name, the image and the flavor have to exist in the OpenStack environment where the template is used. Typically a template is made more easily reusable, though, by defining a set of `input parameters` instead of hard-coding such values.  

<a name="2.5.2"></a>  
###2.5.2.Template input parameters  
Input parameters defined in the parameters section of a HOT template allow users to customize a template during deployment. For example, this allows for providing custom key-pair names or image IDs to be used for a deployment. From a template author’s perspective, this helps to make a template more easily reusable by avoiding hardcoded assSumptions.  
Sticking to the example used above, it makes sense to allow users to provide their custom key-pairs, provide their own image, and to select a flavor for the compute instance. This can be achieved by extending the initial template as follows:  
```sh
heat_template_version: 2015-04-30

description: Simple template to deploy a single compute instance

parameters:
  key_name:
    type: string
    label: Key Name
    description: Name of key-pair to be used for compute instance
  image_id:
    type: string
    label: Image ID
    description: Image to be used for compute instance
  instance_type:
    type: string
    label: Instance Type
    description: Type of instance (flavor) to be used

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      key_name: { get_param: key_name }
      image: { get_param: image_id }
      flavor: { get_param: instance_type }
```  
In the example above, three input parameters have been defined that have to be provided by the user upon deployment. The fixed values for the respective resource properties have been replaced by references to the corresponding input parameters by means of the get_param function.  
You can also define default values for input parameters which will be used in case the user does not provide the respective parameter during deployment. For example, the following definition for the instance_type parameter would select the ‘m1.small’ flavor unless specified otherwise by the user.  
```sh
parameters:
  instance_type:
    type: string
    label: Instance Type
    description: Type of instance (flavor) to be used
    default: m1.small
```  
Another option that can be specified for a parameter is to hide its value when users request information about a stack deployed from a template. This is achieved by the `hidden` attribute and useful, for example when requesting passwords as user input:  
```sh
parameters:
  database_password:
    type: string
    label: Database Password
    description: Password to be used for database
    hidden: true
```  

<a name="2.5.3"></a>  
###2.5.3.Providing template outputs  
In addition to template customization through input parameters, you will typically want to provide outputs to users, which can be done in the outputs section of a template (see also Outputs section). For example, the IP address by which the instance defined in the example above can be accessed should be provided to users. Otherwise, users would have to look it up themselves. The definition for providing the IP address of the compute instance as an output is shown in the following snippet:  
```sh
outputs:
  instance_ip:
    description: The IP address of the deployed instance
    value: { get_attr: [my_instance, first_address] }
```  

Output values are typically resolved using intrinsic function such as the get_attr function in the example above (see also Intrinsic functions).  



















