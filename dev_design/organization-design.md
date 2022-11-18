## **Organization 设计**
`Organization`用于在平台唯一指代联盟的成员组织，为`Namespace`级别的资源。  
>	<mark>**组织与CA为一一对应关系,即`1 Organization = 1 CA `** </mark>  
>   <mark>**组织下仅允许同时存在一个Admin,即`1 Organization = 1 Admin `** </mark>
 

### **权限设计** 
| 用户类型 | 拥有 | 拥有(条件满足)  |  不拥有  |
| ------ | ---- | ------------- |  -----  |  
| Admin(in org)  |  get/list/watch/update/patch  |  - |  create/delete |
| Client(in org) | get/list/watch | - | craete/delete/update/patch |
| Admin(out of org)  |  -  |  - |  all |
| Admin(out of org,in same federation)  |  get/list/watch  |  - |  create/delete/update/patch |
| Client(out of org) | - | - | all |



### **CRD定义**
1. `OrganizationSpec` 
```go
type OrganizationSpec struct {
	// DisplayName for this organization
	// +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
	DisplayName string `json:"displayName,omitempty"`

	// CARef which manage credentials for this organization
	CARef string `json:"ca,omitempty"`

	// Admin is the platform user(CA:Admin) to represents this organization in this federation(Only one admin in each organization)
	// This account will be granted a ClusterRole(`get/list/update/patch`) to this `Federation Resource`
	// +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
	Admin string `json:"admin,omitempty"`

    NumSecondsWarningPeriod int64 `json:"numSecondsWarningPeriod,omitempty"`

	// TODO: (Optional) customizable oranization information
}
```
`OrganizationSpec`包含以下字段：
- `DisplayName`: 组织的展示名称，可读性强
- `CARef`: 组织所属的CA机构名称
- `Admin`: 组织的管理员账户，此账户需同时为：
    - 平台用户，具有该组织`Admin`权限
    - CA用户，具有`CA.Admin`权限
- `NumSecondsWarningPeriod`: `MSP`证书过期告警周期

2. `OrganizationStatus`  
```go
type OrganizationStatus struct {
	// TODO: ...
	Condition metav1.Condition `json:"condition,omitempty"`  
}
```


![CRD](./images/organization-crd.png)



### **Webhook设计**
#### `Mutating Webhook`  
> Default 
#### `Validating Webhook`  
1. Common Validate: 
	- 验证`OrgName` | `DisplayName` | `Admin`是否符合命名规范. 

2. Logic Validate  
- `ValidateCreate`   
	a. 通用校验    
	b. 基于`CARef`验证，CA服务是否正常      
	c. 校验`Admin`账户是否存在，且是否具有`CA.Admin`权限   
- `ValidateUpdate`   
	a. 通用校验   
	b. 拒绝更新`CARef`   
	c. 如果`Admin`更新，则校验新的`Admin`账户是否存在，且是否具有`CA.Admin`权限   
- `ValidateDelete`   
	a. 通用校验   
	b. 验证`CARef`对应的CA服务是否已经删除   


### **Contoller控制器设计** 
#### **Organization创建**
> **场景:  BAAS服务开通**
> ```
> 前置:
> 1) 平台用户开通BaaS服务，分配`user-namespace`平台用户分配`BAAS-Admin`权限
> 2) 平台用户完成企业认证,发起baas服务初始化
> 3) 基于企业认证信息，为平台用户创建了组织`CA`服务。CA服务与平台用户对接完成
> ```
处理流程: 
1. 基于`Admin`的用户名密码`CA`发起证书签名请求，随后生成`MSP`secret 
2. 基于`NumSecondsWarningPeriod` 注册证书更新任务  

> 产物: `MSP`secret，存储组织的身份证书信息


#### **Organization更新**  

场景：
- 场景1: 更新`Admin`用户
流程如下:
1） 在CA中注销原有`Admin`账户
2)  基于新的`Admin`用户，更新`MSP` secret


#### **Organization删除**
> 场景： 平台用户决定关闭BaaS服务或者用户的BaaS服务过期,从而需要进行了`Oranization删除操作`

处理流程:
1) 基于`CARef`验证，CA是否已经删除  
<mark> **TODO:** 考虑跟其他CRD资源的关联，如proposal|vote|network|channel......</mark>





### 遗留问题
1. `Organization`的 `visibility`应该如何设置? 什么情况下可允许平台内其他用户检索???




