# Extend IAM with Fabric

扩展IAM `User CRD`，用于支持区块链服务需求

## CRD UserSpec

```go
type UserSpec struct {
  ... // iam 用户的原来spec字段内容

  // 为 IAM 用户存储的用户扩展信息
  ExternalConf ExternalConf `json:"externalConf,omitempty"`
}

type ExternalConf struct {
  // 用来区分自己的服务，解析下面的配置内容
  // 咱们的可以写BlockChain
  Type string `json:"type"`

  // 存储配置信息，下面SID，Secret等内容都存放到这里
  Conf map[string]interface{} `json:"conf,omitempty"`

  // SID string `json:"secret,omitempty"`
  // Secret string `json:"secret,omitemtpy"`
  // OU string `json:"ou,omitemtpy"`
  // CAReference `json:"caReference,omitempty"`
}

type CAReference struct {
  Name string
  CA string
}
```

`IAM BlockChain` 扩展内容包含以下字段:

- `SID`:区块链服务ID
- `Secret`: CA认证密码。为空时将随机生成,
- `OU`: 对应的CA用户类型，`Admin\Client`
- `Status`: 当前用户状态Deactivated
- `Attibutes`: (Optional) 额外的用户属性
- ` CAReference `: 组织所属的CA机构
  - `Name`: CA机构名称
  - `CA`: 具体使用哪个CA服务，默认为ca（ca/tlsca）


## 需求

**BlockChain 的 ExternalConf 以下统称为 BlockchainExtend**

### IAM用户开通区块链服务

1. 更新填充`BlockchainExtend`,包括:

- secret: 设置为`CA`启动时配置的Admin用户密码
- OU: 设置为`admin`
- CAReference: 配置CA

2. controller监测到 `BlockchainExtend` 的更新处理逻辑：

- 检测CA是否存在
  - 不存在,则记录错误信息
  - 存在，则`enroll`获得用户证书，生成`k8s secret {user}-crypto-secret`

### IAM Admin用户停止该区块链服务

1. 更新 `BlockchainExtend` 的状态为Deactivated
2. controller检测到后：

- 清理`BlockchainExtend`


### IAM用户被加入到由其他人开通的区块链服务

1. 更新填充`BlockchainExtend`,包括:

- secret: 随机生成secret
- OU: 设置为`client`
- CAReference: 设置为该区块链服务对应的CA

2. controller监测到更新`BlockchainExtendSpec`处理逻辑

- 校验CA是否存在
  - 不存在则记录错误信息
- 向CA注册`Client`用户
- `enroll`获得用户证书，生成`k8s secret {user}-crypto-secret`

### IAM Client用户被踢出该区块链服务

1. 更新`BlockchainExtend`对应的状态为`Deactivated`

2. controller监测到`Deactivated`状态后:

- 校验CA是否存在
  - 不存在则直接结束
  - 若存在，则向CA发起revoke用户操作
- 清理`BlockchainExtend`

### controller 改动存在的问题
目前用户做了扩展后，具体的操作逻辑是否也要写到 ima controller中？
如果写到iam的controller，那么估计后续每次新对接，都需要增加一份逻辑。

