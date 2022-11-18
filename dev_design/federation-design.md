## **联盟的设计**
`Federation`是一组平台用户(组织/机构)共同组建的区块链联盟,为<mark>**Cluster**</mark>级别的资源，联盟的内部决策基于`proposal-vote`机制来进行。与Federation相关的proposal类型包括:  

    - CreateFederation  
    - ChangeFederationMember  
    - DissolveFederation  


### **整体设计**
> <mark>**Federation与Proposal-Vote联动可参考`Proposal-Vote`设计**</mark>


### **权限设计**
| 用户类型 | 拥有 | 拥有(条件满足)  |  不拥有  |
| ------ | ---- | ------------- |  -----  |  
| Admin(发起者)  |  create/get/list/watch  |  delete(状态为failed/dissolved) |  update/patch |
| Admin(成员组织)) | get/list/watch  |  -  |  update/patch/delete |
| Client | get/list/watch | - | update/patch/delete |


### Federation CRD定义
1. `FederationSpec`
```go
type FederationSpec struct {
    Initiator string `json:"initiator,omitempty"`
    Description string `json:"description,omitempty"`
    Policy string `json:"policy,omitemtpy"` 
    Members []string `json:"members,omitempty"`  
}
```
解释：
- `Initiator`: 发起者组织名称，对应`CRD Organization`
- `Description`: 联盟描述信息
- `Policy`: 联盟内部决策的策略(详细可参考proposal-vote设计)，支持:
    - `OneVote`
    - `Majority`
-  `Members`: 联盟内部成员列表,对应`CRD Organization`

![Federation-Proposal-Vote](./images/federation-proposal-vote-page.png)

2. `FedeartionStatus` 
```go
type ConditionType string
const (
    Pending ConditionType = "Pending"
    Activated ConditionType = "Activated"
    Failed ConditionType = "Failed"
    Dissolved ConditionType = "Dissolved"
)
type ConditionStatus string
const (
    ConditionTrue ConditionStatus = "True"
    ConditionFalse ConditionStatus = "False"
    ConditionUnknown ConditionStatus = "Unknown"
)

type FedeartionStatus stuct {
    Type ConditionType  `json:"type,omitempty"`
    Status ConditionStatus `json:"status,omitempty"`

    Reason string  `json:"reason,omitemtpy"`
    Message string `json:"message,omitempty"`

    ObservedGeneration int64 `json:"observedGeneration,omitemtpy"`
    LastTransitionTime Time `json:"lastTransitionTime,omitempty"`
}

```


Federation分为四个状态:
- **Pending**: 代表当前联盟的`CreateFederation proposal-vote`还在进行中,仍处在组建当中
- **Activated**: 代表当前联盟的`CreateFederation proposal-vote`成功，联盟正式激活
- **Failed**: 代表当前联盟的`CreateFederation proposal-vote`失败,联盟组建失败
- **Dissolved**: 代表当前联盟的`DissolveFederation proposal-vote`成功，联盟已经解散


**状态转移图如下:**
![federation-state-transition](./images/federation-state-transition.png)






### **Webhook设计**
1. `Mutating Webhook`: 
    - 定义`Initiator`为当前`Admin`所属的组织
2. `Validating Webhook`
    - `ValidateCreate`: 
        - 验证当前`Members`列表中的组织`Organization`是否都存在
    - `ValidateUpdate`: skip
        - 验证新成员组织是否存在`Organization`
    - `ValidateDelete`:
        - 验证当前联盟状态是否为`Failed/Dissolved`

### **Contoller控制器设计**
#### **Federation创建**   
TODO: 无需额外操作

#### **Federation更新**    
三种(prposal相关)更新:   
    -  创建联盟: `CreateFederation`  
    -  更新成员:`ChangeFederationMember`     
    -  解散联盟: `DissolveFederation`    


> `CreateFederation`的处理流程为:
>  1. controller监听`proposal`(type `CreateFederation` related)
>  2. 针对`proposal`状态:
    - 如果proposal为`succ`,则更新`Fedeation`状态为`Activated`
    - 如果proposal为`fail`,则更新`Federation`状态为`Failed`



> `ChangeFederationMember`的处理流程:
>  1. controller监听`proposal succ`(type `ChangeFederationMember` related)
>  2. 在`members`列表中，删除/新增一个成员组织


> `DissolveFederation`的处理流程为：
> 1. controller监听`proposal succ`(type `DissolveFederation` related)
> 2. 更新`Fedeation`状态为`Dissolved`


#### **Federation删除**
<mark> **TODO:** 考虑跟其他CRD资源的关联，如proposal|vote|network|channel......</mark>