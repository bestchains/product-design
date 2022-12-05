## **Organization 设计**

`Organization`用于在平台唯一指代联盟的成员组织，为`Namespace`级别的资源。  
> <mark>**组织与CA为一一对应关系,即`1 Organization = 1 CA`** </mark>  

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
  // License should be accepted by the user to be able to setup console
  // +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
  License License `json:"license"`
  // DisplayName for this organization
  // +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
  DisplayName string `json:"displayName,omitempty"`

  // Admin is the platform user(CA:Admin) to represents this organization in this federation(Only one admin in each organization)
  // This account will be granted a ClusterRole(`get/list/update/patch`) to this `Federation Resource`
  // +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
  Admin string `json:"admin,omitempty"`

  AdminSecret string `json:"adminSecret,omitempty"`

  // CARef which manage credentials for this organization
  CAReference `json:"caReference,omitempty"`
}


type CAReference struct {
  Name string
  CA string
}
```

`OrganizationSpec`包含以下字段:

- `DisplayName`: 组织的展示名称，可读性强
- ` CAReference `: 组织所属的CA机构
  - `Name`: CA机构名称
  - `CA`: 具体使用哪个CA服务（ca/tlsca）
- `Admin`: 组织的管理员账户，此账户需同时为：
  - 平台用户，具有该组织`Admin`权限
  - CA用户，具有`CA.Admin`权限
- `AdminSecret`: 管理员账户对应的CA账户密码

2. `OrganizationStatus`

```go
type OrganizationStatus struct {
  // TODO: ...
  Condition metav1.Condition `json:"condition,omitempty"`  
}
```

![CRD](./images/organization-crd.png)

### **Webhook设计**

#### `Mutating Webhook` （TODO）

Default:

- DisplayName为空时，默认设置为`Organizatin Name`

#### `Validating Webhook`

1. Common Validate

- 验证`OrgName` | `DisplayName` | `Admin` | `CAReference`是否满足条件

2. Logic Validate

- `ValidateCreate`

 a. 校验参数是否服务
 c. 校验`Admin`

- `ValidateUpdate`
  a. 通用校验
- `ValidateDelete`
  a. 通用校验

### **Contoller控制器设计**

#### **Organization创建**

> **场景:  BAAS服务开通**

> ```text
> 前置:
> 1) 平台用户开通BaaS服务，分配`user-namespace`平台用户分配`BAAS-Admin`权限
> 2) 平台用户完成企业认证,发起baas服务初始化
> 3) 基于企业认证信息，为平台用户创建了组织`CA`服务。CA服务与平台用户对接完成
> 4) 平台用户对应的CA.Admin账户密码通过`k8s secret`保存
> ```

处理流程:

1. 基于`Admin`的用户名密码`CA`发起证书签名请求，随后生成两个secret:

- `organization msp secret` : 包含组织msp的证书
- `Admin crypto secret`: 包含`Admin`用户的私钥和证书

#### **Organization更新**  

场景:

- 场景1: 更新`Admin`用户

流程如下:

1. `Admin`或`CAReference.Name`更新，则触发`secret`重新生成

#### **Organization删除**

> 场景： 平台用户决定关闭BaaS服务或者用户的BaaS服务过期,从而需要进行了`Oranization删除操作`
