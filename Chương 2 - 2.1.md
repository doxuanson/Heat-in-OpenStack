#Chương 2 - Phần 1: HOT Template Guide 
***
#Mục lục
[2.2.HOT Template Structure](#2.1)  
[2.1.1.Giới thiệu](#2.1.1)  
[2.1.2.Template structure](#2.1.2)  


<a name="2.1"></a>
##2.1.HOT Template Structure
<a name="2.1.1" =""></a>

###2.1.1.Giới thiệu
HOT là 1 định dạng templale mới thay thế CFN - 1 đạng dạng sẵn có được hỗ trợ bởi Heat trong thời gian qua .  

<a name="2.1.2" =""></a>
###2.1.2.Template structure
Các thành phần của HOT được chia thành các section như sau:  
```sh
-  heat_template_version
#phiên bản của HOT

- description
#chú thích cho HOT ( optional )

- parameter_groups
#input param group ( optional )

- parameters
#Phần này cho phép xác định các thông số đầu vào mà phải được cung cấp khi sử dụng template ( optional )

-  resources
#định nghĩa các resource được sử dụng trong template, cần ít nhất 1 resource trong template, nó chính là sản phầm và mục đích của template.
VD: Khi viết một template để tạo ra các Router, dải mạng, VM thì lúc này cần định nghĩa resources là Router, VM, dải mạng

- outputs
#cho phép xác định các thông số đầu ra có sẵn cho người dùng một khi mẫu được khởi tạo ( optional )

- conditions
#declazation of conditions
```

####1.heat_template_version  
- Mỗi version sẽ hỗ trợ một số tính năng nhất định, truy cập <http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#heat-template-version>  
để biết các tính năng này .  
- Syntax : 
```sh
heat_template_version: 2016-04-08
```

####2.description
- Đây là section tùy chọn cho phép mô tả về template HOT .
- Syntax :
```sh
description: Simple template to deploy a single compute instance
```
- Nếu bạn muốn mô tả không phải trên 1 dòng mà là trên nhiều dòng , bạn có thể làm như sau :
```sh
description: >
  This is how you can provide a longer description
  of your template that goes over several lines.

```

####3.prameters_group
- định nghĩa dưới dạng list với mỗi group chứa một list các params</li>
- các list được sử dụng để biểu thị thứ tự mong muốn của các param</li>
- mỗi param chỉ nên nằm trong một group nhất định</li>
- Cú pháp:
```sh
parameter_groups:
- label: <human-readable label of parameter group>
  description: <description of the parameter group>
  parameters:
  - <param name>
  - <param name>
```

`label`: nhãn mà người dùng có thể đọc được liên hệ đến nhóm của các param
`description`: mô tả
`parameters`: một list của các param trong nhóm này
`param name`: tên của các param trong parameters section

####4.parameters section
- Sử dụng để định nghĩa các tham số thay đổi với mỗi khi triển khai template ví dụ như username hoặc password, image,... Mỗi param được đặt trong một khối với tên của param bắt đầu và theo sau là một loạt các thuộc tính của param .  
- Cú pháp:  
```sh
parameters:
  <param name>:
    type: <string | number | json | comma_delimited_list | boolean>
    label: <human-readable name of the parameter>
    description: <description of the parameter>
    default: <default value for parameter>
    hidden: <true | false>
    constraints:
      <parameter constraints>
    imutable: <true | false>

```  

`param name`: tên của param  
`type`: loại của param là 1 trong số string, number, comma_delimited_list, json and boolean (cần phải có)  
`labe`l: A human readable name for the parameter. This attribute is optional.  
`description`: A human readable description for the parameter. This attribute is optional.  
`default`: giá trị mặc định được sử dụng nếu người dùng không truyền giá trị vào. Đây là giá trị tùy chọn  
`hidden`: Defines whether the parameters should be hidden when a user requests information about a stack created from the template. This attribute can be used to hide passwords specified as parameters.
This attribute is optional and defaults to false.  
`constraints`: Đây là điều kiện ràng buộc của param. Khi khai báo param thì nó sẽ check xem thông số của param có trong hệ thống không. Đây là trường tùy chọn.   
`immutable`  
Defines whether the parameter is updatable. Stack update fails, if this is set to true and the parameter value is changed. This attribute is optional and defaults to false.  
- The table below describes all currently supported types with examples:
***
|Type|Description|Examples|  
|:---|:---------:|-------:|   
|string|A literal string|"String param"|  
|number|An integer or float.|“2”; “0.2”|  
|comma_delimited_list|An array of literal strings that are separated by commas. The total number of strings should be one more than the total number of commas.|[“one”, “two”]; “one, two”; Note: “one, two” returns [“one”, ” two”]|  
|json|A JSON-formatted map or list.|{“key”: “value”}|
|boolean|Boolean type value, which can be equal “t”, “true”, “on”, “y”, “yes”, or “1” for true value and “f”, “false”, “off”, “n”, “no”, or “0” for false value.|“on”; “n”|

- VD :  
```sh
parameters:
  user_name:
    type: string
    label: User Name
    description: User name to be configured for the application
  port_number:
    type: number
    label: Port Number
    description: Port number to be configured for the web server
```
```sh
The description and the label are optional, but defining these attributes is good practice to provide useful information about the role of the parameter to the user.
```

####5.resource section
- Xác định nguồn lực thực tế làm nên 1 stack được deploy từ template hay chính là sản phẩm của template đó.
- Mỗi `resource` được định nghĩa trong 1 khối riêng biệt ở trong `resource section` .
```sh
resources:
  <resource ID>:
    type: <resource type>
    properties:
      <property name>: <property value>
    metadata:
      <resource specific metadata>
    depends_on: <resource ID or list of ID>
    update_policy: <update policy>
    deletion_policy: <deletion policy>
    external_id: <external resource ID>
    condition: <condition name or expression or boolean>
```

`resource ID`: ID của Resource ( tên của Resource ) và giá trị này là duy nhất trong template  
`type:` Loại Resource, ví dụ như: OS::Nova::Server or OS::Neutron::Port. Giá trị này bắt buộc phải có trong Resource và tùy theo Resouce mà cần chỉ ra loại. VD như Resouce là VM thì cần định nghĩa loại là OS::Nova::Server  
`properties`: A list of resource-specific properties. The property value can be provided in place, or via a function (see Intrinsic functions). This section is optional.  
`metadata`: Resource-specific metadata. This section is optional.  
`depends_on`: Dependencies of the resource on one or more resources of the template. See Resource dependencies for details. This attribute is optional.  
`update_policy`: Update policy for the resource, in the form of a nested dictionary. Whether update policies are supported and what the exact semantics are depends on the type of the current resource. This attribute is optional.  
`deletion_policy`: Deletion policy for the resource. The allowed deletion policies are Delete, Retain, and Snapshot. Beginning with heat_template_version 2016-10-14, the lowercase equivalents delete, retain, and snapshot are also allowed. This attribute is optional; the default policy is to delete the physical resource when deleting a resource from the stack.  
`external_id`:  Allows for specifying the resource_id for an existing external (to the stack) resource. External resources can not depend on other resources, but we allow other resources depend on external resource. This attribute is optional. Note: when this is specified, properties will not be used for building the resource and the resource is not managed by Heat. This is not possible to update that attribute. Also resource won’t be deleted by heat when stack is deleted.  
`condition`  :Condition for the resource. Which decides whether to create the resource or not. This attribute is optional.  
**Note**: Support `condition` for resource is added in the Newton version.
All resource types that can be used in CFN templates can also be used in HOT templates, adapted to the YAML structure as outlined above.

- VD : Định nghĩa 1 compute resource đơn giản :
```sh
resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      flavor: m1.small
      image: F18-x86_64-cfntools

```

####6.Output Section
- Định nghĩa ra các params trả về cho người dùng sau khi stack được tạo ra , ví dụ như địa chỉ IP của instance, địa chỉ URL của ứng dụng web được deploy trong stack
- Each output parameter is defined as a separate block within the outputs section according to the following syntax :  
```sh
	outputs:
	  <parameter name>:
	    description: <description>
	    value: <parameter value>
	    condition: <condition name or expression or boolean>
``` 
`parameter name`: The output parameter name, which must be unique within the outputs section of a template.  
`description`: A short description of the output parameter. This attribute is optional.  
`parameter value`: The value of the output parameter. This value is usually resolved by means of a function. See Intrinsic functions for details about the functions. This attribute is required.  
`condition`    : To conditionally define an output value. None value will be shown if the condition is False. This attribute is optional.  
**Note**: Support condition for output is added in the Newton version.  
- **Example** :
```sh
outputs:
  instance_ip:
    description: IP address of the deployed compute instance
    value: { get_attr: [my_instance, first_address] }
```

####7.Conditions section



