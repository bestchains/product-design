## **联盟的设计**

`Federation` 是一组平台用户(组织/机构)共同组建的区块链联盟,为<mark>**Cluster**</mark>级别的资源，联盟的内部决策基于 `proposal-vote` 机制来进行。与 Federation 相关的 proposal 类型包括:

```
- CreateFederation  
- ChangeFederationMember  
- DissolveFederation  
```

### **整体设计**

> <mark>**Federation 与 Proposal-Vote 联动可参考[`Proposal-Vote` 设计](proposal-vote-design.md)**</mark>

### **权限设计**

| 用户类型         | 拥有                  | 拥有(条件满足)                  | 不拥有              |
| ---------------- | --------------------- | ------------------------------- | ------------------- |
| Admin(发起者)    | create/get/list/watch | delete(状态为 failed/dissolved) | update/patch        |
| Admin(成员组织)) | get/list/watch        | -                               | update/patch/delete |
| Client           | get/list/watch        | -                               | update/patch/delete |

### Federation CRD 定义

1. `FederationSpec`

```go
type FederationSpec struct {
    Description string `json:"description,omitempty"`
    Policy string `json:"policy,omitemtpy"` 
    Members []string `json:"members,omitempty"`  
}

type Member struct {
	Name      string `json:"name,omitempty"`
	Namespace string `json:"namespace,omitempty"`
	Initiator bool   `json:"initiator,omitempty"`
}
 
```

解释：

- `Description`: 联盟描述信息
- `Policy`: 联盟内部决策的策略(详细可参考 proposal-vote 设计)，支持:
  - `OneVote`  一票否决。
  - `Majority` 多数人同意即同意。
  - `All` 所有人都需要同意。
- `Members`: 联盟内部成员列表,对应 `CRD Organization`
  - `Name`: 成员名称(组织名称)
  - `Namespace`:  成员所在命名空间
  - `Initiator`： 是否联盟发起人 

![Federation-Proposal-Vote](./images/federation-crd.png)

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
    CRStatus `json:",inline"`

    Networks `json:"networks"`
}

```

Federation 分为四个状态:

- **Pending**: 代表当前联盟的 `CreateFederation proposal-vote` 还在进行中,仍处在组建当中
- **Activated**: 代表当前联盟的 `CreateFederation proposal-vote` 成功，联盟正式激活
- **Failed**: 代表当前联盟的 `CreateFederation proposal-vote` 失败,联盟组建失败
- **Dissolved**: 代表当前联盟的 `DissolveFederation proposal-vote` 成功，联盟已经解散

**状态转移图如下:**
![federation-state-transition](./images/federation-state-transition.png)

### **Webhook 设计**(TODO)

1. `Mutating Webhook`:
   - 定义 `Initiator` 为当前 `Admin` 所属的组织
2. `Validating Webhook`
   - `ValidateCreate`:
     - 验证当前 `Members` 列表中的组织 `Organization` 是否都存在
   - `ValidateUpdate`: skip
     - 验证新成员组织是否存在 `Organization`
   - `ValidateDelete`:
     - 验证当前联盟状态是否为 `Failed/Dissolved`
     - 验证当前联盟是否仍有`Network`

### **Contoller 控制器设计**

#### Controller Watch

1. Watch Federation

- 监听到`Create` Event后，创建cluster role
- 监听到`Update` Event后， 不做特殊操作（Update由controller根据proposal做出）
- 监听到`Delete` Event后，不做操作

2. Watch Proposal

目的：监听`Federation`相关的Proposal状态，从而更新`Federation`的`Spec`和`Status`

- `CreateFederation`: 更新状态为`Activated`，同时更新`Organization`对应的`Status`

- `ChangeFederationMember`
  - 为新用户开通cluster role权限。或者删除cluster role权限
  - 更新`Spec.Members`

- `DissolveFederation`： 更新状态为`Dissolved`

3. Watch Network

目的: 监控`Network`，更新`Federations.Status.Networks`


#### **Federation 更新**

三种(prposal 相关)更新:

- 创建联盟: `CreateFederation`
- 更新成员:`ChangeFederationMember`
- 解散联盟: `DissolveFederation`

> `CreateFederation`的处理流程为:
>  1. controller监听`proposal`(type `CreateFederation` related)
>  2. 针对`proposal`状态:
    - 如果proposal为`succ`则**校验proposal**并更新`Fedeation`状态为`Activated`
    - 如果proposal为`fail`,**校验proposal**并更新`Federation`状态为`Failed`

> `ChangeFederationMember`的处理流程:
>  1. controller监听`proposal succ`(type `ChangeFederationMember` related)
>  2. controller校验`proposal`
>  3. 在`members`列表中，删除/新增一个成员组织

> `DissolveFederation`的处理流程为：
> 1. controller监听`proposal succ`(type `DissolveFederation` related)
> 2. 校验`proposal`
> 3. 更新`Fedeation`状态为`Dissolved`

#### **Federation 删除**

<mark> **TODO:** 考虑跟其他 CRD 资源的关联，如 proposal|vote|network|channel......</mark>
