## **Channel 设计**

`Channel`为区块链中的概念，代表一条独立的链，拥有一套独立的账本和状态数据库

### **权限设计**

| 用户类型 | 拥有 | 拥有(条件满足)  |  不拥有  |
| ------ | ---- | ------------- |  -----  |
| ALL Users |  create  |  - | - |
| network.members  |  create/get  |  - |  update/patch/delete |

详细解释:

1. 网络的所有成员组织都可以创建属于该网络的`channel`  
2. `channel`的成员可以查看该通道
3. 网络所有成员都不具备Channel的`update/patch/delete`权限.需要通过`proposal-vote`机制来完成

### **CRD定义**

1. `ChannelSpec`

```
// ChannelSpec defines the desired state of Channel
type ChannelSpec struct {
	// License should be accepted by the user to be able to setup console
	// +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
	License License `json:"license"`

	// Network which this channel belongs to
	// +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
	Network string `json:"network"`

	// Members list all organization in this Channel
	// +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
	Members []Member `json:"members"`

	// Description for this Channel
	// +operator-sdk:gen-csv:customresourcedefinitions.specDescriptors=true
	Description string `json:"description,omitempty"`
}
```

解释:

- `Network`: Channel所属网络
- `Members`: Channel的成员组织
- `Description`: Channel的描述信息

2. `ChannelStatus`

```
// ChannelPeer is the IBPPeer which joins this channel
type ChannelPeer struct {
	NamespacedName `json:",inline"`
	JoinedTime     metav1.Time `json:"joinedTime,omitempty"`
}

// ChannelStatus defines the observed state of Channel
type ChannelStatus struct {
	CRStatus `json:",inline"`

	// Peers has been joined into this channel
	Peers []ChannelPeer `json:"peers,omitempty"`
}
```
