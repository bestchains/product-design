# Extend IAM with Fabric

扩展IAM `User CRD`，用于支持区块链服务需求

## CRD UserSpec

```go
type UserSpec struct {
  IAMUserSpec `json:"iamUserSpec,omitemtpy"`

  // Extended part 
  Blockchain  BlockchainExtendSpec `json:"blockchain,omitempty"`
}

- `IAMUserSpec`为原IAM用户内容
- `Blockchain` 为区块链用户的扩展内容

其中，`BlockchainExtendSpec`详细说明:

type BlockchainExtendSpec struct {
  SID string `json:"secret,omitempty"`
  Secret string `json:"secret,omitemtpy"`
  OU string `json:"ou,omitemtpy"`
  CAReference `json:"caReference,omitempty"`
}

type CAReference struct {
  Name string
  CA string
}
```

`BlockchainExtendSpec`包含以下字段:

- `SID`:区块链服务ID
- `Secret`: CA认证密码。为空时将随机生成
- `OU`: 对应的CA用户类型，`Admin\Client`
- `Attibutes`: (Optional) 额外的用户属性
- ` CAReference `: 组织所属的CA机构
  - `Name`: CA机构名称
  - `CA`: 具体使用哪个CA服务，默认为ca（ca/tlsca）

## 需求

### IAM用户开通区块链服务

1. 更新填充`BlockchainExtendSpec`,包括:

- secret: 设置为`CA`启动时配置的Admin用户密码
- OU: 设置为`admin`
- CAReference: 配置CA

2. controller监测到更新`BlockchainExtendSpec`处理逻辑

- 检测CA是否存在
  - 不存在,则记录错误信息
  - 存在，则`enroll`获得用户证书，生成`k8s secret {user}-crypto-secret`

### IAM Admin用户停止该区块链服务

1. 更新`BlockchainExtend`对应的状态为`Deactivated`
2. controller检测到后：

- 清理`BlockchainExtend`


### IAM用户被加入到由其他人开通的区块链服务

1. 更新填充`BlockchainExtendSpec`,包括:

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
