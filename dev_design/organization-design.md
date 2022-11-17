## Organization 设计
CRD `Organization`，用于在平台唯一指代联盟的成员组织，Organization为`Namespace`级别的资源
**其中，成员组织与CA为一一对应关系。`1 Organization = 1 CA `**


### 权限设计
1. 组织内部(in namespace):
- `Admin`平台用户: 
    - 具有: `get/list/patch/update`
    - 不具有: `create/delete`
- `Client`平台用户
    -  具有: `get/list`
    -  不具有: `create/delete/update/patch`

2. 组织外部(out of namespace)
- 默认情况下为:`invisible`
- 当加入某个`Federation`后(proposal passed)，为联盟内部成员开放`viewer_cluster_role`权限(`get/list/watch`)


### CRD定义
1. `Spec`定义
```
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



2. `Status`定义
TODO...


### 控制器处理
1. `Organization`创建
- 场景1:  BAAS服务开通
**NOTE:创建/删除流程需与平台用户开通BAAS服务流程集成到一起(考虑统一用admission controller???)**
前置:
1) 平台用户开通BaaS服务，分配`user-namespace`平台用户分配`BAAS-Admin`权限
2) 平台用户完成企业认证,发起baas服务初始化
3) 基于企业认证信息，为平台用户创建了组织`CA`服务。CA服务与平台用户对接完成

`admission/organization controller`流程如下:
1) 基于`CARef`验证，CA服务是否正常
2）验证`Admin`对应的账户是否在CA中存在
3) 基于`Admin`的用户名密码`CA`发起证书签名请求，随后生成`MSP`secret
4) 基于`NumSecondsWarningPeriod` 注册证书更新任务

产物：
1) `MSP`secret，存储组织的身份证书信息


2. `Organization`删除
场景：
- 场景1： 平台用户决定关闭BaaS服务或者用户的BaaS服务过期,从而需要进行了`Oranization删除操作`

`admission/organization controller`流程如下:
1) 基于`CARef`验证，CA是否已经删除
4) 基于`NumSecondsWarningPeriod` 注册证书更新任务



3. `Organization`更新
场景：
- 场景1: 更新`Admin`用户

`admission/organization controller`流程如下:
1) 基于`CARef`验证，CA服务是否正常
2）验证`Admin`对应的账户是否在CA中存在
3) 确认新的`Admin`账户是否存在，不存在则向`CA`注册
4） 在CA中注销原有`Admin`账户
5) 基于新的`Admin`用户，更新`MSP` secret









