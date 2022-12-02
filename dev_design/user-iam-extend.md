# Extend IAM with Fabric

经过讨论最终的实现思路是，将CA这边需要的新记录到IAM的User的Annotations字段中。

## 需求

<!--**BlockChain 的 ExternalConf 以下统称为 BlockchainExtend**-->

- IAM用户开通区块链服务

- IAM Admin用户停止该区块链服务

- IAM用户被加入到由其他人开通的区块链服务

- IAM Client用户被踢出该区块链服务

## 接入整体逻辑

**一、开通服务**  
平台用户在baas服务这边，填写自己的基本组织信息。包括:
- Country
- Province
- Locality
- Organization
- OrganizationUnit
- CommonName

上述信息完成后，为组织创建一个namespace, 部署CA服务，同时给当前用户开通在CA的Admin权限。

**二、enroll**  
![enroll](./images/fabric-register-enroll.jpeg)  

Enroll 的作用就是来获取当前用户的集群中ecert，用户在baas平台点击enroll后，将请求发送到CA。
请求头中带着IAM服务认证的token，这样CA这边的服务，根据启动的参数知道，请求是来自IAM，
然后解析请求头，获取`username`和`token`信息，调用获取用户信息的请求。

请求通过代表用户是通过认证的。然后生成相关的信息返回给用户。



## CA enroll 目前基本实现
类似客户端发送请求过来。

```shell
/fabric-ca-client enroll -u https://<ENROLL_ID>:<ENROLL_SECRET><@CA_URL>:<PORT> --mspdir <MSP_FOLDER> --csr.hosts <CSR_HOSTNAME> --tls.certfiles $TLS_CERT

./fabric-ca-client enroll -u https://org1admin:org1adminpw@example.com:7054 --mspdir ./org1.example.com/msp --csr.hosts 'org1,*.example.com' --tls.certfiles ../tls/tls-ca-cert.pem
```

- 按照BasicAuth解析用户名和密码。然后获取CA的实例信息(NOTE)，去查询用户信息。然后利用用户名和密码验证用户的正确性。

- 通过验证后走入enroll逻辑，检查时间，设置CA内容，最后将生成的证书返回。

## IAM对接实现
### CA启动配置
增加一个参数 -i(可以会有参数冲突，需要调整, 或者就直接从配置文件来。) 表明当前CA启动对接的是IAM。
CA对接IAM的基本配置信息
```yaml
iam:
  enabled: true
  url: https://ima.server.svc.cluster.local
  tls:
    enabled: true
    certFiles: root1.pem,root2.pem
```

### CA Enrolle逻辑改动
1. 先修改BasicAuthentication
这个函数主要被用来验证用户的正确性上。首先获取了BasicAuth基本信息，然后调用registry的GetUser接口返回一个用户实体，最后调用User的login确实用户信息。
我们的改动则是，如果没有正常获取到BasicAuth的信息，则从请求头尝试获取`用户名+token`。

从请求头获取这种情况，我们在iam那边没有保存一个iam的client，所以需要在调用GetUser接口的时候
把GetUser函数的第一个参数加一个处理, 将token和username连接起来，作为GetUser的参数。然后这边在实现的时候，根据分割符在拆解。

```go
username, password, ok := r.BasicAuth()
if !ok {
  // TODO: 如果没有找到尝试从请求头中获取用户名+token。
  token := r.Headers.Get("iam-token")
  username := r.Header.Get("iam-user")
  // 组成一个新的username，
  username = fmt.Sprintf("%s:%s", username, token)
  // TODO: 验证正确行
  ca.registry.GetUser(username, nil)

  /// return
  return "", caerrors.NewAuthenticationErr(caerrors.ErrNoUserPass, "No user/pass in authorization header")
}
// Get the CA that is targeted by this request
..
// Get the user info object for this user
ctx.ui, err = ca.registry.GetUser(username, nil)
```

获取的用户信息，需要增加annotations的返回。

2. Enroll逻辑
认证通过后，就拿到了一个用户名，然后进入到enroll的逻辑。填充请求内容，然后生成证书。返回给用户。

### IAM 用户开通与加入
前面已经介绍了**开通服务**基本逻辑，这里补充一点，就是开通后，会将当前用户设置Admin的权限，
在其annotations标注下面的信息
```json
{
  "ca": {
    "orgid": {
      "hf.EnrollmentID":"1",
      //..
    }
  }
}
```

而对于用户加入某个组织，在点击加入后，在User的annotations标注Client身份的注解，然后由用户自己去走enroll逻辑。

