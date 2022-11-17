## Proposal-Vote机制设计

### 整体设计

### 相关CRDs设计
#### Proposal CRD
##### 权限设计
1. `Admin`用户: xxx
2. `Client`用户: xxx
##### 定义
**NOTE:CRD Spec详细定义**
```
type ProposalSpec struct {
    xxx
}
```



##### Webhook(Optional)

##### 控制器
**NOTE:核心行为控制**
细分到不同的类型处理: 
- `CreateFederation`
- `ChangeMember`
- `DissolveFederation`


1. proposal创建

2. proposal更新


#### Vote CRD
##### 权限设计
1. `Admin`用户: xxx
2. `Client`用户: xxx
##### 定义
**NOTE:CRD Spec详细定义**

##### Webhook(Optional)

##### 控制器
**NOTE:核心行为控制**