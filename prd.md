# 产品需求文档

## 文档更新

*   2022-12-20
    
    *   更新组织管理、提议管理、联盟管理三个部分的原型设计
        
    *   补充各功能模块涉及的CRD联动
        
*   2022-12-21
    
    *   增加查询证书服务
        
    *   所有页面增加详细组件介绍
        
*   2022-12-22
    
    *    调整联盟成员页面，取消已邀请成员
        
    *   增加解散联盟弹窗
        

## 资源

*   蓝湖设计： [https://share.lanhuapp.com/#/invite?sid=lXLamcva](https://share.lanhuapp.com/#/invite?sid=lXLamcva)
    
*   iconfont素材:  需要访问权限，可  $\color{#0089FF}{@姬佩宏}$   $\color{#0089FF}{@王帅}$  帮忙添加
    
*   后端
    
    *   dev设计： [https://github.com/bestchains/product-design/tree/main/dev\_design](https://github.com/bestchains/product-design/tree/main/dev_design)
        
    *   CRD: [https://github.com/bestchains/fabric-operator/tree/main/config/crd/bases](https://github.com/bestchains/fabric-operator/tree/main/config/crd/bases)
        
    *   samples： [https://github.com/bestchains/fabric-operator/tree/main/config/samples](https://github.com/bestchains/fabric-operator/tree/main/config/samples)
        
    *   开发和需求追踪： [https://github.com/orgs/bestchains/projects/4](https://github.com/orgs/bestchains/projects/4)
        
        *   bugs
            
        *   new feature
            
*   Docke镜像仓库：
    
    *   地址： [https://hub.docker.com/](https://hub.docker.com/)
        
    *   账户:  hyperledgerk8s / Arbiter@2022
        

## 产品简介

Blockchain as a Service区块链即服务，协助降低区块链使用门槛，加速区块链场景落地

### 术语说明

|  术语  |  说明  |
| --- | --- |
|  User  |  平台用户，IAM User，开通了集群admin cluster role  |
|  Organization  |  组织，参与区块链网络，成为区块链网络成员  |
|  Federation  |  联盟，为多个组织组件的一个进行区块链行为的联盟  |
|  Network  |  网络，基于联盟创建的区块链网络，运行具体的区块链场景应用  |
|  Channel  |  通道，基于网络创建的区块链账本通道  |

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/45f2c34e-468b-498c-98c8-ac7c9279559f.png)

### 产品整体流程

1.  用户注册登录BAAS服务 
    
2.  用户创建组织 
    
    1.  用户至少拥有一个组织，才能参与区块链网络
        
    2.  配置组织CA(Todo...)
        
3.  用户创建联盟，邀请其他成员用户 
    
4.  通过Proposal-Vote完成联盟的创建
    
5.  用户创建区块链网络，配置区块链节点
    
6.  用户创建通道Channel
    

## 功能性需求

|  所属模块  |  前端页面  |  功能说明  |
| --- | --- | --- |
|  用户管理  |  用户注册登录  |  完成IAM User的创建以及平台admin-cluster-role权限的配置  |
|  组织管理  |  查看组织列表  |  查看平台现有的组织  |
|  |  新增组织  |  用户创建管理自己的组织  |
|  提议投票管理  |  查看相关的Proposal  |  列出与当前用户相关的proposal  |
|  |  处理vote  |  用户处理proposal对应的vote投票  |
|  联盟管理  |  查看联盟列表  |  查看用户具有查看权限的联盟列表  |
|  |  查看某联盟详细信息  |  查看某个联盟的详细信息，如联盟下的网络列表  |
|  |  查看某联盟成员  |  查看某联盟下的成员列表信息，并提供新增成员、剔除成员、解散联盟能力  |
|  |  新增联盟  |  创建一个新的联盟，支持配置联盟成员  |
|  网络管理  |  新建网络  |  配置网络成员、共识组件  |

#### 注册登录

##### 涉及CRDs

*   IAM User
    

##### 需求描述

用户使用平台服务前，必须先注册成为平台用户

##### 原型设计

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/16bc9660-b7ea-49da-8c55-6c14a791414d.png)

##### 流程说明

1.  注册开通平台用户
    
2.  用户实名认证(必填)
    
3.  用户企业认证(可选)
    
4.  User授予admin-cluster-role
    

#### 组织管理

##### 涉及CRDs

*   IAM User
    
*   Organization
    
*   IBPCA
    

##### 需求说明

用户管理其所属的组织，包含：

*   查看组织列表
    
*   创建新的组织
    
*   查看证书服务
    

##### 原型设计

1.  **组织列表**
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/bcf211db-77e3-48e5-a5d0-342b7dab8ba9.png)

*   按钮新建组织 ： 点击后弹出窗口 创建组织
    
*   按钮 全部组织： 点击后取消所有过滤
    
*   按钮 我创建的： 基于labels.organization.Admin过滤
    
*   输入框过滤： 基于organization name和\`labels.organization.admin\`过滤
    
*   Table内容
    
    *   组织名称： organization name
        
    *   管理员Admin: organization.spec.admin
        
    *   创建时间： organization.metadata.creationTimestamp
        
    *   状态：organization.status.type
        
    *   操作
        
        *   查看证书服务: 点击后跳转到 证书服务
            
        *   更多 -  暂无
            

2.  **创建组织**
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/138a1124-31cd-40c3-9e1a-73b8e4e82660.png)

*   组织名称： 
    
    *   输入框，必选项
        
    *   内容限制:大小写英文字母、数字、下划线
        
    *   长度限制：3-10
        
*   展示名：
    
    *   输入框，可选项
        
    *   内容限制: 中英文、数字
        
    *   长度限制：0-20
        
*   组织描述
    
    *   输入框，可选项
        
    *   内容不限制
        
    *   长度限制: 0-200
        
*   按钮确定
    
    *   点击后创建组织 ​
        

1.  **查看证书服务(待调整)**
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/e579195a-52d1-4680-9c08-dc118eba35e4.png)

*   详情卡
    
    *   CA:  ca.name
        
    *   所属组织:  对应的组织名
        
    *   创建时间: ca.metadata.creationTimestamp
        
    *   加密方式：
        
        *   仅支持 非国密ECDSA
            
        *   版本: ca.spec.version
            
        *   状态: ca.status.type
            
*   过滤项
    
    *   ID类型
        
        *   下拉框，单选/多选
            
        *   选项：
            
            *   admin
                
            *   peer
                
            *   orderer
                
            *   client
                
*   Table
    
    *   ID ： 名称
        
    *   类型：  ID的类型
        
    *   创建时间:
        
    *   属性
        
    *   操作
        
        *   查看证书： 点击后弹窗页面展示(text)
            

##### 功能说明

1.  查看组织列表
    
    1.  查询展示平台现有的所有组织
        

    命令：
    	 kubectl get orgs

2.  支持按照用户过滤创建的组织
    

    命令：
       1. kubectl get orgs 
       2. 基于 labels.organization.admin 字段过滤

3.  组织状态，展示分为：
    
    1.  正常:   Status.Type为 Deployed
        
    2.  异常:  Status.Type为其他内容，鼠标点击到异常图标后续能提示异常原因，Status.reason
        
4.  创建新的组织
    
    1.  填写组织名称(唯一性，不允许和其他组织冲突)
        
    2.  配置Admin为当前User
        
    3.  填写描述信息
        
    4.  完成**Organization** CR的创建和状态监控
        

    命令：
        1. kubectl apply -f ibp.com_v1beta1_organization.yaml

:::
NOTE: 必须填充caSpec，参考 fabric-operator/config/samples/orgs/org1.yaml
:::

3.  查看组织的CA
    

    命令：
    	1. kubectl get ibpcas -n{org_name} {org_name}  // 与组织名相同
      2. kubectl get cm -n{org_name} {org_name}-connection-profile

4.  查看CA的ID列表
    

    命令: 
      1. 访问IAM API，查询当前用户信息
      2. 解析`user.annotations.bestchains.list` 
      3. 查看当前CA对应的组织
      4. 读取 `ids`获取列表
     

#### 提议管理 

##### 涉及CRDs

*   Propsoal
    
*   Vote
    

##### 需求说明

*    查看与用户相关的提议列表
    
*   查看提议详情以及投票状态
    
*   更新用户自己的投票
    

##### 原型设计

1.  提议列表
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/e0f5de6b-1ed4-4115-9769-65d35e0a2b76.png)

*   按钮全部提议: 取消所有过滤
    
*   按钮我创建的:  查看用户创建的提议
    
*   按钮 待处理的: 查看用户还未完成投票的用户
    
*   下拉框，单选 类型过滤：
    
    *   列表
        
        *   CreateFederation
            
        *   AddMember
            
        *   DeleteMember
            
        *   DissolveFederation
            
    *   选择后，基于labels.proposal.type过滤
        
*   Table内容
    
    *   提议名称 : proposal name
        
    *   提议类型:  proposal.labels.proposal.type
        
    *   提议策略: proposal.spec.policy
        
    *   创建时间: prposal.metadata.creationTimestamp
        
    *   截止时间: proposal.spec.endAt
        
    *   当前状态： proposal.status.phase
        
    *   已投票
        
        *   基于用户在该proposal投票的状态展示 \`vote.status.phase\`
            
    *   操作
        
        *   查看详情： 点击后进入页面 提议详情
            

1.  提议详情
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/1b3e72fb-48b2-4a9a-8f4b-46ccde3c461f.png)

*   详情卡
    
    *   Title:  proposal.name
        
    *   发起人:  组织名称(用户名) 
        
        *   组织名称: proposal.spec.initiator.name
            
        *   用户名:  组织名对应的Admin organization.spec.admin
            
        *   创建时间: proposal.metadata.creationTimestamp
            
        *   截止时间: proposal.spec.endAt
            
        *   状态: proposal.status.phase
            
*   过滤项：
    
    *   全部: 不过滤，proposal下所有的投票
        
    *   已投票: 过滤当前用户所在组织的投票状态为 Voted
        
    *   未投票： 过滤当前用户所在组织的投票状态为 Created
        
*   Table (Votes可直接从proposal.status.votes获取) 
    
    *   投票人:  organization.name对应组织的Admin organization.spec.admin
        
    *   所属组织:   organization.name
        
    *   投票时间: startTime
        
    *   决定: decision
        
    *   原因： description
        
    *   阶段: phase
        
    *   操作
        
        *   按钮 更新:    点击后弹窗 更新投票
            
            *   需判断当前用户是否为投票人 （非投票人不允许更新）
                
            *   需判断当前proposal是否已经截止 (proposal结束后，不允许更新)
                
            *   需判断当前Vote阶段是否为 Created (vote阶段为 Voted或者Finished，不允许更新)
                

3.  更新投票
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/7813ffd8-d320-49ac-ad35-3c29ad91eba8.png)

*   单选框 决定：
    
    *   同意，即true
        
    *   不同意，即 false
        
*   原因: 对应决定的描述
    
*   按钮 确定： 点击后更新对应的Vote CR
    

##### 功能说明

1.  查看与用户相关的proposal列表
    

    命令：
      1. 获取用户作为admin的组织列表，如{org1,org2}
      2. 获取与用户所属组织相关的Votes
      	- kubectl get votes -n org1 
        - kubectl get votes -n org2
      3. kubectl get proposal {vote.spec.proposalName}
      
     Note: 用户拥有该proposal的vote，即代表proposal与用户相关

2.  过滤用户创建的proposal列表
    

    命令：
    	1. 完成proposal列表查询
      2. 查询用户当前作为Admin的组织列表
      2. 基于 proposal.spec.initiator.name 过滤不属于用户组织的proposal

3.  待处理的proposal列表
    

    命令：
       1. 获取用户作为admin的组织列表，如{org1,org2}
       2. 获取与用户所属组织相关的Votes
      	- kubectl get votes -n org1 
        - kubectl get votes -n org2
       3. 保留 votes.status.phase == Created 的vote
       4. kubectl get proposal {vote.spec.proposalName}

4.  基于类型过滤proposal
    

NOTE: proposal类型目前包括：

    - createFederation
    - addMember
    - deleteMember
    - dissolveFederation

     命令：
     	1. 完成proposal列表查询
      2. 基于 labels.proposal.type 过滤

5.  查看proposal 对应的vote列表
    

    命令：
      1. kubect get proposal {proposal_name}
      2. 查看 `proposal.status.votes`获取当前的投票列表
    

6.  更新vote
    

    命令(参考)：
     1. kubectl patch vote -n org2 vote-org2-create-federation-sample --type='json' -p='[{"op": "replace", "path": "/spec/decision", "value": true}]'

7.  用户发起对proposal的投票(更新对应的Vote)
    

    命令：
      1. kubectl patch vote -n org2 vote-org2-create-federation-sample --type='json' -p='[{"op": "replace", "path": "/spec/decision", "value": true}]'

#### 联盟管理

##### 涉及CRDs

*   IAM User
    
*   Organization
    
*   Federation
    
*   Proposal
    

##### 需求说明

用户进行联盟的管理，包括：

*   查看所在联盟列表
    
*   查看联盟详细信息
    
*   新建联盟，设置成员列表
    
*   添加新的成员
    
*   剔除成员
    
*   解散联盟
    
*   删除联盟
    

##### 原型设计

1.  联盟
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/e4fb893e-dd4b-4ecd-9f9e-cb4a62c3e779.png)

*   按钮新建联盟： 点击后弹出新建联盟窗口
    
*   过滤栏
    
    *   按钮 全部联盟: 不加过滤
        
    *   按钮我创建的: 查看以当前用户为Admin的组织创建的联盟
        
    *   输入框 联盟名称或创建人
        
*   Table
    
    *   联盟名称
        
    *   发起者: 为组合项需展示组织名(用户名)
        
        *   组织名： labels.federation.initiator
            
        *   用户名: 上述组织对应的 \`organization.spec.admin\`
            
    *   成员个数： federation.spec.members数组长度
        
    *   网络个数: federation.status.networks长度
        
    *   创建时间: federation.metadata.creationTimestamp
        
    *   加入时间: 当前用户所属组织加入此联盟的时间(参考功能说明部分的CRD联动命令) 
        
        *   为联盟创建相关的proposal中此组织投票的时间
            
    *   提议策略； federation.spec.policy
        
    *   操作：
        
        *   按钮管理联盟: 点击后跳转到页面联盟详情-联盟信息
            
        *   更多： 包含以下
            
            *   解散联盟
                
                *   点击后，弹窗 解散联盟
                    
                *   点击按钮确认后：发起新的proposal，类型为 dissolveFederation
                    

1.  解散联盟
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/cb2f0ecb-693c-440f-be08-df9672eb6f9c.png)

*   按钮确认
    
    *   点击后，创建一个新的proposal,类型为dissolveFederation
        

3.  新建联盟 
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/f71494fb-1a76-4828-972b-38cfce8abffb.png)

*   联盟名称：
    
    *   限制大小写英文字母、数字、下划线
        
    *   限制长度： 3-20
        
*   成员限制: 固定为实名认证
    
*   选择成员: 多选框
    
*   提议投票策略：
    
    *   ALL
        
    *   Majority
        
*   联盟描述：
    
    *   限制长度： 0-200
        
*   按钮确认： 点击确认后，创建\`Federation CR\`
    

4.  查看联盟信息
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/02fd43da-5141-4610-bafc-a7d14f3c6228.png)

*   联盟信息卡
    
    *   Title: 联盟名称
        
    *   联盟描述: federation.spec.description
        
    *   创建人: 参考 联盟列表Table中的发起人
        
    *   成员限制: 固定为实名认证
        
    *    创建时间: federation.metadata.creationTimestamp
        
    *   加入时间:  参考联盟列表
        
    *   提议策略: federation.spec.policy
        
*   按钮新建网络: 点击后，跳转到新建网络页面
    
*   Table(TODO...) 
    

5.  查看联盟成员
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/1b9d081b-1d5f-4d47-b496-b68692636abb.png)

*   按钮 邀请成员，点击弹窗邀请成员
    
*   成员Table  federation.spec.members
    
    *   成员组织: member.name
        
    *   组织Admin: organization.spec.admin
        
    *   认证信息: 读取组织对应的AdminUser的认证状态
        
    *   加入时间： 参考联盟列表
        
    *   操作：
        
        *   按钮删除
            
            *    点击即发起提议，提议类型为\`deleteMember\`
                

6.  邀请成员
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/8c318a39-88fd-4f8a-96a0-30b0ca9cc705.png)

*   单选框 - 成员组织
    
    *   列表内容需过滤掉已经在联盟中的成员
        
*   按钮 确定
    
    *   点击后，创建一个新的提议，类型为 addMember
        

##### 功能说明

1.  查看联盟列表
    

    命令：
    1. 查询用户所属组织，参考组织管理
    2. 通过organization.status.federations 获取组织所在联盟列表
    3. kubectl get federation {federation_name} // 通过上述获得的联盟名称

2.  过滤用户所属组织发起的联盟
    

    命令：
    1. 通过上述方法获得用户组织所在的联盟
    2. 通过 labels.federation.initiator 判断是否为当前用户所属组织

3.  新建联盟
    
    1.  成员列表为下拉多选框，通过kubectl get orgs获取所有组织列表
        
    2.  填充后通过下述命令完成federation创建
        

    命令：
    1. kubectl apply -f ibp.com_v1beta1_federation.yaml

4.  查看联盟信息
    

    命令：
     1. kubectl get federation {fed_name}

1.  联盟基本信息, federation.spec
    
2.  联盟网络列表， fefederation.status.networks。通过下述接口获取详细网络信息
    

        kubectl get network {network_name}

5.  查看联盟成员列表
    
    1.  通过federation.spec.members获取当前的成员列表 
        
6.  邀请成员
    
    1.  选择成员组织 （组织列表参考组织管理）
        
    2.  填写描述信息
        
    3.  点击确定后，创建proposal
        

    kubectl apply -f ibp.com_v1beta1_proposal_add_member.yaml

7.  剔除成员
    

1.  选择要剔除的成员组织
    
2.  填写描述信息
    
3.  点击确定后，创建proposal
    

    kubectl apply -f ibp.com_v1beta1_proposal_delete_member.yaml

8. 解散联盟

1.  选择要解散的联盟
    
2.  填写描述信息
    
3.  点击确定后，创建proposal
    

    kubectl apply -f ibp.com_v1beta1_proposal_dissolve_federation.yaml

#### 网络管理

##### 需求说明

用户管理区块链网络，包括:

*   查看用户所有参与的区块链网络
    
*   创建新的区块链网络
    

##### 原型设计

1.  网络列表
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/27c4a10a-dfe2-4508-86c2-7dfb5bf20859.png)

2.  新建网络
    

##### 功能说明

1.  查看当前联盟下的区块链网络列表
    

    命令：
      1. 查看用户所有参与的联盟 （参考联盟查询)
      2. 通过联盟获取所有的网络列表 `federation.status.networks`
      3. 查询网络的详细信息  `kubectl get network {network_name}`

2.  创建新的区块链网络
    
    1.  配置联盟
        
    2.  配置共识组件
        
    3.  配置邀请网络成员
        

    命令：
     1. kubectl apply -f ibp.com_v1beta1_network.yaml

## 后端接口(CRDs)

CRD定义：

[https://github.com/bestchains/fabric-operator/tree/main/config/crd/bases](https://github.com/bestchains/fabric-operator/tree/main/config/crd/bases)

CRD使用示例：

[https://github.com/bestchains/fabric-operator/tree/main/config/samples](https://github.com/bestchains/fabric-operator/tree/main/config/samples)

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e1X3lE5M6kWAlJbv/img/2d981098-030a-4134-9056-5fc972c7019d.png)

|  CRD  |  Scope  |  定义  |  说明  |
| --- | --- | --- | --- |
|  Organization  |  Cluster  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_organizations.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_organizations.yaml)  |   |
|  Federation  |  Cluster  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_federations.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_federations.yaml)  |   |
|  Proposal  |  Cluster  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_proposals.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_proposals.yaml)  |   |
|  Vote  |  Namespaced  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_votes.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_votes.yaml)  |   |
|  Network  |  Cluster  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_networks.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_networks.yaml)  |   |
|  IBPCA  |  Namespaced  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_ibpcas.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_ibpcas.yaml)  |   |
|  IBPOrderer  |  Namespaced  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_ibporderers.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_ibporderers.yaml)  |   |
|  IBPPeer  |  Namespaced  |  [https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com\_ibppeers.yaml](https://github.com/bestchains/fabric-operator/blob/main/config/crd/bases/ibp.com_ibppeers.yaml)  |   |