---
title: _Golang 游戏后端
---
# 真实的项目需要什么
## 1.数据库
```	
mysql
mongodb
tidb
时间序列数据库
```

## 2.缓存
```	
redis
memcache
```

## 3.消息处理
```	
mq
kafka
```

## 4.传输相关
```	
json
protobuf
grpc
thrift
```

## 5.认证相关
```
cookie
session
token
jwt
```

## 6.日志
```
ELK
```


# Go游戏服务开发
来源：https://zhuanlan.zhihu.com/p/614144990
https://zhuanlan.zhihu.com/p/614294289


## （一）技术选型
Golang一般指Go，是由Google开发的一种静态强类型、编译型语言。
Go语言入门简单，标准库强大，第三方库丰富，社区活跃，用Go开发一款服务速度很快。越来越多的公司采用Go进行服务端开发。
Go特别适合创业型的小公司，因此本系列教程采用GO语言，由浅入深，一步步实现一套通用的游戏服务。

Go语言版本选择当前最新版本1.20.2。

### 前后端通信协议
客户端与服务端的通信协议，采用WebSocket。

WebSocket是一种在单个TCP连接上进行全双工的通信协议。它允许服务端主动向客户端推送消息和数据，这在游戏中十分重要。因为游戏经常需要通过广播消息，同步各个游戏客户端之间的数据。

服务端采用WebSocket，前端在选型上就很灵活了，可以很方便的支持小程序开发。

游戏内部消息的编码协议使用protobuf，方便后期扩展。

### 后端服务通信
后端会涉及多个服务，采用微服务架构，服务间通信使用RPC。

Go实现RPC有好几种方式，本系列最终选择jsonrpc，对应的库为"net/rpc/jsonrpc"。这个库在部署和使用上都很简单。

### 数据库
后端持久化数据存储，使用redis数据库。

### 前端选择
本系列主要介绍后端服务开发，前端只要能实现调试测试即可，因此仍使用Go语言实现，GUI使用fyne实现简单的测试界面。
实际开发2D可以考虑采用CocosCreater，支持WebSocket连接；3D根据情况选unity或UE5。

### 开发环境
IDE选择VSCode，插件支持的比较多，是免费开源的编辑软件，当然有条件的可以选择更专业的Goland。
VSCode是跨平台的编辑器，在Windows和MacOS上都能很好的支持。
本系列工程主要在Windows上实现。

### 部署环境
服务开发完成后，部署在RockyLinux上。
RockyLinux可以看作是CentOS的替代版本。
CentOS一度是国内主流的服务部署环境，不过在2014年被红帽收购，CentOS 7在2020年第四季度停止更新，并于2024年6月停止维护。以后CentOS stream版本将是一个测试版本，不再具有以前的企业级稳定。
CentOS的创始人Gregory Kurtzer在CentOS被停更后，发起了新的项目Rocky Linux来实现CentOS最初目标，其操作与CentOS一脉相承。
本系列将使用Rocky9作为服务部署环境。


## （二）创建项目
打开VSCode，在工作目录下建立一个新的项目。

后面以斗地主游戏为例，因此项目取名为LordGameSvr。



# Zinx（基于Golang轻量级TCP并发服务器框架）
来源：
https://www.yuque.com/aceld
https://github.com/aceld/zinx
https://www.yuque.com/aceld/npyr8s/bgftov

## Zinx v0.1 基础的Server
### 服务器接口（IServer）
启动服务器：`Start()`

停止服务器：`Stop()`

运行服务器：`Server()`

### 服务器结构体（Server）
* 属性：
	服务器名称：`Name string`
	服务器绑定的ip版本：`IPVersion string`
	监听的ip：`IP string`
	监听的端口：`Port int`

* 方法：
	**实现服务器接口**
	初始化Server模块的方法：`func NewServer(name string) ziface.IServer`
	
## Zinx v0.2 简单的链接封装和业务绑定
### 链接接口（IConnection）
启动连接：让当前的连接准备开始工作：`Start()`

停止连接：结束当前连接的工作：`Stop()`

获取当前链接的绑定的socket conn：`GetTCPConnection() *net.TCPConn`

获取当前连接模块的连接ID：`GetConnID() uint32`

获取远程客户端的 TCP状态 IP port：`RemoteAddr() net.Addr`

发送数据，将数据发送给远程的客户端：`Send(data []byte) error`

### 定义处理链接业务的方法
参数：客户端conn，要处理的数据，要处理数据的长度
`type HandleFunc func(*net.TCPConn, []byte, int) error`

### 链接结构体（Connection）
* 属性：
	当前连接的socket TCP套接字:`Conn *net.TCPConn`
	连接ID：`ConnID uint32`
	当前连接的状态：`isClosed bool`
	当前链接所绑定的处理业务方法API：`handleAPI ziface.HandleFunc`
	告知当前连接已经退出/停止的channel：`ExitChan chan bool`

* 方法：
	**实现链接接口**
	初始化连接模块的方法：`func NewConnection(conn *net.TCPConn, connID uint32, callback_api ziface.HandleFunc) *Connection`

## zinx v0.3 基础router模块（路由）
### Request请求封装（IRequest、Request）
将链接和数据绑定在一起
* 属性
	已经和客户端建立好的链接：`conn ziface.IConnection`
	客户端请求的数据：`data []byte`

* 方法
	得到当前链接
	得到请求的消息数据

### IRouter接口
处理业务之前的方法：`PreHandle(IRequest) {}`

处理业务的主方法：`Handle(IRequest) {}`

处理业务之后的方法：`PostHandle(IRequest) {}`

### BaseRouter结构体
**实现IRouter接口**
处理conn业务之前的钩子方法Hook：
`func (rb *BaseRouter) PreHandle(router ziface.IRequest) {}`
处理conn业务的主方法Hook：
`func (rb *BaseRouter) Handle(router ziface.IRequest) {}`
处理conn业务之后的钩子方法Hook：
`func (rb *BaseRouter) PostHandle(router ziface.IRequest) {}`

### 集成router模块
IServer增添路由添加功能：`AddRouter(IRouter)`
Server类增添Router成员：`Router ziface.IRouter`
Connection类绑定一个Router成员：`Router ziface.IRouter`
在Connection类调用已经注册的Router处理业务

## Zinx v0.4 全局配置模块

### 全局配置模块（util/globalobj.go）
读取用户配置好的json文件
用户使用：服务器应用/conf/......json

定义一个全局的对外`GlobalObj`对象；
```golang
var GlobalObject *GlobalObj
```
提供一个init方法，初始化`GlobalObjec`。`func init() {}`

* **属性**
```golang
type GlobalObj struct {
	/*Server*/
	//当前zinx全局的Server对象
	TcpServer ziface.IServer
	//当前服务器主机监听的IP
	Host string
	//当前服务器主机监听的端口
	TcpPort int
	//当前服务器的名称
	Name string

	/*Zinx*/
	//当前zinx的版本号
	Version string
	//当前服务器主机的最大连接数
	MaxConn int
	//当前zinx框架异常数据包的最大值
	MaxPackageSize uint32
}
```

* **方法**
	加载json：`func (g *GlobalObj) Reload()`
	
## Zinx v0.5 消息封装
### 消息接口（IMessage）
Get....
Set....

### 消息结构体（Message）
* **属性**
消息ID：`Id uint32`
消息的长度：`DataLen uint32`
消息的内容：`Data []byte`

* **方法**
Get....
Set....

### 解决TCP粘包问题的拆包封包的模块（IDataPack，DataPack）
直接面向TCP连接中的数据流，用于处理TCP粘包问题

* **方法**
获取包长度的方法：`GetHeadLen() uint32`
```golang
func (dp *DataPack) GetHeadLen() uint32 {
	//DataLen uint32 (4字节) + ID uint32 (4字节)
	return 8
}
```

封包：
	针对Message进行TLV格式的封装
	1) 写Message的长度、ID、内容
	`Pack(IMessage) ([]byte, error)`
	
```golang
func (dp *DataPack) Pack(msg ziface.IMessage) ([]byte, error) {
	//创建一个存放byte字节的缓冲
	dataBuf := bytes.NewBuffer([]byte{})

	//1.将DataLen写进dataBuf中
	if err := binary.Write(dataBuf, binary.LittleEndian, msg.GetMsgLen()); err != nil {
		return nil, err
	}
	//2.将MsgID写进dataBuf中
	if err := binary.Write(dataBuf, binary.LittleEndian, msg.GetMsgId()); err != nil {
		return nil, err
	}
	//3.将Data数据写进dataBuf中
	if err := binary.Write(dataBuf, binary.LittleEndian, msg.GetMsgData()); err != nil {
		return nil, err
	}

	return dataBuf.Bytes(), nil
}
```

拆包
	针对Message进行TLV格式的拆包
	1) 先读取固定长度的head，得到消息内容的长度`DataLen`和消息的类型`MsgID`
	2) 再根据消息内容的长度`DataLen`，再次进行一次读写，从conn中读取消息的内容
	`UnPack([]byte) (IMessage, error)`
```golang
func (dp *DataPack) UnPack(binaryData []byte) (ziface.IMessage, error) {
	//创建一个从输入二进制数据的ioReader
	dataBuf := bytes.NewReader([]byte{})

	//只解压head信息，得到DataLen和MsgID
	msg := &Message{}

	// 1.读DataLen
	if err := binary.Read(dataBuf, binary.LittleEndian, &msg.DataLen); err != nil {
		return nil, err
	}
	// 2.读MsgID
	if err := binary.Read(dataBuf, binary.LittleEndian, &msg.Id); err != nil {
		return nil, err
	}
	//判断DataLen是否已经超出了配置允许的最大包长度
	if utils.GlobalObject.MaxPackageSize > 0 && msg.DataLen > utils.GlobalObject.MaxPackageSize {
		return nil, errors.New("too Large message Data!!")
	}

	return msg, nil
}
```

### DataPack拆包封包的单元测试
```golang
package znet

import (
	"fmt"
	"io"
	"net"
	"testing"
)

/*
	只是负责DataPack拆包封包的单元测试
*/

func TestDataPack(t *testing.T) {

	/*模拟服务器*/
	// 1.创建socketTCP
	listenner, err := net.Listen("tcp", "127.0.0.1:8999")
	if err != nil {
		fmt.Println("server listen error:", err)
		return
	}
	fmt.Println("server listen")

	// 2.从客户端读取数据 拆包处理
	go func() {
		for {
			conn, err := listenner.Accept()
			if err != nil {
				fmt.Println("server accept error:", err)
				return
			}
			fmt.Println("server accept success")

			go func(conn net.Conn) {
				//处理客户端的请求
				//----拆包的过程----
				dp := NewDataPack()
				for {
					// 1) 第一次从conn读，把包的head读出来
					headData := make([]byte, dp.GetHeadLen())
					_, err := io.ReadFull(conn, headData)
					if err != nil {
						fmt.Println("read head error:", err)
						break
					}
					msgHead, err := dp.UnPack(headData)
					if err != nil {
						fmt.Println("server DataPack (Head) Unpack error:", err)
						return
					}

					if msgHead.GetMsgLen() > 0 {
						//msg是有数据的，需要进行第二次读取
						// 2) 第二次从conn读，根据head的DataLen读取Data内容
						msg := msgHead.(*Message)
						msg.Data = make([]byte, msgHead.GetMsgLen())

						//根据dataLen的长度再次从io流中读取
						_, err := io.ReadFull(conn, msg.Data)
						if err != nil {
							fmt.Println("server DataPack (Data) Unpack error:", err)
							return
						}

						//完整的一个消息已经读取完毕
						fmt.Println("----> Recv MsgID= ", msg.Id, " , dataLen= ", msg.DataLen, " , data: ", string(msg.Data))
					}

				}

			}(conn)
		}

	}()

	/*模拟客户端*/
	conn, err := net.Dial("tcp", "127.0.0.1:8999")
	if err != nil {
		fmt.Println("client dial error:", err)
		return
	}
	fmt.Println("client dial success")

	//创建一个封包对象
	dp := NewDataPack()

	//模拟粘包过程，封装两个msg一同发送
	// 封装第一个msg1包
	msg1 := &Message{
		Id:      1,
		DataLen: 4,
		Data:    []byte{'z', 'i', 'n', 'x'},
	}
	sendData1, err := dp.Pack(msg1)
	if err != nil {
		fmt.Println("client DataPack (msg1) Pack error:", err)
		return
	}
	// 封装第二个msg2包
	msg2 := &Message{
		Id:      2,
		DataLen: 6,
		Data:    []byte{'h', 'e', 'l', 'l', 'o', '!'},
	}
	sendData2, err := dp.Pack(msg2)
	if err != nil {
		fmt.Println("client DataPack (msg2) Pack error:", err)
		return
	}

	// 将两个包粘在一起
	sendData1 = append(sendData1, sendData2...)

	// 一次性发送给服务端
	conn.Write(sendData1)

	//客户端阻塞
	select {}
}

```

### 集成消息封装机制
1.将Message添加到Request属性中；
2.将Connection模块中之前单纯读取byte数据的机制 改成 拆包的按TLV形式读取；
3.给连接提供一个封包机制，将发送的消息进行打包再发送；


## Zinx v0.6 多路由模式
### 消息管理模块（MessageHandler）
支持多路由业务api调度管理的

* **属性**
集合：消息Id和对应的router的关系`Apis map[uint32] ziface.IRouter`

* **方法**
根据MsgId来索引调度的方法：`DoMsgHandler(request ziface.IRequest)`
添加路由方法到集合中：`AddRouter(msgId uint32,router ziface.IRouter)`

### 消息管理模块集成到Zinx框架中
1.将Server模块中的Router属性 替换成 MessageHandler属性；
2.将Server模块中的AddRouter()方法 替换成 调用MessageHandler的AddRouter()；
3.将Connection模块中的Router属性 替换成 MessageHandler属性，修改NewConnection()方法；
4.将Connection模块中之前调度Router的业务 替换成 MessageHandler调度，修改StartReader()方法。


## Zinx v0.7 读写分离模型
1.添加Writer的goroutine
2.添加Reader和Writer之间的通信channel
3.将Reader直接发送给客户端 修改为 发送给Writer
4.启动Reader和Writer一同工作

## Zinx v0.8 消息队列及多任务
1.**创建消息队列**
	增加`MessageHandler`的属性：
	消息队列：`TaskQueue []chan ziface.IRequest`
	worker工作池的数量：从全局配置中获取`WorkPoolSize uint32`
	
2.**创建多任务worker的工作池并且启动**
	创建worker，每个worker都应用一个协程承载，永远阻塞等待与当前worker对应的channel消息；一旦有消息到来，worker应处理当前消息对应的业务，调用DoMsgHandler()
	`StartWokerPool()`
	`startOneWorker(workerID int, taskQueue chan ziface.IRequest)`

3.**将之前的发送消息，改成 把消息发送给消息队列和工作池来处理**
	保证每个Worker所受到的request任务是均衡（负载均衡），让哪个Worker去处理，只需要将这个request	请求发送给对应的TaskQueue就可以了。
	将方法发送给相应的channel。
	`SendMsgToTaskQueue(request ziface.IRequest)`

### 消息队列集成到Zinx框架中
1.开启消息队列及WorkerPool
	保证WorkerPool只有一个，应在创建Server模块时开启
	在`Start()`开启
2.将从客户端处理的消息，发送给当前的Worker工作池来处理：
	`Connection.go`
	在`StartReader()`更改
```golang
if utils.GlobalObject.WorkerPoolSize>0 {
	//已经开启了工作池机制，将消息发送给WorkerPool
	c.MsgHandler.SendMsgToTaskQueue(&req)
}else {
	//根据MessageHandler绑定好的MsgID取执行注册的路由方法
	go c.MsgHandler.DoMsgHandler(&req)
}
```


## Zinx v0.9 链接管理模块
为Zinx框架增加链接个数的限定，如果超过一定量的客户端个数，Zinx为了保证后端的及时响应，而拒绝链接请求。
给Zinx框架提供 创建 连接之后/销毁连接之前 所要提供的一些业务。提供给用户Hook函数。

### IConnManager接口
添加连接：`Add(IConnection)`
删除连接：`Remove(IConnection)`
根据连接Id查找连接：`Get(uint32) (IConnection,error)`
获取总连接个数：`Len() int`
清理并终止全部连接：`ClearConn()`

### ConnManager结构体
* **属性**
已创建的Connection集合——Map：
`connections map[uint32]ziface.IConnection`
保护connection集合的读写锁：
`connLock sync.RWMutex`

* **方法**
添加连接
删除连接
根据连接Id查找连接
获取总连接个数
清理并终止全部连接

### 连接管理模块集成到Zinx框架中
1.将ConnManager加入到Server模块中
	给Server添加ConnManager属性；
	在Accept()后判断当前的链接数量是否已经超出最大值；
	在Server的Stop()调用ConnManager的ClearConn()

2.每次成功与客户端建立连接后，将连接加入到ConnManager
	在Connection模块中增加属性Server()，使当前连接有和Server一对一的关系；
	在Connection模块中的NewConnection()方法中调用ConnManager模块的Add()方法；

3.每次与客户端连接断开后，在ConnManager删除连接
	在Connection模块中的Stop()方法中调用ConnManager模块的Remove()方法；

4.Server模块停止时，清除所有连接

### 链接的创建/销毁的Hook方法顺序
给Zinx框架提供创建连接/消耗连接之前所要处理的一些业务，由用户提供Hook函数。

1.给Server模块添加属性
Server创建链接之后自动调用Hook函数：`OnConnStart func(conn ziface.IConnection)`
Server销毁连接之后自动调用Hook函数：`OnConnStop func(conn ziface.IConnection)`

2.给Server模块添加方法
注册`OnConnStart`方法：`SetOnConnStart(func(IConnection))`
注册`OnConnStop`方法：`SetOnConnStop(func(IConnection))`
调用`OnConnStart`方法：`CallOnConnStart(IConnection)`
调用`OnConnStop`方法：`CallOnConnStop(IConnection)`

3.在Conn创建之后调用`OnConnStart`
在Connection模块的Start()调用

4.在Conn消耗之后调用`OnConnStop`
在Connection模块的Stop()调用


## Zinx v1.0 链接属性配置
给Connection模块添加可以配置属性的功能
* **新增属性**
链接属性集合——Map：`property map[string]interface{}`
保护链接属性的读写锁：`propertyLock sync.RWMutex`

* **新增方法**
设置链接属性：`SetProperty(string,interface{})`
获取链接属性：`GetProperty(string) (interface{},error)`
移除链接属性：`RemoveProperty(string)`

# 基于Zinx的服务器应用——MMO多人在线网游
* 协议的定义
* AOI兴趣点算法
* protobuf数据传输协议
* 玩家的业务
	* 玩家上线
	* 世界聊天
	* 玩家上线广播（上线位置的信息同步）
	* 移动位置与广播
	* 玩家下线

## AOI兴趣点算法
AOI(Area Of Interest)

![enter description here](./img/AOI兴趣点算法.png)

### AOI格子的数据类型（Grid）
* **属性**
	格子id：`GID int`
	格子的左边边界坐标：`MinX int`
	格子的右边边界坐标：`MaxX int`
	格子的上边边界坐标：`MinY int`
	格子的下边边界坐标：`MaxY int`
	当前格子内玩家/物体id的集合——Map/切片：`playerIDs map[int]bool`
	保护集合的锁：`pIDLock sync.RWMutex`

* **方法**
	初始化格子：`func NewGrid(gID,minX,maxX,minY,maxY int) *Grid`
	从格子中添加一个玩家：`func (g *Grid) Add(playerID int)`
	从格子中删除一个玩家：`func (g *Grid) Remove(playerID int)`
	得到当前格子中所有的玩家：`func (g *Grid) GetPlayerIDs() (playerIDS []int)`
	调试使用——打印出格子的基本信息：`func (g *Grid) String() string`


### AOI管理格子（地图）数据类型（AOIManager）
* **属性**
	//地图id
	区域的左边界坐标：`MinX int`
	区域的右边界坐标：`MaxX int`
	X方向的格子数量：`CntsX int`
	区域的上边界坐标：`MinY int`
	区域的下边界坐标：`MaxY int`
	Y方向格子的数量：`CntsY int`
	当前区域中有哪些格子map(key=格子id，value=Grid)：`grids map[int]*Grid`

* **方法**
	初始化AOI管理模块：`func NewAOIManager(minX, maxX, cntsX, minY, maxY, cntsY int) *AOIManager`
	
	得到每个格子在X轴方向的宽度：`func (aoiMgr *AOIManager) gridWidth() int`
	
	得到每个格子在Y轴方向的长度：`func (aoiMgr *AOIManager) gridLength() int`
	
	根据格子id得到当前格子的周围九宫格的格子id集合：
		先判断当前格子id的左边和右边是否有格子；
		再统一判断这些格子的上边和下边是否有格子。
		`func (aoiMgr *AOIManager) GetSurroundGridsByGID(gID int) (grids []*Grid)`
	
	通过x、y横纵轴坐标得到格子的周边九宫格内全部的玩家id：
		`func (aoiMgr *AOIManager) GetSurroundPlayerIDsByPos(x, y float32) (playerIDs []int)`
	
	通过x、y横纵轴坐标得到当前的格子id：`func (aoiMgr *AOIManager) GetGIDByPos(x,y float32) int`
	
	添加一个玩家到一个格子中：`func (aoiMgr *AOIManager) AddPidToGrid(pID, gID int)`
	
	移除一个格子中的一个玩家：`func (aoiMgr *AOIManager) RemovePidFromGrid(pID, gID int)`
	
	通过格子id获取格子的全部玩家id：`func (aoiMgr *AOIManager) GetPidsByGridID(gID int) (playerIDs []int)`
	
	通过坐标将玩家添加到一个格子中：`func (aoiMgr *AOIManager) AddPidToGridByPos(pID int, x, y float32)`
	
	通过坐标将玩家从一个格子中移除：`func (aoiMgr *AOIManager) RemovePIDToGridByPos(pID int, x, y float32)`
	
	通过坐标获取对应的玩家所在的格子id：`func (aoiMgr *AOIManager) GetGidByPos(x, y float32) int`
	
	调试使用——打印当前AOI模块基本信息:`func (aoiMgr *AOIManager) String() string`

* 一些格子和坐标的算法
用格子id得到==格子坐标==：x=gID % CntsX，y=gID / CntsX
用==格子坐标==得到格子id： gID = y * cntsX + x

## 数据传输协议protobuf
### protobuf环境安装
#### protobuf 编译工具安装
1.安装依赖库
`sudo apt-get install autoconf  automake  libtool curl make  g++  unzip libffi-dev -y`

2.在github下载压缩包 protoc-23.0-rc-2-linux-x86_64.zip
https://github.com/protocolbuffers/protobuf

3.在 /usr/local/ 解压
`unzip protoc-23.0-rc-2-linux-x86_64.zip`

4.成功后用命令测试
`protoc -h`

#### protobuf 的 go 语言插件安装
由于protobuf并没直接支持go语言需要我们手动安装相关插件
1.获取 proto包(Go语言的proto API接口)
```
go get  -v -u github.com/golang/protobuf/proto
go get  -v -u github.com/golang/protobuf/protoc-gen-go
```
2.编译 
```
cd $GOPATH/src/github.com/golang/protobuf/protoc-gen-go/
go build
```
3.将生成的 protoc-gen-go可执行文件，放在/bin目录下
`sudo cp protoc-gen-go /bin/`

### 编译protobuf
` protoc --proto_path=IMPORT_PATH --go_out=DST_DIR path/to/file.proto`
其中：
1.  `--proto_path`，指定了 .proto 文件导包时的路径，可以有多个，如果忽略则默认当前目录。 
2.  `--go_out`， 指定了生成的go语言代码文件放入的文件夹 
3.  允许使用`protoc --go_out=./ *.proto`的方式一次性编译多个 .proto 文件 
4.  编译时，protobuf 编译器会把 .proto 文件编译成 .pd.go 文件 

`protoc --go_out=. *.proto`
将本地当前文件夹下的全部proto文件进行编译，编译的结果也放在当前文件夹下

## 玩家上线
### Player玩家模块
* **属性**
玩家ID：`Pid  int32 `
连接信息：`Conn ziface.IConnection`
玩家当前坐标：
```golang
X    float32            //平面的x坐标
Y    float32            //高度
Z    float32            //平面的y坐标
V    float32            //旋转的角度（0-360度）
```

* **方法**
创建一个玩家方法：`func NewPlayer(conn ziface.IConnection) *Player`
玩家可以和客户端通信的发送消息的方法：先将proto Message结构体序列化 转换成二进制，再将二进制 通过zinx框架的SendMsg方法将数据发送给客户端：`func (p *Player) SendMsg(msgId uint32, data proto.Message)`

### 实现上线业务功能
给server注册一个创建连接之后的Hook函数：`func OnConnectionAdd(conn ziface.IConnection)`
给客户端发送MsgID：1的消息
给客户端发送MsgID：200的消息

给Player提供两个方法：
将PlayerID同步给客户端
将Player上线的初始位置同步给客户端


## 世界聊天
1.获取所有在线玩家信息
2.广播消息给所有在线玩家

### 世界管理模块（WorldManager）
作用：可以获取世界的全部玩家信息，和全部的AOI地图信息
* **属性**
当前世界地图AOI的管理模块 AOIManager：`AOIMgr *AOIManager`
当前全部在线的player集合：`Players map[int32]*Player`
保护player集合的锁：`pLock sync.RWMutex`

* **方法**
初始化方法
添加一个玩家
删除一个玩家
通过玩家ID查询player对象
获取全部的在线玩家


## 玩家上线广播
当前玩家上线后，触发同步当前玩家位置信息：
获取当前玩家周围的玩家（九宫格）
将当前玩家的位置通过MsgID：200 发送给周围玩家
将周围玩家的位置通过MsgID：202 发送给当前玩家

## 移动位置与广播

## 玩家下线



# 百万级并发IM即时消息系统
来源：https://www.bilibili.com/video/BV17K41167aa/?spm_id_from=333.788.recommend_more_video.0&vd_source=7326fe06b64e27279f7a29142c1cf0b6

## 能做什么
消息发送与接收
文字、表情、图片、语言、视频
...等

访客模式
点对点私聊
群聊、广播
机器人
...等

心跳检测下线
快捷回复
撤回记录
拉黑
...等

## 需要什么
H5 ajax获取音频
websocket发送消息
js/vue 单页APP
mui/css3
...等

websocket组件转发消息
channel/goroutine 提高并发性
gin，template，swagger
gorm，logger，govalidator
SQL，NoSQL，MQ
...等


# 海量用户即时通讯系统
来源：https://www.bilibili.com/video/BV1ME411Y71o?p=301&vd_source=7326fe06b64e27279f7a29142c1cf0b6

## 需求分析
用户注册、登录
显示在线用户列表
群聊（广播）
点对点聊天
离线留言

# 在Go中使用Redis（go-redis）
https://github.com/redis/go-redis
官方文档：https://redis.uptrace.dev/zh/

## 为什么使用连接池？
来源：https://www.cnblogs.com/CJ-cooper/p/15149273.html
首先Redis也是一种数据库，它基于C/S模式，因此如果需要使用必须建立连接，稍微熟悉网络的人应该都清楚地知道为什么需要建立连接，C/S模式本身就是一种远程通信的交互模式，因此Redis服务器可以单独作为一个数据库服务器来独立存在。假设Redis服务器与客户端分处在异地，虽然基于内存的Redis数据库有着超高的性能，但是底层的网络通信却占用了一次数据请求的大量时间，因为每次数据交互都需要先建立连接，假设一次数据交互总共用时30ms，超高性能的Redis数据库处理数据所花的时间可能不到1ms，也即是说前期的连接占用了29ms，连接池则可以实现在客户端建立多个链接并且不释放，当需要使用连接的时候通过一定的算法获取已经建立的连接，使用完了以后则还给连接池，这就免去了数据库连接所占用的时间。

单线程不是redis的性能瓶颈，对redis而言，有两个性能所在，一个是计算机执行命令的速度，另一个是网络通信性。很显然，执行命令速度不是redis的性能瓶颈，通信才是其瓶颈。据我所知，redis每秒可执行10万次，因此，对于客户端将若干条命令传输给redis服务，命令执行时间和通信时间大概是比等于0，将设以1s举例，几条命令传输时间为40ms，而每秒可执行10万条命令，那么这些命令只是花费1ms来执行，其他39ms时间无事可做，等待下一个命令的到来，其中的间隙，造成redis的闲置。

综上，要提高redis的性能，可以降低单位时间内的通信成本，那么连接池就是一个不错的选择。客户端使用连接词+多线程方案，使得redis服务闲置时间降低，极大的提高了服务效率。

go-redis模块自带连接池，所有参数都是可选的，参数配置说明示例如下：
```golang
package main
 
import (
	"fmt"
	"github.com/go-redis/redis"
	"net/http"
	"net"
	"time"
)
 
var gClient *redis.Client
 
func handler(w http.ResponseWriter, r *http.Request) {
	gClient.Ping().Result()
	printRedisPool(gClient.PoolStats())
	fmt.Fprintf(w, "Hello")
}
 
func printRedisPool(stats *redis.PoolStats) {
	fmt.Printf("Hits=%d Misses=%d Timeouts=%d TotalConns=%d IdleConns=%d StaleConns=%d\n",
		stats.Hits, stats.Misses, stats.Timeouts, stats.TotalConns, stats.IdleConns, stats.StaleConns)
}
 
func printRedisOption(opt *redis.Options) {
	fmt.Printf("Network=%v\n", opt.Network)
	fmt.Printf("Addr=%v\n", opt.Addr)
	fmt.Printf("Password=%v\n", opt.Password)
	fmt.Printf("DB=%v\n", opt.DB)
	fmt.Printf("MaxRetries=%v\n", opt.MaxRetries)
	fmt.Printf("MinRetryBackoff=%v\n", opt.MinRetryBackoff)
	fmt.Printf("MaxRetryBackoff=%v\n", opt.MaxRetryBackoff)
	fmt.Printf("DialTimeout=%v\n", opt.DialTimeout)
	fmt.Printf("ReadTimeout=%v\n", opt.ReadTimeout)
	fmt.Printf("WriteTimeout=%v\n", opt.WriteTimeout)
	fmt.Printf("PoolSize=%v\n", opt.PoolSize)
	fmt.Printf("MinIdleConns=%v\n", opt.MinIdleConns)
	fmt.Printf("MaxConnAge=%v\n", opt.MaxConnAge)
	fmt.Printf("PoolTimeout=%v\n", opt.PoolTimeout)
	fmt.Printf("IdleTimeout=%v\n", opt.IdleTimeout)
	fmt.Printf("IdleCheckFrequency=%v\n", opt.IdleCheckFrequency)
	fmt.Printf("TLSConfig=%v\n", opt.TLSConfig)
 
}
 
func main() {
	gClient = redis.NewClient(&redis.Options{
		//连接信息
		Network:  "tcp",                  //网络类型，tcp or unix，默认tcp
		Addr:     "127.0.0.1:6379", //主机名+冒号+端口，默认localhost:6379
		Password: "",                     //密码
		DB:       0,                      // redis数据库index
 
		//连接池容量及闲置连接数量
		PoolSize:     15, // 连接池最大socket连接数，默认为4倍CPU数， 4 * runtime.NumCPU
		MinIdleConns: 10, //在启动阶段创建指定数量的Idle连接，并长期维持idle状态的连接数不少于指定数量；。
 
		//超时
		DialTimeout:  5 * time.Second, //连接建立超时时间，默认5秒。
		ReadTimeout:  3 * time.Second, //读超时，默认3秒， -1表示取消读超时
		WriteTimeout: 3 * time.Second, //写超时，默认等于读超时
		PoolTimeout:  4 * time.Second, //当所有连接都处在繁忙状态时，客户端等待可用连接的最大等待时长，默认为读超时+1秒。
 
		//闲置连接检查包括IdleTimeout，MaxConnAge
		IdleCheckFrequency: 60 * time.Second, //闲置连接检查的周期，默认为1分钟，-1表示不做周期性检查，只在客户端获取连接时对闲置连接进行处理。
		IdleTimeout:        5 * time.Minute,  //闲置超时，默认5分钟，-1表示取消闲置超时检查
		MaxConnAge:         0 * time.Second,  //连接存活时长，从创建开始计时，超过指定时长则关闭连接，默认为0，即不关闭存活时长较长的连接
 
		//命令执行失败时的重试策略
		MaxRetries:      0,                      // 命令执行失败时，最多重试多少次，默认为0即不重试
		MinRetryBackoff: 8 * time.Millisecond,   //每次计算重试间隔时间的下限，默认8毫秒，-1表示取消间隔
		MaxRetryBackoff: 512 * time.Millisecond, //每次计算重试间隔时间的上限，默认512毫秒，-1表示取消间隔
 
		//可自定义连接函数
		Dialer: func() (net.Conn, error) {
			netDialer := &net.Dialer{
				Timeout:   5 * time.Second,
				KeepAlive: 5 * time.Minute,
			}
			return netDialer.Dial("tcp", "127.0.0.1:6379")
		},
 
		//钩子函数
		OnConnect: func(conn *redis.Conn) error { //仅当客户端执行命令时需要从连接池获取连接时，如果连接池需要新建连接时则会调用此钩子函数
			fmt.Printf("conn=%v\n", conn)
			return nil
		},
 
	})
	defer gClient.Close()
 
	printRedisOption(gClient.Options())
	printRedisPool(gClient.PoolStats())
 
	http.HandleFunc("/", handler)
 
	http.ListenAndServe(":8080", nil)
}

```


# 游戏服务器架构系列 - 浅谈Go语言自研的分布式游戏服务器架构
来源：https://www.jianshu.com/p/84ab097df650

## 架构图

![架构图](./img/游戏服务器架构系列001.png)

1. CDN：负责游戏客户端的代码热更新、游戏内的广告图

2. 版控服务器：负责版本更新控制、区服状态、登录历史

3. 中心服务器：用于控制所有服务器，提供服务注册与发现，服务配置与控制

4. 网关服务器：用于客户端连接与通信，所有消息都会通过网关进行转发

5. 单区服务器：用于处理单区的相关业务

6. 跨区服务器：用于处理跨区的相关业务

7. 镜像服务器：用于处理无状态的业务，如：战斗、匹配

## 通信协议
1. 客户端与版控服务器采用的是HTTP + Json通信协议

2. 客户端与网关服务器采用的是Socket + Protobuf通信协议

3. 中心服务器、网关服务器、单区服务器等内部都是采用Socket + Protobuf通信协议

## 数据存储
1. 游戏数据存储：单区服务器、跨区服务器都是采用MySQL进行游戏数据存储

2. 缓存数据存储：单区服务器为了提高效率，会采用Memcache进行数据缓存

3. 文件数据存储：单区服务器、跨服服务器都会用文件存储来备份一些临时数据；战斗服务器会将战报存储到文件中。

## 中心服务器
中心服务器是个单点，维护着区服信息、节点信息、配置信息，用于其他服务器来进行读取。

1. 只有当中心服务器启动后，其他服务器才能启动并注册上去；

2. 中心服务器在运行过程中，会与其他服务器进行PING，来保持连接，并且收集运行状态；

3. 中心服务器挂了后，其他服务器不会挂，只是不能支持新服务器启动与配置；

4. 中心服务器配套一个可视化的控制台，用于架构全面监控与控制。

## 网关服务器
1. 每个网关服务器可以支持多个区的连接，但一个区只能在一个网关服务器上；

2. 网关服务器不做业务处理，只做鉴权处理与消息转发；

3. 网关服务器进行了简单的限流处理。

## 单区服务
1. 单区服务包括多个单区业务服务器，用于分散业务处理压力，降低单点风险；

2. 单区服务会将玩家状态保持在内存中，以加速业务处理；

3. 单区服务会定时保存玩家的状态到单区数据库中；

4. 单区服务会将部分玩家下线后的状态缓存在Memcache中，用于加速玩家登录。

## 跨区服务
1. 跨区服务也包括多个跨区业务服务器，用于分散业务处理压力，降低单点风险；

2. 跨区服务也会将玩法状态保持在内存中，以加速业务处理，同样会定时存储到跨区数据库中；

3. 跨区服务的跨区规则，是通过中心服务器采用设定规则自动分配；

## 镜像服务
1. 镜像服务包括多个镜像业务服务器，例如：战斗服务器、匹配服务器、聊天服务器等；

2. 镜像服务器的特点是无状态，可以部署多个，玩家请求哪一个都能得到结果，避免单点出问题；

# 游戏服务器架构系列 - 网关服务
来源：https://www.jianshu.com/p/fcf60d64c6c9

## 服务描述
即用于维持玩家客户端的连接，将玩家发的游戏请求转发到具体后端服务的服务器。

## 功能特性
1. 对外开放：即客户端需要知道网关的IP和端口，才能连接上来；
2. 统一入口：架构中可能存在很多后端服务，如果没有一个统一入口，则客户端需要知道每个后端服务的IP和端口。
3. 请求转发：由于统一了入口，所以网关必须能将客户端的请求转发到准确的服务上。
4. 无感更新：由于玩家连接的是网关服务器，只要连接不断；更新后端服务器对玩家来说是无感知的，或者感知很少（根据实现方式不同）。

一般情况下，有了以上4个特性，这个网关就可以用了。
但是如果只有上面4个特性，我们用Nginx做为网关也是可以的，为什么还需要自己做网关？

因为我们的游戏网关还需要具备以下特殊功能：

## 特殊功能
1. Session认证：即能维护客户端是否登录成功的状态，对于未登录的请求，不予以转发，从而预防恶意攻击。
2. 流量限流：游戏经常会遇到DDOS攻击，一个客户端可以通过一个for循环一直给你发请求，所以必须进行限制。
3. 踢下线：游戏维护时，为了让玩家能更新补丁，会将玩家踢下线，重新走一遍登录流程，避免客户端与服务端的数据不一致，造成显示上的BUG。此外客服也需要经常对一些违规的玩家进行踢下线处理。
4. 在线统计：为什么网关来做在线统计呢？因为只有它有所有的玩家连接信息，所以可以轻松统计当前有多少玩家在线。
5. 协议加密：为了避免客户端的恶意攻击，我们需要将请求进行加密，由于统一了入口，所以网关来做非常容易。
6. 心跳检测：用于检测客户端是否已经掉线，一般超过几分钟没有收到心跳请求，则认为客户端已经掉线，直接请求登录数据，让玩家重新走登录流程。

集成以上的功能后，便形成了以下网关服务架构图：

![网关服务架构图](./img/游戏服务器架构系列002.png)
这张图中的路由控制，会根据不同游戏类型会有所变动，其中：

1. 后端服务路由表：维护了后端当前有哪些服务注册到网关了，以及服务对应哪些区服的配置信息。
2. 区服注册表：维护了当前开了哪些区服及区服信息。
3. 终端管理：所有连接上网关的设备或进程都被认为是一个终端，终端会有一个编号，这个编号对应后端服务编号或者玩家编号。当需要转发消息给后端服务或客户端时，就需要从终端管理里找到具体的连接进行消息发送。

## 客户端、网关服务器、后端服务器的交互流程
接下来用图示介绍一下，客户端、网关服务器、后端服务器的交互流程：

![客户端与网关、后端服务的交互流程](./img/游戏服务器架构系列003.png)
Step1：客户端连接网关服务器，然后发送登录请求给网关；

Step2：网关直接将登录请求转发给对应区服的后端服务器进行登录验证；

Step3：后端服务器验证成功后，会返回登录信息给网关；

Step4：网关会根据登录信息维持一个Session映射，用于安全验证和重登判断，然后转发登录信息给客户端；

Step5：客户端收到登录成功消息后，就可以发送业务请求给网关了；

Step6；网关收到业务请求后，会先判断玩家是否登录过，登录过的才转发给后端服务器，并且在协议头增加玩家标记；

Step7：后端服务器收到业务请求后，根据协议头的玩家标记，找到玩家的数据进行业务处理，然后返回给网关；

Step8：网关收到业务回复后，直接返回给客户端；

# 游戏服务器架构系列 - 网关限流
来源：https://www.jianshu.com/p/0bd8587084ef
## 为什么要进行网关限流？

在前面我们介绍的游戏服务端架构中，客户端通过Socket连接直连网关，所有请求都需要经过网关，然后由网关统一进行转发，为了避免玩家的DDOS攻击，所以需要在网关进行限流。

常见的算法主要有计数器限流、令牌桶限流和漏桶限流，这些算法都是单机的算法，正好可以用在网关限流。

## 算法

### 1、计数器限流

严格意义上来说计数器限流不属于限流算法，使用计数器来进行限流，主要用来限制总并发数，比如数据库连接数；只要全局总请求数或者一定时间段的总请求数设定的阀值则进行限流，是简单粗暴的总数量限流，而不是平均速率限流。

### 2、令牌桶算法（Token Bucket）

令牌桶算法是一个存放固定容量令牌的桶，按照固定速率往桶里添加令牌。令牌桶算法的描述如下：

假设限制1秒钟生成2个令牌，则按照500毫秒的固定速率往桶中添加令牌；
桶中最多存放a个令牌，当桶满时，新添加的令牌被丢弃或拒绝；
当有n个请求到达，将从桶中删除n个令牌，接着请求被发送到网络上；
如果桶中的令牌不足n个，则不会删除令牌，该请求将被限流（要么丢弃，要么缓冲区等待）。

![令牌桶算法](./img/游戏服务器架构系列004.png)
### 3、漏桶算法（Leaky Bucket）

漏桶算法是一个存放固定水滴的桶，按照固定速率流出水滴，可以用于流量整形和流量控制，漏桶算法的描述如下：

一个固定容量的漏桶，按照常量固定速率流出水滴；
如果桶是空的，则不需流出水滴；
可以以任意速率流入水滴到漏桶；
如果流入水滴超出了桶的容量，则流入的水滴溢出了（被丢弃，要么缓冲区等待），而漏桶容量是不变的。

![漏桶算法](./img/游戏服务器架构系列005.png)
## 对比

令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看桶中令牌是否足够，当令牌数减为零时则拒绝新的请求；漏桶是按照固定速率从桶中流出请求，当流入的请求数累积到漏桶容量时，则新流入的请求被拒绝

令牌桶限制的是平均流入速率，允许突发请求，只要有令牌就可以处理，支持一次拿5个令牌，6个令牌，并允许一定程度突发流量；漏桶限制的是常量流出速率，即流出速率是一个固定常量值，比如都是2的速率流出，而不能一次是2，下次又是3，从而平滑突发流入速率

基于Golang的算法实现：https://github.com/MaxwellBackend/Games/tree/master/ratelimit


# 游戏服务器架构系列 - 网关协议加密
来源：https://www.jianshu.com/p/26b6f417413a
## 一、概要
网关在游戏服务器架构中充当着很重要的角色,根据不同类型项目承担的功能也不一样,主要的功能有以下几种：

1. 压缩：压缩是一件比较耗时操作,放在网关可以减轻游戏压力；

2. 过滤：过滤主要是识别非正常请求，保护后端服务；

3. 转发：转发是保证客户端消息准确快速到达目标地址,加密来提高客户端与服务器之间通信安全；

4.  加密：这篇文章主要介绍一下游戏加密相关的技术。

## 二、加密算法的分类
加密主要提高破解通信协议的难度,避免协议被识别篡改,加密的技术有很多，大致可以分为以下几种：

1. 可逆加密

    1.1 对称加密：DES、AES、RC4等；

    1.2 非对称加密 ：RSA等；

2. 不可逆加密：如：md5等；

## 三、加密流程
游戏网关通信协议采用可逆加密,使客户端可以解密协议,让客户端与服务器可以安全通信,加密的一般流程如下

1. 握手阶段，基于RSA算法

2. 交换KEY，基于DH算法

3. 数据流加密与解密，可采用：AES,RC4等算法

### 握手阶段（RSA）

客户端与服务器握手阶段主要是验证服务器真伪,这一步可以用RSA来实现(可以省略)

1. 服务器随机一段串A

2. 务器用客户端公钥加密发送客户端

3. 客户端收到后用客户端私钥解密出串A

4. 客户端用服务器公钥加密串B和解密后的串A,发送给服务器

5. 服务器收到后用服务器私钥解密串A和之前服务器生成的对比验证,ok则握手成功

### 交换KEY（DH）

1. 客户端通过Diffie-Hellman算法生成服务器加密,解密种子(服务器加密种子是客户端解密种子,服务器解密种子是客户端加密种子),

2. 服务器通过Diffie-Hellman算法根据加密,解密种子生成加密,解密KEY,客户端加密,解密种子

3. 客户端根据加密,解密种子生成KEY

### 数据流加密与解密

用协商过的KEY,进行数据流加密与解密(aes,rc4)

可以根据需要选择不同的加密算法,安全性高的可以选用AES,数据长度不变的可以选用RC4,

AES和RC4在所有算法中都是比较高效的算法,RC4效率好于AES，但安全不如AES,

对AES,DES,RC4几种算法使用案例及性能对比

Github实现：https://github.com/MaxwellBackend/Games/tree/master/security

作者：MaxwellGames
链接：https://www.jianshu.com/p/26b6f417413a
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# 游戏服务器架构系列 - 服务注册与发现
来源：https://www.jianshu.com/p/6854b62387c2

当架构中有了网关之后，客户端就可以连接上来了。接下来客户端就需要请求游戏业务了，网关负责转发；
那么问题来了，网关怎么知道转发到哪里？

## 一、解决网关转发到“业务服务器”
面对游戏需求，我们的结构如下：

【客户端 * N】 -----请求----> 【网关服务器 * N】 -----请求----> 【业务服务器 * 1】

从上图，我们可以看到有N台客户端请求N台网关服务器，然后转发到1台业务服务器；

按照这种结构，先用最快速的方法，把功能实现了：

> 把业务服务器的“IP”和“端口”写死在网关服务器代码里。

过了两天...

策划说：我们现在需要增加一台战斗服务器，没有战斗的游戏怎么玩？

## 二、解决网关转发到<多台>业务服务器
当网关服务器后面有多台业务服务器时，我们发现写死在代码里，总不是那么好，修改特别不方便。

为了避免后续又增加很多业务服务器，我们决定：

> 用JSON或XML结构把每一台业务服务器的“IP”和“端口”写死在配置（文件或数据库）中。

当增加、修改、删除业务服务器时，直接修改配置文件就搞定了，比写死在代码里好多了。

终于上线了...

运营说：今天怎么线上突然报了很多“战斗请求失败”的问题啊？

先让我追查一番...

终于发现，原来是有1个战斗服务器因为“内存满了”挂掉了，当网关在按照配置文件请求时，每次请求到这个战斗服务器就出现“战斗请求失败”了。

这下该怎么办呢？

话不多说，先修复掉...

过了几天，运营又打电话来，说又挂了...，表示很无奈！！！

## 三、解决业务服务器突然挂掉的问题？
业务服务器挂掉是经常发生的事情，有时候“内存满了”，“磁盘满了”，还有的时候就是代码BUG了。

那么，业务服务器挂了怎么办？我觉得网关应该直接转到其他业务服务器，而不能转到这个挂掉的业务服务器。

如果当前业务服务器不够了，我能动态添加业务服务器去做支援。所以，我们就需要做到：

* 服务注册：当业务服务器启动后，自动注册到配置中心；用于增加服务后，能够立即被调用；

* 服务发现：客户端可以从配置中心获取到所有的业务服务器信息；

* 健康检查：当业务服务器挂掉或无法处理请求时，需要被及时发现，然后从配置中心移除；

* 负载均衡：当某个业务的服务有多个时，可提供负载均衡算法进行选择；

看到上面这几个需求，你是不是怕了，有没有种感觉“项目又要延期了”？

不要怕，我给你推荐几个工具！

1. Etcd：一个高可用，分布式，一致的key-value存储，用来共享配置和服务发现。Kubernetes和Cloudfoundry都使用了etcd；
2. Consul：一个发现和配置服务的工具。客户端可以利用它提供的API，注册和发现服务。Consul可以执行监控检测来实现服务的高可用；
3. Apache Zookeeper：一个常用的，为分布式应用设计的高可用协调服务，最开始Zookeeper是Hadoop的子项目，现在已经顶级项目了。
4. Eureka：Eureka 是 Netflix 出品的用于实现服务注册和发现的工具。 Spring Cloud 集成了 Eureka，并提供了开箱即用的支持。

这些工具都可以解决服务注册与发现，流程如下：

![enter description here](./img/游戏服务器架构系列006.png)
这几个工具的对比如下：

![工具对比](./img/游戏服务器架构系列007.png)

## 四、工具理论 - 服务发现模式（Service Discovery）
服务发现模式分：客户端服务发现（client-side discovery）和服务器端服务发现（server-side discovery）

### 客户端服务发现（client-side discovery）
客户端请求服务注册表，获取一个服务列表；
客户端使用一个负载均衡算法，选择一个服务；
客户端直接请求这个服务。
优点：简单直接；客户端实现负载均衡。

缺点：每一种客户端都要实现一套服务发现逻辑；无法根据服务器使用情况进行负载均衡；

### 服务器端服务发现（server-side discovery）
客户端请求服务端的负载均衡器（如网关）；
负载均衡器去请求服务注册表，利用负载算法选择一个服务；
获取具体服务后，可直接转发，也可发给客户端进行调用。
优点：客户端不需要知道具体的负载算法，不需要实现服务发现逻辑；

缺点：对于负载均衡器的稳定性要求比较高（如果是网关，就要开多台），也增加了整体系统的维护性。

## 五、工具理论 - 服务注册表（Service Registry）
服务注册表是服务发现的关键部分，是一个包含了服务实例的网络地址的数据库，必须是高可用和最新的。客户端可以缓存从服务注册表处获得的网络地址。但是，这些信息最终会失效，客户端会找不到服务实例。所以，服务注册表由一个服务器集群组成，通过应用协议来保持一致性。

## 六、工具理论 - 服务注册（Service Registration）
服务注册模式分：服务实例自己注册（self-registration模式）和其它的系统组件管理服务实例的注册（third-party registration模式

### 自注册模式（The Self-Registration Pattern）
服务实例启动后，自己注册到服务注册表
服务实例停止后，自动从服务注册表注销
当服务实例异常挂掉时，服务注册表通过“过期时间”或“心跳检测”来判断失效
优点：简单，不需要其他组件

缺点：每一种服务实例需要实现注册和注销的代码

### 第三方注册模式（The Third-Party Registration Pattern）
服务实例启动后，由另一个系统组件service registrar负责检测和注册
服务实例停止（或异常挂掉）后，也同上
优点：解耦了服务实例和服务注册表，不需要实现服务注册代码；

缺点：对系统组件service registrar的稳定性要求很高，也增加了整个系统的维护性。

# 游戏服务器架构系列 - 分布式ID生成
来源：https://www.jianshu.com/p/e56c8a4185eb
## 为什么要生成分布式ID？

在复杂分布式系统中，往往需要对大量的数据和消息进行唯一标识。例如在游戏中，游戏数据日渐增长，对数据分库分表后需要有一个唯一ID来标识一条数据或消息，数据库的自增ID显然不能满足需求，那业务系统对ID号的要求有哪些呢？

1）全局唯一性：不能出现重复的ID号，既然是唯一标识，这是最基本的要求。

2）趋势递增：在MySQL InnoDB引擎中使用的是聚集索引，由于多数RDBMS使用B-tree的数据结构来存储索引数据，在主键的选择上面我们应该尽量使用有序的主键保证写入性能。

3）单调递增：保证下一个ID一定大于上一个ID，例如事务版本号、IM增量消息、排序等特殊需求。

4）信息安全：如果ID是连续的，恶意用户的扒取工作就非常容易做了，直接按照顺序下载指定URL即可；如果是订单号就更危险了，竞对可以直接知道我们一天的单量。所以在一些应用场景下，会需要ID无规则、不规则。

### 1、UUID

使用网卡地址、时间戳和随机数进行生成唯一ID，Java中就自带生成UUID的方法。

优点：本地即可生成，不需要网络开销

缺点：字符串占用内存，不自增，对数据库索引不友好，MySQL官方推荐不要使用

### 2、snowflake算法

![enter description here](./img/游戏服务器架构系列008.png)
1位：保留位不用。二进制中最高位为1的都是负数，但是我们生成的id一般都使用整数，所以这个最高位固定是0

41位：用来记录时间戳（毫秒），41位可以表示2^41−1个数字，(2^41−1)/(1000∗60∗60∗24∗365)=69年

10位：用来记录工作机器id

12位：序列号，用来记录同毫秒内产生的不同id

优点：存在自增趋势，只占用64位

缺点：强依赖机器时钟

### 3、Flicker公司的解决方案

使用MySQL的auto_increment自增特性来生成唯一ID。

创建优惠券表：

CREATETABLEDiscount (idbigint(20)unsignedNOTNULLauto_increment,stubchar(1)NOTNULLdefault'',PRIMARYKEY(id),UNIQUEKEYstub (stub))ENGINE=InnoDB

获取ID： 在一个事务中执行如下sql，replace和insert语句区别主要是replace在插入数据的时候，如果数据存在（通过主键和唯一索引来查找）则先删除，然后再进行插入。

START TRANSACTION;

REPLACE INTO Tickets64 (stub) VALUES ('a');

SELECT LAST_INSERT_ID();

COMMIT;

上面这种方法只在单台MySQL上生成ID，从高可用角度考虑，接下来就要解决单点故障问题：可以启用两台数据库服务器来生成ID，通过区分auto_increment的起始值和步长来生成奇偶数的ID。

DiscountServer1// 优惠券服务1

auto-increment-increment =2// 自增值

auto-increment-offset =1// 起始值

DiscountServer2// 优惠券服务2

auto-increment-increment =2// 自增

auto-increment-offset =2// 起始值

优点：充分借助数据库的自增ID机制，提供高可靠性，生成的ID有序。

缺点：强依赖数据库，占用两个独立的MySQL实例，有些浪费资源，成本较高，而且增删MySQL实例很复杂。

### 4、MongoDB的ObjectId

MongoDB中我们经常会接触到一个自动生成的字段：”_id”，类型为ObjectId。上面方法中用到了MySQL数据库时，主键都是设置成自增的。但在分布式环境下，这种方法就不可行了，会产生冲突。为此，MongoDB采用了一个称之为ObjectId的类型来做主键。ObjectId是一个12字节的 BSON 类型字符串。

4字节：UNIX时间戳3字节：表示运行MongoDB的机器2字节：表示生成此_id的进程3字节：由一个随机数开始的计数器生成的值

前9个字节保证了同一秒不同机器不同进程产生的ObjectId的唯一性。后三个字节是一个自动增加的计数器（一个mongod进程需要一个全局的计数器），保证同一秒的ObjectId是唯一的。同一秒钟最多允许每个进程拥有（256^3 = 16777216）个不同的ObjectId。

优点：算法实现思路和snowflake类似，但是相比更消耗空间


# 服务端架构中的“网关服务器”
来源：https://www.ngui.cc/el/1532025.html?action=onClick

借用知乎王明雨知友的一个比喻：
>把服务器想象成饭店，没有网关服务器的情况，就如同每一个厨师服务一桌顾客，从点菜开始到炒菜到上菜到收银，有n个厨师就只能服务n桌顾客。有了网关服务器的话，网关服务器就成了强大的服务员，把招呼，点菜、上菜和收银的活都做了，厨师只需要专心炒菜就行。这样饭店的效率就大大提高了。

1. 这样可以把要承载高并发，高性能任务的网络服务独立出来专门做好，做强（对于http协议的场景，可以直接用nginx做网关服务器）。这样各个应用只需把重点放在对业务逻辑的处理即可。从技术架构和项目协作上都做到了解耦。

2. 增强了系统的健壮性，一个应用出现故障并不会对其他应用产生影响。后期运维也好做灰度更迭。

3. 有应用集群的情况下，可以通过网关服务器做负载均衡，把请求分发在负载低的服务器上。

再引用一个游戏公司对网关服务器的评价：
>服务器架构
采用带网关的服务器架构，将客户端与游戏服务器隔离，相比传统的客户端-服务端直连的架构有如下优势：
（1）作为网络通信的中转站，负责维护将内网和外网隔离开，使外部无法直接访问内部服务器，保障内网服务器的安全，一定程度上减少外挂的攻击。
（2）网关服务器负责解析数据包、加解密、超时处理和一定逻辑处理，这样可以提前过滤掉错误包和非法数据包。
（3）客户端程序只需建立与网关服务器的连接即可进入游戏，无需与其它游戏服务器同时建立多条连接，节省了客户端和服务器程序的网络资源开销。

>服务端高度模块化
大厅服务端将登录、用户信息、房间信息、日常任务、道具、银行、比赛、排行、活动、网站等11个功能拆分成11个独立的服务端子模块，模块之间不会相互影响，即使某模块出错也不会影响全局，提高了服务端的稳定性；与子模块平行的新功能可以自由新增挂载，扩展性强。


# 网络游戏开发中，服务端网关有什么优势和劣势？
作者：咚咔
链接：https://www.zhihu.com/question/21038610/answer/2991873662

优势分几个方面。

功能性：
注：定位跟 nginx 差不多，只是定制的应用层协议……
1. 反向代理，一个地址即可代表所有服务。
2. 负载均衡，且扩容过程对客户端透明。
3. 消息路由，分布式架构下可以把它当消息总线用。
 
安全性：
1. 隔离内外网，对外只暴露网关 IP，减少攻击面。
注：被 DDoS 冲击时 IP 太多会很麻烦，此外上防火墙时 IP 越多越费钱。
2. 统一实现通信安全，报文加密，防篡改、防重放等等。
注：没有网关时，这层会嵌在其它对外通信的进程中，但网关优势在于统一化。
注：我觉得可以用 TLS ，但需为性能妥协加密强度。具体蹲个高手解答。

运维便利：
1. 公网带宽只给网关配，配一个大带宽好过配一堆小带宽（资源利用率）。
2. 流量朝外和朝内，彼此分得更清楚，易于监控盯梢。

劣势嘛，延迟多一次内网通信，实际取决于机房质量和主机距离。但考虑到外网动不动 50ms~200ms 的延迟，这可以接受。


作者：wuxinliulei
链接：https://www.zhihu.com/question/21038610/answer/59804727

1.转发       
转发客户端和服务器间的消息，网关将场景、会话、数据、名字、平台等服务器的数据转发给客户端，接收客户端的数据，把消息转发给相应的服务器。很多时候，我们需要将消息发给所有的用户，如果让会话服务器来执行这个功能，如果这个时候服务器很负载很大，有2万人在线，那么就得发两万条消息给客户端，这个肯定是不允许的，所有我们将需要转发的消息给网关，如果每个 网关挂了4000人，那么网关就将这条消息发给每个用户了。当然了，在游戏设计时，特别是策划在活动制作时，必须尽量避免群发消息的功能。即使要发，我们将这2万人分组，然后分时分组发送。一般情况下，网关与场景是多对多的关系，和其他服务器都是1对多的关系       

2.心跳
网关定时发送心跳给连接在这个网关上的所有客户端，保证客户端与网关的连接，如果某个客户端掉线了，那么网关就通知各个服务器去做玩家的下线处理       

3.负载均衡      
多网关来支持平衡游戏负载，当然负责负载均衡的模块可以放在一个管理所有服务器的Super服务器，       

4.账号管理      
玩家删除账号，更改名字等操作，只需要登录到网关，然后与数据服务器交互就行了，然后还有一个处理就是，对玩家的名字/聊天的消息进行关键词过滤      

5.维护玩家登陆信息      
玩家发来登陆操作时，去数据库服务器寻找账号，匹配密码，然后连接到场景服务器上。很多公司都使用了统一的账号平台来管理所有的游戏账号，这时，玩家登陆游戏时，发一条登陆消息，先登陆到平台，平台验证账号的正确性，然后去相应的游戏服务器，分配一个网关，把这个网关的ip/端口这些发给客户端，客户端去连接这个网关，当然中间的规则，就自己设定了。      

现在做的是端游，等这边结束了，希望能尝试做一个页游。      明天来写会话服务器。


# 基于Token的登录流程
来源：https://cloud.tencent.com/developer/article/1444727

## 一、身份验证
要想区分来自不同用户的请求的话，服务端需要根据客户端请求确认其用户身份，即身份验证

在人机交互中，身份验证意味着要求用户登录才能访问某些信息。而为了确认用户身份，用户必须提供只有用户和服务器知道的信息（即身份验证因子），比如用户名/密码

Web 环境下，常见的身份验证方案分为 2 类：

* 基于 Session 的验证
基于 Session 的方案中，登录成功后，服务端将用户的身份信息存储在 Session 里，并将 Session ID 通过 Cookie 传递给客户端。后续的数据请求都会带上 Cookie，服务端根据 Cookie 中携带的 Session ID 来得辨别用户身份

* 基于 Token 的验证
而在基于 Token 的方案中，服务端根据用户身份信息生成 Token，发放给客户端。客户端收好 Token，并在之后的数据请求中带上 Token，服务端接到请求后校验并解析 Token 得出用户身份，过程如下：

![enter description here](./img/基于Token的登录流程001.png)

token based login

P.S.用户名/密码属于知识因子，另外还有占有因子和遗传因子：

知识因子：用户登录时必须知道的东西都是知识因子，比如用户名、密码等
占有因子：用户登录时必须具备的东西，比如密码令牌、ID 卡等
遗传因子：个人的生物特征，比如指纹、虹膜、人脸等
P.S.Authentication（验证）与 Authorization（授权）不同，前者验证身份，后者验证权限

## 二、token
身份验证中的 Token 就像身份证，由服务端签发/验证，并且在有效期内都具有合法性，认“证”（Token）不认“人”（用户）

Session 方案中用户身份信息（以 Session 记录形式）存储在服务端。而 Token 方案中（以 Token 形式）存储在客户端，服务端仅验证 Token 合法性。这种区别在单点登录（SSO，Single Sign On）的场景最为明显：

* 基于 Session 的 SSO：考虑如何同步 Session 和共享 Cookie。比如登录成功后把响应 Cookie 的 domain 设置为通配兄弟应用域名的形式，并且所有应用都从身份验证服务同步 Session
* 基于 Token 的 SSO：考虑如何共享 Token。比如进入兄弟应用时通过 URL 带上 Token

Token 相当于加密过的 Session 记录，含有用户 ID 等身份信息，以及 Token 签发时间，有效期等用于 Token 合法性验证的元信息，例如：
```
{
  // 身份信息
  user_id: 9527,
  // Token元信息
  issued_at: '2012年3月5号12点整',
  expiration_time: '1天'
}

// 加密后
895u3485y3748%^HGdsbafjhb
```

任何带有该 token 的请求，都会被服务端认为是来自用户 9527 的消息，直到一天之后该 token 过期失效，服务端不再认可其代表的用户身份

Token 形式多种多样，其中，JSON Web Token是一种比较受欢迎的 Token 规范

## 三.JSON Web Token
>JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties.

简言之，一种通信规范（简称 JWT），用来安全地表示要在双方之间传递的声明，能够通过 URL 传输
P.S.声明可以是任意的消息，比如用户身份验证场景中的“我是用户 XXX”，好友申请中的“用户 A 添加用户 B 为好友”

### Token 格式
JWT 中的 Token 分为 3 部分，Header、Payload 与 Signature，例如：
`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLCJleHAiOjEzMDA4MTkzODAsImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlLCJvdGhlckZpZWxkIjoiZXRjLiJ9.BkrYadYHS66ZrONzFoA91gBJK0iE-S2ZYOWCpRhgFJQ`
两个.字符隔开三部分，即：
`Header.Payload.Signature`
含义上，Header表示 Token 相关的基本元信息，如 Token 类型、加密方式（算法）等，具体如下（==alg==是必填的，其余都可选）：
* ==typ==：Token type
* ==cty==：Content type
* ==alg==：Message authentication code algorithm
Payload表示 Token 携带的数据及其它 Token 元信息，规范定义的标准字段如下：

* ==iss==：Issuer，签发方
* ==sub==：Subject，Token 信息主题（Sub identifies the party that this JWT carries information about）
* ==aud==：Audience，接收方
* ==exp==：Expiration Time，过期时间
* ==nbf==：Not (valid) Before，生效时间
* ==iat==：Issued at，生成时间
* ==jti==：JWT ID，唯一标识 

这些字段都是可选的，Payload 只要是合法 JSON 即可

### 生成
Token 的三部分分别为：
`Base64编码的Header.Base64编码的Payload.对前两部分按指定算法加密的结果`
例如，对于
```javascript
// JOSE Header
const header = JSON.stringify({"typ":"JWT", "alg":"HS256"});
// JWT Claims Set
const claims = JSON.stringify({
  "iss":"joe", "exp":1300819380, "http://example.com/is_root":true
});
```
对 JOSE Header 和 JWT Claims Set 分别进行 Base64 编码得到 JWT Token 中的 Header 与 Payload 部分：
```javascript
const tokenHeader = Buffer.from(header).toString('base64');
const tokenPayload = Buffer.from(header).toString('base64');
```


# 分布式与集群的区别是什么？
来源：https://www.zhihu.com/tardis/bd/ans/2469691331?source_id=1001

这是一个极好的问题，解决这个问题，是学习分布式系统的基础。

而确实有很多人，在没有了解清楚这个概念的情况下，开始学习分布式系统的细节。这容易导致一个问题：越学越乱。

所以，我们今天花费一些时间，来分析清楚这个问题。该回答部分截取自我的书籍《分布式系统原理与工程实践》，要想进一步探究的同学可以移步该书籍。（仅为了得知“分布式与集群的区别”的话，则完全不用买，本回答会解释的很详细。）

## 1 概述
“分布式系统”是我们经常遇到的一个概念，而“集群”也是。但是很多时候，大家会将这些概念混淆，一个原因是：分布式系统的定义本身就不明确。

学术界中，对分布式系统的定义并不统一。

* 有的学者将分布式系统定义为 **“一个其硬件或软件组件分布在联网的计算机上，组件之间通过传递消息进行通信和动作协调的系统[1]”；
* 有的学者将分布式系统定义为 **“若干独立计算机的集合，这些计算机对于用户来说就像是单个相关系统[2]”。

显然，这些定义都可以涵盖分布式系统，但又过于宽泛和模糊，与软件开发者日常讨论的分布式系统的概念相去甚远。

工程界中，分布式系统的概念也是模糊的。例如我们会说ZooKeeper是分布式系统，也会说微服务系统是分布式系统。但实际上，两类系统的差别很大（后面分析了两个系统的具体归类，这里我们给出结果：两者都属于分布式系统。而准确地说，ZooKeeper是属于分布式系统中的信息一致的节点集群，而微服务应用本身就是分布式系统的一个子类）。

那我们平时所说的分布式系统到底是什么，其判断标准是怎样的？

接下来我们就要回答上述问题。我们会从应用演化历程的角度介绍应用如何一步步从单体发展到分布式。然后在此基础上，给出分布式系统的确切定义。

## 2 应用的演进历程
在这一章节我们要了解应用如何从单体结构逐渐演变为分布式结构，并详细介绍演变过程中出现的各种结构的优势与缺陷。

### 2.1 单体应用

单体应用是最简单和最纯粹的应用形式，它就是部署在一台机器上的单一应用。单体应用中可以包含很多功能模块，模块之间会互相调用，但这些调用都在应用内展开，十分方便。因此，单体应用是一个高度内聚的个体，其内部的各个模块间是高度耦合的。

单体应用的开发、维护、部署成本低廉，适合实现功能简单、并发数低、容量小的应用。当应用的功能、并发数、容量不断提升时，单体应用的规模会不断增大。这会带来两个方面的挑战：

* 硬件方面。庞大的单体应用需要与之对应的服务器提供支持，这种服务器被称为“大型机”，其购买、维护费用都极其高昂。
* 软件方面。单体应用内模块间是高度耦合的，应用规模的增大使得这种耦合变得极为复杂。这使得应用软件的开发维护变得困难。

这里要单独说一点，很多时候从程序员的角度看，往往觉着“软件方面”的因素是单体应用分布化的主要推动力，其实不是的。从历史角度和宏观角度看，“硬件方面”才是！

因此，当应用的功能、并发数、容量增加到一定程度时，需要对单体应用进行拆分，以便于对功能、并发数、容量进行分散。这就演变成了集群应用。

### 2.2 集群应用
**集群应用可以对应用的并发数、容量进行分散。集群应用包含多个同质的应用节点，这些节点组成集群共同对外提供服务**。这里说的“同质”是指每个应用节点运行同样的程序、有着同样的配置，它们像是从一个模板中复制出来的一样。

为了让集群应用中的每个节点都承担一部分并发数和容量，可以通过反向代理等手段将外界请求分散到应用的多个节点上。集群应用的结构如图所示。

![enter description here](./img/分布式与集群的区别是什么？001.png)

但集群应用也带来了一些新的问题。一个最明显的问题是同一个用户发出的多个请求可能会落在不同的节点上，打破了服务的连贯性。

例如用户发出R1、R2两个请求，且R2的执行要依赖R1的信息（例如R1会触发一个任务，而R2用来查询任务的执行结果）。如果R1和R2被分配到不同的节点上，则R2的操作可能无法正常执行。

为了解决上述问题，演化出以下几种集群方案。

#### 2.2.1 无状态的节点集群
无状态应用是最容易从单体形式扩展到集群形式的一类应用。所谓无状态应用是说假设用户先后发出R1、R2两个请求，则应用无论是否在之前接收过请求R1，总对请求R2返回同样的结果。即应用给出的任何一个请求的结果都和应用之前收到的请求无关。

要想让应用满足无状态，必须保证应用的状态不会因为接口的调用而发生变化。查询接口能满足这点。

无状态节点集群设计简单，可以方便地进行扩展，较少遇到协作问题。但只适合无状态应用，有很大的局限性。

很多应用是有状态的。例如某个节点接收到外部请求后修改了某对象的属性，那后面的请求再查询对象属性时便应该读取到修改后的结果。如果后面的请求落到了其他节点上，则可能读取到修改前的结果。对于这类应用，无法扩展为无状态的节点集群。

#### 2.2.2 单一服务的节点集群
许多服务是有状态的，用户的历史请求在应用中组成了上下文，应用必须结合用户上下文对用户的请求进行回复。例如聊天应用中，用户之前的对话（通过过去的请求实现）便是上下文；在游戏应用中，用户之前购买的装备、晋升的等级（也是通过过去的请求实现）便是上下文。

有状态应用必须要在处理用户的每个请求时读取和修改用户的上下文信息。这在单体应用中是容易实现的，但在节点集群中，这一切就变得复杂起来。其中一个最简单的办法是在节点和用户之间建立对应关系：

* 任意用户都有一个对应的节点，该节点上保存有该用户的上下文信息
* 用户的请求总是落在与之对应的节点上

![enter description here](./img/分布式与集群的区别是什么？002.png)
这种集群的典型特点就是各个节点是完全隔离的。这些节点运行同样的代码，有着同样的配置，然而却保存了不同用户的上下文信息，各自服务自身对应的用户。

虽然集群包含多个节点，但是从用户角度看服务某个用户的始终是同一个节点，因此我们将这种集群称为单一服务的节点集群。

**实现单一服务节点集群要解决的一个问题是如何建立和维护用户与节点之间的对应关系。** 具体的实现有很多种，我们列举常用的几种：

* 在用户注册时由用户自由选择节点。很多游戏服务就采用这种方式，让用户自由选择账户所在的区。
* 在用户注册时根据用户所处的网络分配节点。一些邮件服务采用这种方式。
* 在用户注册时根据用户id随机分配节点。许多聊天应用采用这种方式。
* 在用户登录时随机或者使用规则分配节点，然后将分配结果写入cookie，接下来根据请求中的cookie将用户请求分配到指定节点。

其中最后一种方式与前几种方式略有不同。前几种方式能保证用户对应的节点在整个用户周期内不改变，而后一种方式则只保证用户对应的节点在一次会话周期内不改变。后一种方式适合用在两次会话之间无上下文关系的场景，例如一些登录应用、权限应用等，它只需要维护用户这次会话内的上下文信息。

无论采用了哪种方式，用户的请求都会被路由到其对应的节点上。根据应用分流方案的不同，该路由操作可以由反向代理、网关等组件完成。

单一服务节点集群能够解决有状态服务的问题。但因为**各个节点之间是隔离的，无法互相备份。当某个服务节点崩溃时，会使得该节点对应的用户失去服务。因此，这种设计方案的容错性比较差。

#### 2.2.3 共享信息池的节点集群
有一种方案可以解决有状态服务问题，并且不会因为某个服务节点崩溃而造成某些用户失去服务，那就是共享信息池的节点集群。在这种集群中，**所有节点连接到一个公共的信息池上，并在这个信息池中存储所有用户的上下文信息。** 该应用的结构如图所示。

![enter description here](./img/分布式与集群的区别是什么？003.png)

这是一种常见的将单体应用扩展为多节点应用的方式。通常我们会将服务进程在不同的机器上启动多份，并将它们连接到同一个信息池，便可以获得这种形式的集群。

任何一个节点接收到用户请求，都从信息池中读取该用户的上下文信息，然后进行请求处理。处理结束后，立刻将新的用户状态写回信息池中。信息池不仅可以是传统数据库，也可以采用其他类型。例如可以使用Redis作为共享内存，存储用户的Session信息。

在共享信息池的节点集群中，每个节点都从同一个信息池中读写信息，因此对于用户而言，每个节点都是等价的。用户的请求落在任意一个节点上都会得到相同的结果。

在这种集群中，节点之间可以基于信息池进行通信，进而开展协作。

共享信息池的节点集群通过增加服务节点而提升了集群的计算能力、容错能力。但因为多个节点共享信息池，受到信息池容量、读写性能的影响，应用在数据存储容量、数据吞吐能力等方面的提升并不明显。并且信息池也成了应用中的故障单点。

#### 2.2.4 信息一致的节点集群
为了避免信息池成为整个应用的瓶颈，我们可以创建多个信息池，分散信息池压力的同时也避免单点故障。

为了继续保证应用提供有状态的服务，我们必须确保**各个信息池中的信息是一致的**，这就组成了下图所示的信息一致的节点集群。

![enter description here](./img/分布式与集群的区别是什么？004.png)

通常，我们会让每个节点独立拥有信息池，并且将信息池看作节点的一部分，即演化为下图的形式。这是一种更为常见的形式。

![enter description here](./img/分布式与集群的区别是什么？005.png)

在这种形式的应用中，每个节点都具有独立的信息池，保证了容量和读写性能。同时，因为各个节点的信息池中的数据是一致的，任何一个节点宕机都不会导致整个应用瘫痪。

**应用中的任何一个节点接收到外界变更请求后，都需要将变更同步到所有的节点上。这一同步工作的实施成本是巨大的。** 因此，信息一致的节点集群适合用在读多写少的场景中。在这种场景中，较少发生节点间的信息同步，又能充分发挥多个信息池的吞吐能力优势。

### 2.3 狭义分布式应用
应用从诞生之初便不断发展，在这个发展过程中，应用的边界可能会扩展、应用的功能可能会增加，进而包含越来越多的模块。最终，应用的规模（注意，这里的规模是应用自身的规模，而不是外部请求压力的规模）不断增大。

应用规模的增大会带来诸多问题：

* 硬件成本提升：应用规模的增加会增加对CPU资源、内存资源、IO资源的需求，这需要更为昂贵的硬件设备来满足。
* 应用性能下降：当硬件资源无法满足众多模块的资源需求时，会引发性能下降。
* 业务逻辑复杂：应用中包含了众多功能模块，而每个模块都可能和其他模块存在耦合。应用开发者必须了解应用所有模块的业务逻辑后才可以进行开发。这给开发者，尤其是团队的新开发者带来了挑战。
* 变更维护复杂：应用中任何一个微小的变动与升级都必须要重新部署整个应用，随之而来的还有各种回归测试等工作。
* 可靠性变差：任何一个功能模块的异常都可能导致整个应用不可用。应用模块众多又使得应用很难在短时间内恢复。

以上这些问题**都不能够通过扩展为集群应用来解决。因为集群应用只能减少应用的并发数和容量，并不能够缩减应用自身的规模。

为了解决以上问题，我们可以将单体应用拆分成为多个子应用，让每个子应用部署到单独的机器上，然后让这些子应用共同协作完成原有单体应用的功能。这时，单体应用变成了狭义分布式应用，如图所示。

![enter description here](./img/分布式与集群的区别是什么？006.png)

我们将其称为狭义分布式应用，是为了和后面讨论的概念进行区分。

与集群应用不同，狭义分布式应用中的不同节点上可能运行着不同的应用程序，因此各个节点是异质的。

通过将单体应用拆分为子应用，狭义分布式应用既能将原本集中在一个应用、机器上的压力分散到多个应用、机器上，还便于单体应用内部模块之间的解耦，使得这些子应用可以独立地开发、部署、升级、维护。

在实际生产中，建议优先对大的单体应用进行拆分，将其拆分为狭义分布式应用，然后再对各个子应用分别扩展集群。而不是一上来就对单体应用扩展集群。

当拆分后的狭义分布式应用遇到性能或容量瓶颈时，再有针对性地将并发数过高的子应用按需扩展为集群，如图所示。

![enter description here](./img/分布式与集群的区别是什么？007.png)
这种先拆子应用再扩展集群的方式使得每个子应用能够根据自身所需资源情况进行扩展。例如有的子应用需要扩展计算能力，有的子应用需要扩展存储能力；有的子应用需要布置3个节点，有的子应用只需要布置1个节点。这避免了对大的单体应用进行扩展所造成的资源浪费，更为合理和高效。

### 2.4 微服务应用
在狭义分布式应用中，子应用存在的目的是完成分布式应用中的部分功能。子应用和应用之间存在严格的从属关系。然而，这种严格的从属关系却可能造成资源的浪费。

例如存在一个应用A，它包含着三个子应用，分别是负责完成商品订单管理功能的子应用A1，负责完成库存管理功能的子应用A2，负责完成金额核算功能的子应用A3。当我们需要进行销售金额核算（涉及订单管理和金额核算）时，需要调用应用A。此时，应用A下的子应用A2与这次操作请求无关，它是闲置的。这就意味着，当应用A在执行某些操作时，与操作无关的相关子应用是闲置的，无法发挥其性能。

这就相当于商店只提供汉堡、可乐、薯片组成的套餐，而当我们不需要可乐时，购买这种套餐便造成了浪费。避免浪费的方式是允许我们自由组合购买。

于是，我们可以在进行销售金额核算（涉及订单管理和金额核算）时直接调用子应用A1和子应用A3，而在进行库存资产核算（涉及库存管理和金额核算）时直接调用子应用A2和A3。这样，我们不需要在子应用的外部封装一个应用A，而是直接让各个子应用对外提供服务。外部的调用者则可以根据需要自由地选择服务。这便组成了微服务应用。

**在微服务应用中，每个微服务子应用都是完备的，可独立对外服务的，也可以自由组合后对外提供服务，具有很高的灵活性。** 下图展示了微服务应用的示意图。

![enter description here](./img/分布式与集群的区别是什么？008.png)

每个微服务子应用对各类资源的依赖程度是不同的，被调用的频次也是不同的，因此，我们可以针对每个微服务子应用进行资源配置、集群配置，从而提升每个微服务子应用的性能、资源利用率、容量。

在单体应用内部，任何一个模块都有可能和其他模块存在耦合。而在微服务集群中，每个微服务的内聚性很高，而和其他微服务的耦合较低。因此对于某个微服务而言，只要保证对外接口不变，便可以自由修改内部逻辑。这使得每个微服务可由独立的团队开发、维护、升级，而不需要了解其他微服务的实现细节。这有利于提升应用的成熟度、可用性、容错性、可恢复性。

## 3 分布式系统的定义
“分布式系统”也常被称为“分布式应用”，是软件从业者经常遇到的一个概念。在本书的讨论中，为了和“单体应用”对应，我们也会采用“分布式应用”这一称呼。那它们具体指代什么呢，本节我们将讨论清楚这个问题。

我们平时所说的“分布式应用”包含2.3中所述的狭义分布式应用，但范围更广，是一个广义的概念。

例如，我们会说ZooKeeper集群是一个分布式应用，但是它内部并没有拆分子应用，其各个节点运行的程序、配置是完全相同的（节点中Leader、Follower、Learner的角色划分只是程序运行过程中的中间变量）。因此，准确地说，ZooKeeper集群是一个由同质节点组成的信息一致的节点集群。

例如，我们会说包含订单服务、库存服务、支付服务的电商应用是一个分布式应用。而准确地说，如果所有服务只能作为应用的一部分联合起来对外提供服务，那这是一个狭义分布式应用；如果每个服务既可以独立对外提供服务也可以联合对外提供服务，则这是一个微服务应用。

那我们平时所说的“分布式应用”到底指的什么呢？判断一个应用是否为“分布式应用”的依据是什么呢？

我们在2应用的演进历程中详细介绍了应用从单体发展为分布式的过程，以及期间可能产生的各种形式，如下图所示。

![enter description here](./img/分布式与集群的区别是什么？009.png)
**我们平时所说的“分布式应用”，包含了信息一致的节点集群、狭义的分布式应用、微服务应用三大类，如图所示。**

**判断一个应用是否为分布式应用的主要依据是：应用节点是否使用多个一致的信息池。**

在无状态的节点集群中，不存在存储用户上下文的信息池；单体应用、信息共享的节点集群中都只存在一个信息池；单一服务的节点集群中每个节点都具有一个信息池，但是他们是各自独立的，不需要一致地变更。因此以上这些形式的应用都不是分布式应用。

信息一致的节点集群、狭义分布式应用、微服务应用中都包含多个信息池，每个信息池可以独立提供数据读写能力，但它们又要一致地变更。因此，以上几种形式的应用都是分布式应用。

应用使用多个一致的信息池是分布式应用的重要特点，这意味着应用需要面临分布式一致性问题。

## 4 分布式系统的相关问题
分布式系统的出现确实解决了很多硬件资源、软件规模的问题。但是也带来了很多问题。

这些问题的解决并不简单，这也是大家学习分布式系统时，常感觉混乱和头疼的原因。

这些问题主要有下面一些。

### 4.1 一致性问题
**分布式一致性问题是分布式应用面临的最为突出的问题。**

在单体应用中，应用本身只有一个节点，外部的任何变更请求都由该节点直接处理，并在接下来向外给出最新的结果。

在分布式应用中，应用包括多个节点。外部的变更请求会落到应用的任意一个节点上，随后，外部的读取请求可能会落到其他的节点上。这样，外部可能读取到一个变更前的结果。即出现了读写不一致。

为了避免读写不一致，分布式应用需要及时地将一个节点上的变更反映到所有节点上，即实现分布式应用的一致性。然而实现分布式应用的一致性是一个涉及理论、实践的十分复杂的过程，稍有不慎便会对应用的性能造成影响。

**这里需要大家注意的是，我们常听到“事务的一致性”、“最终一致性”、“一致性哈希”等，它们表述的并不是同一个概念。** 这里就不具体展开讲了，一时半会讲不完。大家感兴趣可以阅读《分布式系统原理与工程实践》。

### 4.2 共识问题
在实现分布式一致性的过程中，首先要确保各个节点对某次变更达成共识，即所有节点都认可这一变更。这就又涉及另一个复杂的问题——共识问题。

对于共识，要单独强调一点：**一致性（Consistency）和共识（Consensus）并不是同一个概念，** 它们在英文中也不是一个单词。但是两者却在很多文章、书籍中被混淆，这一点我们要特别注意。两者具体的区别和联系，大家如果感兴趣可阅读书籍。因为和本问题关系不大，就不再细说里面了。

共识里面有赫赫有名的Paxos算法。Paxos算法较为晦涩。《分布式系统原理与工程实践》的共识章节，不仅给出Paxos算法的提出过程、证明思路、具体内容，还会给出它的实现分析、应用示例，以帮助大家理解该算法。然后书中还进一步剖析了Paxos算法为什么晦涩，并引出Paxos算法的衍生算法Raft算法。Raft算法在工程界得到了广泛的运用。

### 4.3 其他
分布式系统带来的算法方面的问题主要就是上述两点，**但在工程上还带来了很多的问题，例如：节点发现问题（注册中心等等）、节点调用问题（RPC等等）、节点协作问题（分布式锁、分布式事务等等）。**



# 游戏服务器架构的历史、现在以及未来(云游戏)
来源：https://zhuanlan.zhihu.com/p/500831183

## 1.单服务器进程
1978年，英国的一名学生编写了世界上第一个MUD程序《MUD1》，MUD1主要采用文字叙述方式，进入游戏后服务器通过文字告诉你在什么场景，你有什么数据等等交互，比如你输入往上走，程序会告诉你爬到了半山腰
1980年《MUD1》程序接入ARPANET并且程序的源码也共享到了ARPANET，此后《MUD1》出现了众多改编版本，不断完善的《MUD1》也产生了开源的MUDOS。MUDOS采用单线程无阻塞套接字来服务所有玩家，所有玩家的逻辑请求都发到同一个线程去处理，主线程每隔一秒钟更新一次所有对象，玩家数据直接存储到本地磁盘。

## 2.多服务器进程
2000左右，网络游戏已经从文字MUD进入了全面图形化年代。游戏内容的越来越丰富，游戏数据量也越来越大，早期MUDOS的架构变得越来越吃不消了，各种负载问题慢慢浮上水面，传统的单服务器结构成为了瓶颈。因此将服务器拆分成多个游戏逻辑服务器(Game)，将玩家分摊到各个游戏进程中，每个游戏逻辑服务器负责一定数量玩家的服务，如下图。

![enter description here](./images/1689847049881.png)

## 3.多类型服务器进程
早先参与网络游戏玩家较少，游戏内容也比较简单，所有的游戏内容逻辑都在同一个服务器进程中运行。随着图形化技术的发展，网络游戏玩家越来越多，游戏内容越来越丰富，越来越复杂，导致游戏数据也越来越大，因此出现不同类型的服务器进程来服务不同需求。

### 3.1 数据库的拆分
服务器的一个重要功能是存储玩家的游戏数据，以便下次玩家上线时能读取先前的游戏数据继续游戏。早前文字MUD游戏的数据持久化存储使用的是服务器本地文件存储，玩家下线时或者定时将数据存储在EXT磁盘中，这样的存储在逻辑上是没有问题的，但是玩家频繁的上下线会导致服务器频繁的I/O，导致负载越来越大，以及EXT磁盘分区比较脆弱，稍微停电容易发生数据丢失，因此第一步就是拆分文件存储到数据库(Mysql)去，如下图。

![数据库的拆分](./images/1689847164678.png)


#### 3.1.1 数据库代理进程(DB)
将数据库拆分出来后，游戏逻辑与数据存储分离，减少了系统的耦合性。如果各个游戏逻辑服务器(Game)需要取数据的话，可以直接连接数据库获取数据，但是当有大量玩家同时操作时，多个游戏服务器同时存取数据库，出现大量数据重复访问和数据交换，使得数据库成为瓶颈。于是在游戏逻辑服务器(Game)和数据库之间增加一个数据代理进程(DB)，游戏服务器不在直接访问数据库而是访问数据代理进程，数据代理进程同时提供内存的数据缓存，缓存一些热点数据，减少数据库的I/O。如下图。

![数据库代理进程(DB)](./images/1689847204661.png)

#### 3.1.2 内存数据库
互联网技术的开源化，极大的促进了各类专用软件的发展和完善，开源技术出现了一种专用的内存数据库，这类专用内存数据库提供易用方便的存储结构体，而且拥有很高读取性能，因此游戏服务器引进这类专用内存数据库，用于缓存服务器热点数据。目前内存数据库使用比较广泛的是Redis和Memcached。引入内存数据库之后，数据库代理进程(DB)不在缓存数据，而是变为玩家上线时将玩家数据从Mysql数据库中加载到内存数据库中和定时将内存数据库中的数据存储到Mysql数据库中。

![内存数据库](./images/1689847273108.png)


### 3.2 逻辑进程的拆分
上面的结构并没有持续太长时间，由于游戏功能类型的逐渐丰富，为了提升游戏各功能运行速度以及游戏体验，开始出现多种专用服务器类型，比如聊天服务器(IM)，匹配服务器(Match)和排行榜服务器(Rank)等，将这些专用服务器独立出来有几个好处：

* **降低耦合性**：如果所有逻辑在一个进程中运行，某个游戏功能出现故障的话，将会影响到游戏的所有功能，导致游戏不能继续下去。比如将聊天拆成单独的服务器，聊天服务器挂了的话，只会影响聊天，不会影响你的大厅逻辑、场景逻辑和其他逻辑。
* **分摊压力**：像匹配这种功能通常需要大量的玩家放到一个池子里，然后从中选出几个能力值相近的玩家进行对战，这其中涉及大量的排序和比较运算，会导致CPU占用大，从而阻塞其他逻辑的处理。所以一般都会将匹配这样的功能拆成匹配服务器(Match)。
* **保证数据一致性**：典型的就是组队这种，队伍由多个人组合到一起，每个玩家由于压力的负载可能被分配到不同的逻辑服务器(GameServer)中，如果在各自逻辑服务器中维护一个队伍数据，会存在数据不一致的情况，比较难处理，所以较好的方式是拆分队伍功能成组队服务器(Team)，将队伍数据放到一个地方去。

拆分成不同服务器类型后，架构图如下：

![逻辑进程的拆分](./images/1689847361639.png)

拆分成各个不同的服务器之后，同一个玩家客户端可能出现与多个服务器同时进行交互，比如你组队时匹配或者匹配时聊天等，而且有些状态频繁变更，比如玩家频繁匹配和取消匹配。如果每个具体功能客户端都要连接一个对应服务器的话，连接的变更和数据的通信会变的非常麻烦，而且中间状态容易出错，所以开始拆分出一个网关服务器(Gateway)。

![网关服务器(GateWay)](./images/1689847440454.png)
把客户端和服务器之间的网络功能单独提取出来，让客户端统一去连接网关服务器，客户端发送的数据由网关服务器转发到后端各个类型的游戏服务器。而各个游戏服务器之间的数据交换也统一到网关服务器进行交换。

### 3.3 消息队列
上面类型的服务器基本能稳定的为玩家提供游戏服务，但是扩展性非常的差，每个网关服务器(Gateway)要跟所有类型的所有逻辑服务器维持连接，一些大型游戏这样的逻辑服务都可能有好几千个，如果每个都维护一个连接的话，光维护这些连接就会消耗大量的内存和运算，而且每新增一个逻辑服务器都要跟所有网关服务器连接，扩展性很差。所以服务器之间的通信通常都会增加一个消息队列进程，专门用来服务器之间内部转发消息。

#### 3.3.1 全局服务管理器(Master)
老版的服务器通常会有一类全局服务管理器(Master)，全局服务管理器全服只有一个，会存放全服共享或者唯一的数据，早期的数据缓存功能也是由全局服务管理器来实现的，通常全局服务管理器还会存储和管理所有服务器的信息，比如服务器的IP地址、服务器的容量和服务器的增删等，还有就是全局服务管理器(Master)需要与每个服务器进行连接，用心跳维护服务器是否处在可用状态，所以服务器之间的消息转发由它来承担，全局服务管理器(Master)扮演着消息队列的角色。

![全局服务管理器](./images/1689847549918.png)
#### 3.3.2 分布式消息队列
一些分区分服的服务器架构，比如分上海服、北京服和杭州服等，一个分服同时在线人数可能不超过几万人，这样服务器架构差不多够用了。现在基本上很多大型游戏也使用这样的结构，但是对于全国同服或者全世界同服的大服务器结构下，这样的架构存在一个严重的单点问题，因为所有服务器间的消息都要通过这个全局服务管理器(Master)转发，这个服务管理器的压力非常大，只要它崩掉的话，那整个游戏都跑不起来，所以会将单点的消息队列换成分布式消息队列，以减轻单个消息队列的压力。现在外界使用比较广泛的分布式消息队列有kafka、rabbitmq和nats，这些都是工业级的消息队列，功能很完善，但游戏界为了性能和自定义化一些功能，通常是自己写分布式消息队列，比如将Master换成分布式Master。

![分布式消息队列](./images/1689847583074.png)
消息队列一般使用订阅/发布(sub/pub)模式，每个服务器向消息队列订阅自己要监听的主题(Topic)，如果自己有消息发布(pub)的的话，就向消息队列发布，不用管向哪个服务器发布。自己订阅自己需要的，自己发布自己要发送的，很好的解耦了服务器间的耦合。

### 3.4 服务发现
消息队列由单点转换成分布式，消息队列也会从服务管理器中脱离出来，专心做消息转发的功能，那服务管理器中剩下的功能怎么办？剩下的其实就是服务发现的功能：存储各个服务器信息(IP地址，容量，服务器ID，服务器类型等)和管理服务器的增删等功能。

怎么理解服务发现的功能？我举一个实际例子。假设你的游戏已经上线，服务器也正在运行中，到了某个节日，游戏做了一个活动，这个活动做的太好了，吸引了一大批玩家，大量的玩家导致你原有的服务器撑不住，需要在线加服务器，这时候就会出现几个问题：原有的服务器怎么才能知道新加了服务器？新加的服务器怎么知道老服务器信息？服务发现就是要解决这样的问题。

![服务发现](./images/1689847621923.png)
新加的服务器A向服务发现中心注册自己的信息，信息注册后，服务发现中心会将服务器A的信息广播给所有老的服务器，通知老服务器有新的服务器加进来，同时也会向新服务器A发送所有老服务器的信息。

服务发现本质上是一类数据库，这类数据库为了保证数据的高可用会支持容错性。容错性一般由数据库备份保证，也就是一份数据会保存在多个机器上，保证了容错性又会带来一个新的问题，那就是多个机器上的数据是否一致，业务逻辑应该用哪个机器上的数据，数据一致性由一类共识性算法解决，比较出名的是paxos算法及其简化版raft算法，这类算法比较复杂，这里不细说，这两类算法的工业级软件实现是zookeeper和etcd。

![etcd](./images/1689847641086.png)
### 3.5 负载均衡
什么是负载均衡？为什么要负载均衡？
每个服务器由一台计算机组成，单个计算机只有有限计算资源(CPU、内存和网络带宽)，当大量玩家都往一个服务器中发送请求时，服务器因为计算不过来，大量请求数据积累超过内存，这样会直接导致计算机崩溃。虽然可以将服务器换成性能更好的服务器，但单个服务器性能受硬件的限制，计算的有限性很低，而且动态扩展性不强，比如线上要临时加计算资源时，不可能直接关服务器吧。所以一般都是通过软件方法，将输入的网络流量分配到多个计算机中，以平摊计算压力，这个软件方法就是负载均衡，总的来说负载均衡是高效的分配输入的网络流量到一组后端服务器。

一个游戏会有多个阶段需要用到负载均衡，这些阶段都有个共同点就是涉及到服务器的选择和变更，比如登录时要选择哪个服务器登录，RPG场景切换时服务器的切换，匹配成功进行战斗时等等，这些阶段都会涉及到负载均衡。负载均衡针对不同需求场景有不同的负载均衡方法，这里主要写登录服(Login)和网关服(Gateway)的负载均衡。

#### 3.5.1 登录服务器的负载均衡
登录服务器是在线游戏的入口，没有登录服的话，你的游戏客户端进不去游戏，卡在登录页面。登录服的负载均衡方法大概可以分为两种，这两种方法与服务器的架构相关联的，第一类是分区分服的负载均衡，第二类是大服务器下的负载均衡方法，大服务器和分区服服务器主要的区别是：大服务器下所有人(全国或者全世界的人)可以一起比赛，不会出现同一个账号有不同数据，而分区分服的服务器是不同的登录服你的角色数据是不同的，比如你在北京服是100级，那你选择杭州服后是1级。分区分服和大服务器架构最本质的区别是是否公用了数据库。

##### 3.5.1.1 分区分服
这类服务器会显示的让玩家选择在哪个服务器登录，比如在杭州服登录，比较典型的像王者荣耀这种就是使用这种方式，你进入游戏时会推荐几个空闲的服务器给你登录，如果选择已经爆满的服务器登录，会提示你服务器爆满，不能登录，这类负载均衡的方式是由玩家来决定的。

##### 3.5.1.2 大服务器架构
大服务器架构下所有人都可以在一起比赛，在登录时也不会让你选服，只有一个登陆按钮，典型的像吃鸡这种游戏就是使用这种架构。那一个登录按钮是否意味着只有一个登录服务器？所有人都连一个服务器不会爆炸吗？其实登录服还是有多个的，只不过在登陆时客户端登录时程序会自动的帮你选择在哪个登录服登录，登录进去后，所有人共用同一组后端服务器和同一组数据库，所以全部人可以一起比赛。这类负载均衡方式是由客户端计算哪个服务器，计算的方法有很多，比较简单直接的就是在登录服中随机选择一个。

#### 3.5.2 网关服务器的负载均衡
Login服务器是在线游戏的入口，通常为了更好动态的扩展服务器、减少向外界暴露内部服务器的入口以及承担更多人的登录逻辑，登录服务器所处理的逻辑很少，基本上是账号的验证、角色的创建和角色数据的加载，等账号验证完毕后，登录服根据负载均衡算法给玩家分配一个网关服务器，之后游戏的逻辑消息由网关服务器处理和转发。网关服务器的负载均衡方式通常是最小分配，也就是在一定阈值之上从现有的网关服务器中选择负载最小的一个。

![网关服务器的负载均衡](./images/1689847750425.png)
登录成功后，客户端会和登录服务器断开，以减少登录服务器压力以供其他人登录。

### 3.6 战斗服务器
上面的服务器基本上是一个大型房间类游戏服务器的完整结构了，网关服务器用来转发消息，各类内部服务器用来处理各个游戏功能的逻辑。但是所有类型的游戏功能都要经过网关服务器来转发到后端服务器吗？比如游戏的实时战斗功能，像体育竞技，吃鸡，王者荣耀的战斗。如果这类对于实时性要求非常高的战斗功能也经过网关服务器的转发的话，产生的延时是非常高的，你释放一个技能，对面半天才掉血，这非常影响战斗体验。所以一般会有一类战斗服务器(Battle)，客户端直接连接战斗服务器(Battle)，不用网关服务器的转发。

![战斗服务器](./images/1689847814779.png)
## 4. 分布式存储
由于大型服务器数据量比较大，最终的DB和Redis也会换成分布式数据库。下图是现在房间型游戏服务器主要结构。

![分布式存储](./images/1689847844536.png)

## 5. 游戏服务器的未来
玩家习惯的变迁是个缓慢的过程，因此游戏玩法在短时间内也不会有大的变化，这也会导致游戏逻辑服务器架构也不会发生太大的变化。2003年魔兽无缝大世界出来时，刷新了大家对服务器的认知，但是这类游戏服务器架构出来还是太早了，到现在用无缝大世界的游戏也非常少，个人认为无缝大世界在近期不会有太大的发展。

从服务器历史发展来看，服务器发展趋势主要取决于玩家游戏需求以及上下游技术产业链。游戏内容的越来越精细，互联网传输量的越来越大以及云计算的发展壮大，个人认为未来服务器发展是游戏上云以及云游戏。

## 6. 游戏上云
先前的游戏服务部署都需要经历两个阶段，一个阶段是服务器硬件的采购以及服务器部署的配置，另一个阶段是服务器的长期运维。服务器采购以及部属配置的整个流程可能需要数个月之久，而且服务部署配置后需要专门的专家进行管理，这些流程对于一些中小型公司来说是很繁杂的，会造成很大的成本问题，这也就催生出了云计算。

云计算是一种基于互联网的计算方式，通过这种方式，共享的软硬件资源和信息可以按需求提供给计算机各种终端和其他设备，使用服务商提供的电脑基建作计算和资源。有了云计算后，公司可以快速购买和配置所需的服务资源，从而花更多的资源在产品本身上。

游戏上云需要两个技术，一个是虚拟化技术。虚拟化技术是一种资源管理技术，是将计算机的各种实体资源，如服务器、网络、内存及存储等，予以抽象、转换后呈现出来，打破实体结构间的不可切割的障碍，使用户可以比原本的组态更好的方式来应用这些资源。这些资源的新虚拟部份是不受现有资源的架设方式，地域或物理组态所限制。另一个是服务编排技术，一个游戏服务的数量可能有成百上千个，而且不同的服务可能分布在不同机器上，这会给服务管理造成很大的困难，所以需要服务编排技术来解决。

### 6.1 Docker
主要是三个部署方式，传统部署、虚拟机部署以及容器部署
* 传统部署
比较典型的游戏部署方式是服务器运行某个操作系统(如Ubuntu)，然后各个游戏服务以进程的方式直接运行在操作系统上，因为每个进程有自己的地址空间，所以服务间的内存空间是隔离的。这种部署方式的优点是简单直接，服务没有额外的一些运算，可以极大化的利用运算资源。但这种部署方式很难限制同一台物理机上每个服务的资源使用(CPU内核的分配)，存在资源争夺情况，很可能因为某个服务占用较大的计算资源导致其他应用不能正常服务，而且因为服务在同一个宿主操作系统中，各个服务可以通过读取文件感知到对方的存在，存在很大的安全隐患问题。

* 虚拟机部署
一台专用服务器的CPU内核数可能有上百个或者上千个，为了更好利用一台服务器的计算资源，避免不合理的竞争，所以引入一种完全虚拟化技术。虚拟化技术允许在物理机上同时运行多个次级操作系统。我之前用的比较多的虚拟软件是VMware，VMware使用Hypervisor对硬件资源进行虚拟化，在使用VMwave的时候需要配置虚拟机的CPU、内存和网络等资源，配好后在虚拟机中安装对应的操作系统和软件。每台虚拟机可以配置自己要使用的资源，各个虚拟机的资源互不干扰，有一个很好的隔离效果，但是虚拟机需要安装完整的操作系统，导致非常笨重。

* 容器部署
比虚拟机更轻量级的是容器技术，如Docker。容器与虚拟机的主要差别是隔离层不一样，虚拟机在物理层面上进行隔离，然后在之上安装操作系统，容器是直接运行在宿主操作系统上，并不需要在额外安装操作系统。Docker使用Linux隔离技术(CGroup和Namespace)来隔离各个服务，并直接调用操作系统的系统调用接口，从而降低每个容器的系统开销，实现降低容器复杂度、启动快、资源占用小等特征。
虚拟机和容器都使用镜像来存储运行环境。对于Docker来说，应用的源代码与它的依赖都打包在Docker镜像中。一个游戏产品生命周期通常要经过开发团队、测试团队和运维团队。早前没有这样的容器时候，服务的交付和升级都会导致多个团队重复性的安装新的依赖环境，而且一些老上线运营的游戏都不敢轻易升级服务，生怕环境不一致导致服务运行不起来。Docker的出现很好的解决了环境问题，环境一次配置各地运行，这也降低了各个团队的交付成本。

![部署](./images/1689848123166.png)

### 6.2 Kubernetes
Docker容器本身适合运行单个服务，但一个游戏的服务有可能上千个或者上万个，这些服务可以分布在不同物理机上。如果只有几个容器，那管理是非常简单的，但当容器有成千上万个的时候，很可能会导致容器管理和编排变得非常困难。对于一些大公司来说，因为要经常维护大型服务器集群，所以都会有自己写的一套集群管理方案，这些方案包含服务资源配置、服务资源使用监控、健康状态检查以及服务的负载均衡等。

* Kubernetes在游戏界的使用
2014年Google开源了自己的容器编排与管理工具Kubernetes，Kubernetes开始逐步在互联网界扩散，2017年之后大量互联网公司开始应用Kubernetes。但奇怪的是游戏界到现在其实也很少游戏服务使用Kubernetes部署，难道是因为技术原因吗？这个问题其实还有个大学老师专门写了篇论文来探讨(https://www.diva-portal.org/smash/get/diva2:1562637/FULLTEXT01.pdf)，给出的结论是没有技术原因，只是因为习惯问题。结合自己在游戏界的经验，这个结论是正确的，当然还有一些游戏特性原因，游戏相对于互联网产品的主要差异是游戏对网络延时要求更高，使用Kubernetes的话会因为一些额外配置带来网络延时和处理延时。

之前游戏部署更多的是使用脚本部署，一些Kubernetes主要支持的特性，如服务状态监控和负载均衡功能，其实是游戏服务内部功能就要支持的，并不需要外部工具在单独支持，再加上一些大型游戏公司的产品已经上线很久了，相应的服务部署工具也会不断针对性完善，有了更熟悉的专用部署工具，而且Kubernetes本身是有上手成本的，一堆新概念要学习，所以游戏界应用Kubernetes需要一定的过度时间。当然，现在的一些新游戏都开始使用Kubernetes部署，像网易雷火的各个新游戏也都用Kubernetes部署，使用Kubernetes部署可以更方便接入公有云，动态创建游戏服务，更合理分配服务计算资源。

![Kubernetes](./images/1689848180193.png)
* Kubernetes基本概念
Kubernetes的核心概念是pod，一个pod包含一个或者多个共享存储和网络的容器，以及怎样运行这些容器的规范，pod是Kubernetes上最小的部署单元，其他的一些工作负载是多个pod的扩展组合。一个pod典型包含一个运行游戏服务的容器以及一些额外的监控容器和发送log服务容器。

Kubernetes使用YAML来定义pod如何运行，如pod名字、使用端口以及IP、CPU和内存限制。Kubernetes的整个架构图如下。

![Kubernetes基本概念](./images/1689848222238.png)

## 7. 云游戏


# 网络游戏架构的前世今生——网络连接
来源：https://zhuanlan.zhihu.com/p/551138550

## 2.2 网络连接方案
相比于网络同步方案，游戏在网络连接的方案上和其他应用上并没有太大差异。有些轻量级休闲游戏，会选择 HTTP/HTTPS 等短连接方案，少量利用 websocket 全双工做一些主动消息推送业务。这种方案的好处在于，有大量成熟的三方库和参考案例，并且不局限于游戏领域之内，门槛低、方便实现并易于更新迭代。

虽然 HTTP 可以用于大流量的通信场景，但对低延迟通信来说并不是最好的选择，相比之下，主流的游戏网络连接方案还是 TCP 长连接，这是因为短连接会不断的创建和释放连接，既消耗服务器性能又增加了平均延迟。
>即使当今的 http 库通常都自带连接池，短连接的建立和释放并不是一一对应 TCP 的握手和挥手，但应用层不必要的建立和释放过程也会影响程序运行性能，在玩家数量高时尤为明显。

MUD、MUX等游戏会选择 Telnet 等普遍使用的 TCP 长连接协议，是由于这些游戏往往在客户端渲染表现上投入较少，在游戏世界的设计和玩法内容设计上投入较多，所以会使用较为常规的长连接协议减轻客户端工作量。另一个方面原因，这些游戏往往可以通过统一的客户端进行登录游玩，保持协议的统一也是游戏间的互相促进和成就。

客户端界面更用户友好的游戏，还是会以自定义 TCP 长连接协议居多。对于网络安全要求较高的公司和游戏产品，还需要在协议上进行加密。我所服务过的好几家大型游戏公司，内部都有专门的团队在做网络协议、网络加密的更新迭代，这也是游戏背后的网络攻防战（网络安全不属于本栏目的话题，在这里仅作基本介绍）。游戏中的自定义网络协议并不像某些更专业的领域那么严格（如 IOT等，学习看懂都是一件很复杂的事），通常需要考虑以下几个点： 
1. **粘包拆包问题的处理 **
2. **包体的序列化与反序列化**
3. **包头包尾是否需要特定标识**

```golang
// 伪代码展示网络连接的处理逻辑，语法使用 golang
for {
  ... // 预处理判断网络连接是否正常，设置读取 deadline 等
  bytesLength, err := tcpConnection.Read(b) // 从网络连接中读取数据
  if err != nil {
    // 错误异常处理
  }

  if bytesLength > 0 {
    ringbuffer.PushPacket(b[:n]) // 将当前收到的数据包塞入 ringbuffer
    // 一次获取到的网络数据包中，可能是包含数个网络包粘包的结果，循环处理
    for {
      // 通过 codec 解码 buffer 中的包体
      outBytes, err := codec.Decode(ringbuffer)
      if err != nil || outBytes == nil {
        // 暂未收到完整包，或者有错误异常
      }      
      ... // 自定义一些特殊包的处理，例如心跳包等
      agent.OnMessage(outBytes) // 向业务层 agent 通知收到消息事件
    }
  }
}
```

一般来说，上述三个点中只有前两个点是相对必要的。粘包拆包问题并不是游戏行业的特有问题，很多成熟的网络库提供好了现成的方案，如果自己编写的话也不难，只需要选择适合的 codec 即可。例如在包头中带包体长度，或是发送定长包体等。

![前四位存储包体长度的 codec](./images/1689859602188.png)
```golang
// 以带包体长度（LengthFieldBased）为例，b 为序列化后的输出结果
func (codec *Codec) Encode(b []byte) (out []byte, err error) {
  length := len(b) // 获取长度
  out = getLengthBytesByBigEndian(out, length) // 获取大典序的长度 bytes
  out = append(out, b...) // 拼接加码结果
  return
}

func (codec *Codec) Decode(buffer *RingBuffer) (out []byte, err error) {
  // buffer读指针不移位的情况下读取4位，获取包体长度
  lengthBuffer, err := buffer.LazyReadN(4)
  if err != nil {
    return // 没有4位可以读，包不完整
  } else {
    frameLength = getFrameLength(lengthBuffer) // 获取包体长度的数字
  }
  // buffer读指针不移位的情况下读取{包体实际长度位}4位，获取实际包体内容
  body, err := buffer.LazyReadN(frameLength)
  if err != nil {
    return
  }
  buffer.ShiftN(frameLength + 4) // buffer 读指针移动到正确位置
  // 注：这里千万要注意 LazyReadN 的返回值是值拷贝还是引用拷贝，需要复制出一份新的内存数据出来
  out = make([]byte, frameLength)
  copy(out, body)
  return
}
```

序列化与反序列化的方式也有很多，不过从传输的效率和性能上考虑，选择 protobuf 或其他高压缩率的序列化方案是主流，这一点和其他应用不太一样。其他应用可能会选择 json、yaml 等主流通用的序列化方案，这些方案的三方库很多，解决方案也多。但这些往往需要占用更多的网络带宽。我在之前的项目中，一般都是以公司层级去写自己公司的网络序列化库（即公司旗下所有游戏都是公用同一个网络库），这样既安全又高效；不过这两年 protobuf 用的更多，生态和开发效率上都是更好的选择，尤其在给新入职的程序员做介绍时，自己写的库往往要讲半天，protobuf 更好上手，资料肯定比公司自己写文档要全。

![加解码的过程](./images/1689859692412.png)
当然，开发者们对游戏性能的追求是无止境的，这其中也包含网络连接。TCP 作为最主流的传输层协议，在高峰用网期间是会受到一定影响的，近几年来尤其如此；并且由于其设计上的限制，导致在跨国跨洋的场景上往往不尽如人意。这一点无论是对短连接HTTP(S)，还是长连接自定义协议都是如此。为了优化玩家的游戏体验，我们自然把目光放到了另一个耳熟能详的传输层协议 —— UDP 上，希望 UDP 能够优化游戏的网络连接。

最最开始，UDP 只是在有限的游戏流程内进行优化，但很快，部分游戏把 UDP 作为“最终杀器”完全取代了 TCP。老一辈即时战略类游戏如魔兽争霸3，使用 UDP 进行信息的广播；部分有区服概念的 RPG，使用 UDP 进行网速检测、玩家区服信息的传输。MOBA等对网络延迟要求很高的游戏，或是在东南亚等网络条件复杂地区发行的游戏，会使用 UDP 去模拟 TCP 做有状态连接，在第七层应用层做自定义 UDP 协议，从而达到有连接并保序的要求。在我经历过的项目中，使用过 Raknet、ZeroMQ（UDP）、KCP 库进行过自定义 UDP 协议的开发。我个人的经验而言，KCP 是我用起来最顺手，也是测试下来最稳定的开源库，在印度到美国的跨洋连接上也有不俗的稳定表现。

>可以参考我之前练手的一个开源项目https://github.com/finishy1995/gmould/tree/main/network/ucpnet （请勿直接使用这个库在项目代码中，后续许多改进修复并未上传）

在写过测过一些网络连接方案之后，我发现当网络环境较好的情况下，其实 TCP 要明显比 UDP 更快更好，毕竟 TCP 天生就是用作长连接场景的。所以现代 MOBA 类游戏通常都是智能判断的网络连接，一开始使用 TCP 长连接，在检测到玩家连接状况不稳定时（通常是收集玩家的ping值数据，通过方差标准差数学统计的方法监控网络的抖动情况，当监控值比预先设定的值大时，触发警报），自动切换为 UDP 协议，从而保证玩家流畅的游戏体验。

从工程开发角度，也有不少可以演进的点。最早写网络库我用 muduo 做底层，C++ 编写；用 Golang 写项目习惯后，我使用 gnet 做底层；gnet 在 Linux 内核系统上的效率名列前茅，且库内对“网络惊群效应”等问题有较好的支持。对中小型的游戏公司而言，选择一个成熟的网络库，在它之上搭建自己的协议是更为经济有效的方案。

。。。。。。。。

来自朋友的一些提问及回答：

1. Q：为什么没有统一的 TCP 协议支持游戏场景？
A：统一协议降低攻击门槛，公开辛苦开发的协议是一件吃力不讨好的事情，且游戏场景各有不同，很难大一统。

2. Q：大公司大多都有专门的团队做网络库网络协议，如何在更新时尽量不影响玩家？
A：其实有很多还是采取定期停服维护，有少部分是靠支持多版本号的协议来不停服热更。简单来说，更新前网络库是v1.0版本，更新后服务器同时支持v1.0、v1.1 版本即可。下图为自定义协议比对版本号的示例（未兼容多版本）：
![网络库校验版本，可支持多版本](./images/1689860017883.png)


# 网络游戏架构的前世今生——计算
来源：https://zhuanlan.zhihu.com/p/551577166

## 计算方案演进
### 游戏的分类
游戏的分类方式有很多种，从玩家容易理解的玩法角度，我们可以把游戏分为ACT、AVG、TCG、RPG、SLG、FPS、RTS等类别，从战斗流程来划分，可以分为即使战斗、回合制等类别。从游戏开发的角度来看，可以把游戏简单分成两类，会话类（session-based）和非会话类游戏。我最近玩的较多的游戏 糖豆人 就属于会话类游戏。

我们来深入理解一下会话（session）这个概念。会话指的是，区别于一个连续不断的虚拟世界，玩家进入游戏大厅后，会进入一个个的游戏房间，完成一局游戏后回到大厅等待下一轮的游戏。当然，我们也可以把一个连续不断演进的世界看成一个无时间边界的会话，即非会话类游戏。我理解的会话类游戏的前身是桌面游戏（当然传统棋牌也属于广义上的桌面游戏），我们召集到一群玩家，在固定的规则下，为了胜利各司其职；而非会话类游戏的前身是虚拟世界，电脑的发明促使了这一想法的落地，我们可以在一个虚构的世界内自由探索，体验游戏世界的时间流逝，享受虚拟世界带来的快乐。

既然非会话类游戏是一个独立的虚拟世界，世界中也可以塞入一些相对独立的会话类小游戏。很多日系老RPG游戏，会在世界中放入精心设计的关卡、小游戏，以增加游戏世界的丰富程度。巫师三中塞入昆特牌，魔兽世界中塞入的炉石传说，都是很好的例子。非会话类游戏中的pvp玩法，也可以看成一个内置的会话游戏。对于这些游戏，我们按核心玩法分为非会话类，但其中包含的会话玩法，也可以按照会话类游戏的架构方案进行演进。

### 会话游戏的简单架构
前文中已经有所涉及，我们直接从 c/s 架构开始。假设我们开发了一款线上五子棋游戏，最简单的结构是，我们在客户端中写入一系列可用的服务器地址，每次都随机的进入任意一个会话对局中，等待一个对手到来并开始对局。更进一步，我们允许玩家查看每个服务器现在的玩家状况（即是否存在对手，对手的基本信息等），玩家会自行挑选合适的对手进行游戏。

![最基本的架构](./images/1689860470970.png)
>这样的结构很类似早期的线上棋牌游戏大厅，那时候玩家对这种线上游戏乐此不疲。

我们发现，给客户端开放了过多且复杂的能力，这会让整个游戏变得不安全且难以维护，所以我们需要做一个游戏大厅，而把承载游戏会话的服务器“隐藏起来”。同时，我们需要一个匹配（matchmaking）机制，自动分配玩家到实力相近的对局之中。我们将“服务器”这个概念拆解为两个，一个是逻辑服务器（逻辑服，logic server，负责对局外的逻辑处理），另一个是战斗服务器（战斗服，game server，负责对局内的对战信息处理）。

![拆分逻辑服战斗服](./images/1689860515361.png)

至此，我们已经完成了一个基本能玩的会话游戏，可以自由匹配，进入对局完成局内对战，然后再进入新一轮循环。为了增加更多的可玩性和成长玩法，我们可以加入玩家资产（背包）相关的功能，让玩家能在战局之外也能感受到自己在“成长升级”，给予持续不断的正反馈；我们也可以加入玩家数据相关的功能，让玩家可以查询自己的游玩数据，例如总游戏时长、胜利局数等；我们还可以加入别的功能，例如邮件系统、挑战任务系统等等，来进一步丰富游戏的玩法和增加用户粘性。

![实现更多功能](./images/1689860568962.png)
伴随游戏发售后，新一波的挑战随之而来。我们的游戏涌入一大波新玩家，新玩家开始抱怨匹配时间过长/排队等待时间过长，我们需要应对日益增长的玩家数量的要求。同时，由于过多的人涌入大厅，单个大厅应用不堪重负，时长发生崩溃导致停服的事件。所以我们需要扩展我们单体应用的计算能力，以应对持续增长的同时在线人数。

![会话管理模式](./images/1689860610838.png)
上图同时展示了一种当下主流的战斗服管理方式，即由会话管理器（session manager）管理战斗服的生命周期（开启、人员加入、关闭）。根据策略，将一局需要的所有玩家，尽可能分配到最合适的会话对局之中，完成对局后再回收。

### 分服、合服、全球同服
玩家有时候会产生困惑：当前在线玩家很多，为什么我还是匹配不到人？为什么我在大厅遇到的很少是同一个国家地区的人？我们当前只是粗暴的水平扩展了大厅的数量，却无法满足玩家多样化的需求。所以我们决定，把选择权交还给玩家，把一个游戏拆分成多个区服，同一个区服的玩家总会进入同一个大厅，他们总是能见到同样一批队友或对手，互相加好友社交，一起游戏。我们可以按照国家地区进行分服，也可以按照其他的规则进行分服，让玩家可以自行选择区服进入。

![分区分服](./images/1689860670150.png)
分服之后，我们将玩家分隔在一个个独立的区服之中。刚开始一切都很美妙，随着游戏的持续运营，玩家数量走下坡路，在个别区分难免会遇到玩家过少，难以正常游戏的情况。遇到这种情况后，我们选择将两个或多个人较少的鬼服合并，从而保证游戏的正常运营。

>合服的技术难点主要在玩家数据的整合之上，这一点数据库章节会展开讨论。

分服和合服容易在商业游戏中被滥用，逐渐消耗玩家的游戏热情。越来越多的玩家开始渴望，所有玩家能够在一起游玩、社交、组队，游戏内的逻辑边界——区服被舍弃，但硬件的发展远远赶不上玩家对游戏品质的要求。所以在2019年前后，全球同服是游戏业内热门的话题，全球同服的游戏，对玩家更具吸引力，也为玩家带来更好的游戏体验，促进了玩家的留存与活跃。全球同服的需求带来了一系列的技术挑战，分布在全球各地的玩家几乎在24小时内不间断的体验游戏内容，每一个玩家都希望体验到低延迟的流畅体验，每一个玩家都不希望在自己想玩的时候遭遇游戏维护，要求更短的游戏维护窗口。

并不是所有游戏都适合实现全球同服，对于要求极高的实时响应延迟的游戏，通常要求50毫秒内的延迟，例如即时格斗、即时运动类游戏，通常的做法还是划分出多个区服，亚洲服，欧洲服等，服务不同分布的玩家群体。对于那些能忍受100毫秒左右延迟的游戏而言，最简单的做法是逻辑服集中式部署，战斗服分布在全球各地部署，在匹配时收集玩家的延迟数据，尽可能分配到低延迟的战斗服之中。而逻辑服的操作较为低频，玩家对延迟的感受相对较弱，这种架构是最常见的，也是初期应该采用的架构。

![全球同服](./images/1689860732417.png)
如果我们想要进一步优化逻辑服的延迟，我们可以在全球范围内部署多个代理做中转，以达到更好的游戏体验。

![全球同服优化](./images/1689860754727.png)
更进一步想，如果我们把逻辑服分布部署到全球各地，效果会更好，但需要特别注意的去处理数据存储方案，对于跨洋数据同步性问题的处理，可能会导致灾难，需要慎重考虑。我的建议是，除非是上线后其他优化方案还是不满足需求，否则不要轻易尝试。

> 本文只从游戏架构的角度介绍了优化全球同服体验的方案，但在实际开发过程中，游戏渲染、游戏设计的角度也能起到不可忽视的优化作用。例如：我们可以通过技能前摇、放置冷却等数值设置，配合游戏动画的播放时间降低网络延迟的影响。如果技能的前摇是300毫秒，而这个释放技能指令在网络上消耗了130毫秒，那接收到指令的客户端只需要播放剩余的170毫秒前摇动画，就能降低非竞技型玩家对延迟的感知。这些例子在弱竞技场景下还有很多案例。


# 网络游戏架构的前世今生——非会话游戏
来源：https://zhuanlan.zhihu.com/p/552584723

## 3.4 非会话游戏架构
还记得什么是非会话游戏吗？非会话游戏突出的特点就是，这类游戏构造出一个虚拟的世界，这个世界会随着时间的流逝进行演变，RPG（角色扮演类）游戏就属于这个类别。早期的RPG主要以单机虚拟世界为主，后来演化出一个子品类 MMORPG （MMO 即大型多人在线）。MMORPG早从上个世纪末开始就有非常多耳熟能详的作品，热血传奇、魔兽世界、奇迹、暗黑破坏神系列、梦幻西游等，也衍生出大量的端游、手游、页游作品。可以说页游最辉煌的时候，这类游戏占据了大头，直到如今，不少游戏或其衍生品依然在持续运营，玩家群相当广泛。

从技术角度来看，这类游戏的重点是在服务器上构造出一个虚拟的世界，所以世界的内容越多越丰富，就会越多的消耗服务器的CPU和内存资源。当玩家分布在虚拟世界的各个角落时，服务器需要实时计算虚拟世界的演化，并把“结果”传送给不同的玩家客户端。

>在MMO游戏中，单个玩家的客户端只需要知道自己需要知道的信息即可，可以理解为“视野”，但这里不是指玩家视觉能看到的区域，而是一个可被感知的范围，称之为 AOI。在主流 AOI 实现的方式内，很容易出现“超距挂”、“透视挂”等客户端作弊手段 ，其本质是更多的解析服务器传来的消息，通过外置渲染将信息绘制在屏幕上，从而让玩家获得游戏设计内不具备的信息，简化游戏难度。

当我们想要构造的虚拟世界内容不多时（地图可以很大，但这里主要看动态的部分），且单服承载的玩家数量不多，我们可以把一台 物理机/虚拟机 与一个游戏服一一对应，这种架构设计起来和上文中很接近，不同点在于，每个服务器的状态信息需要高效的备份存储。在项目组技术力不足或者架设私服时，通常都会采取这种做法。缺点也很明显，内容匮乏与频繁的滚服。

![游戏服与区服](./images/1689861055744.png)
如果我们想要制作一款内容足够丰富的MMORPG，类似于魔兽世界，我们应该怎么做呢？首先应该意识到，采购性能配置更好的物理机也有穷尽的那一天，垂直扩展是会受到当代硬件整体水平影响的，那我们必须迈出水平扩展的那一步，将一个大世界拆分成有限个小世界，分别在不同服务器上进行运算。

![大世界拆分为多个小地图](./images/1689861082669.png)
看似完美解决了这个问题，但考虑这样一个场景，当玩家在这些小世界中穿梭会发生什么呢？由于这些小世界运行在不同的服务器上，无论程序实现的再怎么完美，都无法避免切换服务器导致的卡顿问题。当然，我们有一些游戏设计的方案，能够降低玩家对这个问题的感知，经典的做法是加载地图。当然加载地图大部分时候是用来解决客户端场景切换设计方面的问题，但加载地图loading界面的时间，足够服务器“瞒天过海”完成切换。

![利用游戏加载页面实现切换地图](./images/1689861112211.png)
问题又来了，如果我们想要制作的是一款无缝大地图游戏呢？魔兽世界就是这样划时代的作品。我们可以巧妙的切分一张大地图，使得存在幽灵（ghost）区，巧妙的解决上述服务器端遇到的问题。当然如果存在超远距离传送的设计，超远距离传送的前摇需要大于切换服务器的时间。

![无缝大地图实现](./images/1689861144278.png)
>客户端用来实现无缝大地图涉及的技术更杂更广，本文不做展开，有兴趣的朋友可以自行查阅相关资料。

上述是理论的方案，有成熟的框架可以直接用吗？如果你做过游戏开发，很可能听说过（或是正在用）BigWorld 框架，魔兽世界早期、大量游戏作品都是采用这一框架。而 BigWorld 的核心理念，其实就是上述理论上更进一步，游戏场景分成多个小块（Cell），不同的块运行在不同的服务器上，这些块的划分很可能是不均匀且动态的，根据玩家密集程度动态调整块的大小。从程序实现上，采用了 Actor 模型，将每个游戏对象抽象为一个个独立的 Entity，其他服务器及客户端都是通过消息的方式与这些 Entity 进行交互。

从一个架构师的角度来看，放在如今 BigWorld 也是一个非常优秀的游戏框架，但它有一个很大的问题——开发时间很高，学习成本也相当高。我曾用这个框架写过游戏，开发过程相当复杂，维护这种老项目需要花费数月的前置时间去学习了解（不仅仅是需要学习框架，还需要学习公司内前人对框架的改造和业务实现），才能真正自如的去改动代码增添新功能。这对于当下的“游戏需要高频迭代及优化”的市场需求是背道而驰的。而这份经历也让我越发感受到，什么是一个真正优秀的架构？今天的我认为，能适应市场需求、能落地产出的才是真正优秀的架构。在项目里，我们必须要把理论和实现结合、把理想和现实结合，才能设计出适合项目组开发的架构。

>这个专栏写作也是一直本着这种目的。在这里我不分享最佳实践，不分享如何产出顶级的应用，而是能让项目组招三流的程序、写二流的代码、出一流的应用，这也是我研究并实践微服务架构的最终目标。我认为一个项目做到顶级很难，需要起码一流的团队，但完成一个一流的项目却不难。所以很多时候必然会在合乎理论和容易实现间做出妥协。

BigWorld 衍生出一些改进版框架，KBEngine 是我用过的其中一个，做游戏服务器性能上确实没问题，项目最后上线也能承载很多玩家。当然缺点也和 BigWorld 一样，虽然支持了 Python 降低了不少开发难度，同时支持了热更，但实际的学习门槛并没有降低（甚至多了一些 Python 的内容需要掌握），这或许是重量级框架的通病。

![KBEgine架构](./images/1689861225047.png)

>这里我原本准备介绍 skynet 框架，skynet 是当前的主流框架之一，大量lua体系游戏公司采用。云风大神的博客给了我不少灵感和领悟，但我本人没有在实践中使用过 skynet，并不能给出实际项目中的评价。感兴趣的朋友可以自行查找资料学习实践。


# 网络游戏架构的前世今生——网关
来源：https://zhuanlan.zhihu.com/p/553042905

## 3.5 网关
2018年，我在上海游戏圈推行网关 Gate 这个概念，也是我第一次从一个架构的学习者，逐步转变成一个架构的实践者的过程。其实现在网关这个概念早已走进民众视野，打开购物网站 app 搜索网关，能搜出一系列的产品，通常是指家用物联网网关。**网关在我的理解中，既是对外实现广域互联的统一入口，也是对内实现局域互联的中转站**，能解决设备或应用变多而导致的组网问题。简单来理解，就是一个网络流量的统一入口，转发到其他实际工作的服务器上。

![IOT物联网中的网关](./images/1689861326647.png)

**在游戏中，网关是对外面向客户端的统一入口**。在玩家完成登录等前置操作后，网关开始发挥作用，理论上网关不执行任何业务逻辑，只做筛选和转发。

![游戏中的网关](./images/1689861354882.png)
如果没有网关，逻辑服直接暴露给玩家，会产生如下几个问题：

1. 逻辑服实例越多，管理越麻烦，切换也麻烦
2. 无法做到无缝切服换服，有等待时间
3. 权限和审查下移，安全风险大大增加

一条一条来看，如果是小体量的游戏，所有逻辑服业务逻辑写成一个单体应用，那对于第一条不会有太大感触。随着业务量的提升，不管是出于功能的分类还是水平的扩展角度，逻辑服实际包含的服务器实例的数量会变多。客户端需要写入更多的服务器地址，或者使用其他地址获取方案。而如果使用网关，客户端只需要记录网关服的服务器地址即可，不管之后游戏功能如何变迁，永远只需要和网关对接，客户端不需要受后端架构影响。

第二条是最容易体会的。传统的切服换服是需要断开长连接返回“登陆后尚未建立长连接”的状态，由派发服 Dispatch （注：这里的“派发服”是我自己取的名，仅供参考）列出当前可以选择的所有区服，择一获取区服连接地址建立连接（有些游戏的区服选择不在UI进行展示，玩家感知较弱，但其本质相同）。这个过程涉及到断开长连接与再建立的过程，不管切换这个过程的UI交互如何设计，必然会面临切换连接的网络卡顿问题，无非是UI交互上如何优雅的提升用户体验。而如果最前面是一层网关层，玩家客户端是与网关建立的长连接，切服换服只是把网关背后的逻辑服实例更换了一下，并不会直接影响到客户端与网关直接的连接，所以可以做到无缝切服换服。

![第二条的优势](./images/1689861409597.png)
第三条是**隐性且容易被忽视的，一个游戏在被 DDos 攻击后，被其他各种各样的手段入侵后，才会去考虑这一系列的问题**。这时候游戏架构已经成型，无法大改，只能选择接入一些安全产品，能提升多大的防护力不得而知。如果我们将逻辑服的业务逻辑直接暴露在公网之上，想要做安全策略需要在每一个服务上编写，例如审计、IP黑白名单、用户权限、异常记录、封禁等。这样不仅工作量大，也很难从工程上确保每个服务都保质保量的做好安全策略，即**攻击面分散，易被各个击破**。我们只需要缩小直接面对敌人的接入面，就能将防守的精力集中，才有可能做好这件事，即**最小化攻击面**。回头再看看上面网关的架构，网关作为对外的统一入口，其实也承担了安全防护的职责。我们只需要在网关上编写我们的安全策略即可，当然也可以在网关上集成第三方的安全服务，事半功倍。

>我一直认为网络安全是一场攻防战，不存在最锋利的矛，也不存在最坚固的盾。最小化攻击面实则为工程实现提供了一种可能——只需要在一处地方编写安全策略，从而防护所有服务的可能，这并不意味着不去编写安全策略就能防护，更不意味着编写了安全策略就能防住。需要有正确合适的心理预期在这件事上。

由上面几点引申而来，分出网关服这一方案有一个突出的优点，这个方案实则是从强有状态的逻辑服中剥离出一块**无状态**的部分。为了追求接口的处理速度和返回效率，在游戏工程项目上逻辑服都是有状态的，每个用户的状态信息被储存在内存中，且优先被读取使用。而网关服要做的事和游戏业务逻辑无关——筛选和转发，是很容易写成无状态的。这里的无状态不是严格意义上的无状态，而是网关服不存游戏玩家信息，可以轻易切换、关闭而不用担心会丢失信息。这一特性决定了，网关服能直接水平扩展，即可以开任意多个，可以自由的在高峰期扩展数量、在低谷期收缩数量，可以在 DDoS来临时快速启动更多的实例扛住并筛选掉攻击流量，保证后方业务逻辑正常运转。当然这是建立在物理机或虚拟机资源无限的情况下，实战中还需考虑成本。

>网关中唯一的“状态”是用户的长连接信息，网关1保存的客户端连接和网关2中保存的客户端连接完全不同。但这个信息是可被丢弃的，客户端只需要和其他网关重新建立长连接即可。

![网关是无状态的](./images/1689861526660.png)
**在游戏中，网关也是对内实现局域互联的中转站**。游戏通讯是全双工的，逻辑服某一块业务例如邮件系统，有主动发送内容给客户端的需求，而不是请求-响应（request & response）模型。逻辑服中这样的系统有很多，如果逻辑服实例各自为战，每一块代码都是自行通知客户端，则整个网络拓扑复杂。我们可以通过网关来实现一个中转，从而实现逻辑服业务层隐藏在接入层之下，应对策划等需求方不断的业务调整。

![ServerCall服务器主动推送消息](./images/1689861570069.png)
当然，网关也可以**作为逻辑服内实例的中转站、逻辑服与战斗服的中转站**。逻辑服与战斗服的交互可以很好的借助网关来完成，同时能利用上述的鉴权等安全措施防止伪造战斗服。同时也进一步解耦战斗服和逻辑服，中间由无状态的网关隐去双方的业务实现。至于逻辑服内部的中转，实用性不大，有更好的服务发现方案运转这个体系，后文会有详细介绍。

![战斗服和逻辑服都通过网关调用逻辑接口](./images/1689861613796.png)


# 网络游戏架构的前世今生——热更
来源：https://zhuanlan.zhihu.com/p/555471062

## 3.6 停服与热更
把眼光跳出游戏行业之外，停服维护是大多数应用更新版本时的选择。一个项目周期推进完毕后，修复了一些已知bug，增加了一些新的功能，选择在线用户较少时更新服务器版本（通常是在半夜）。有些应用会提前发出维护公告，或是每周固定一个时间段进行维护。也会遇到突发状况，例如临时发现紧急bug需要尽快修复上线，这种突发状况会导致应用的某些页面或模块会处于无法正常响应用户请求的状态，日常使用手机、电脑的过程中也不少见。

这是最容易实现的更新方案。我们在本地开发好一个新的版本，为了避免出错，最简单的办法就是关闭老版本的服务器，等客户端全部下线后，再部署并开启新版本的服务器，经过内部人员的测试通过后，再开放给用户正常使用。更进一步，我们可以准备一个只对内部开放的灰度服，则可以尽量降低停服维护的时间。

![网页应用通过域名解析切换版本](./images/1689861705834.png)
游戏服务器的更新也是如此。回到之前五子棋的例子，我们计划将原先的匹配功能，拆分为休闲模式和积分匹配模式两种，于是我们花费了两周的时间开发、测试，准备更新版本。我们的五子棋拥有网页客户端、手机客户端等不同客户端的接入。

对于网页客户端，我们可以提前准备好网页的分发，停服维护后切换域名解析即可。对于手机客户端，需要提前打包好游戏，上传 app 分发平台审核，并预先设置好发布时间。偶尔会遇上客户端打包太晚、或是审核不能及时通过的情况，保险起见我们让开发注意新的服务器版本兼容当前版本，即让当前版本也能正常游戏。

我们成功完成了第一次停服更新。过了一天后，我们发现了一个严重的 bug，导致休闲模式无法正常匹配，需要紧急修复。于是我们临时发送了一个停服维护的通告给全体玩家，然后按约定时间开始停服维护。维护后不久，我们又发现了积分匹配模式的严重 bug，这一次需要客户端和服务器都进行更新。而手机客户端的更新流程较长，玩家在这期间怨声载道，我们需要解决这一问题。

原先我们使用一种语言编写游戏客户端，UE4 C++ 或是 Unity C#，为了赋予客户端热更的能力，我们需要改造这一架构。我们**将手机客户端的编写分为两层，一层是运行高效的非脚本语言，另一层是方便热更新的脚本语言，例如常见的 C# + Lua、C# + JS 等**。将底层的模块由非脚本语言编写，例如网络模块、渲染、场景切换等，而将用户逻辑层的模块由脚本语言编写，例如游戏逻辑 Gameplay、商城页面等。而这种切分方案也符合二八定律，八成的代码是脚本语言，拥有方便热更新的能力，但同时只有不到二成的时间在运行；二成的代码是非脚本语言，没有热更新的能力但稳定高效，超过八成的时间在底层网络、渲染、交互上。脚本语言的上手门槛低和调试门槛低，更适合一个高速迭代的游戏开发团队使用；而非脚本语言的内容，往往会提炼成一套公司特有的游戏框架，经历项目的考验和修复，不会轻易大改。

>早期的应用市场支持 C# 的热更，但现在根据平台的不同有不同的限制，使用 Lua 等脚本语言是主流。
游戏客户端的热更偏离了本专栏的重点，可以查阅其他书籍资料查看这块内容。

![通过脚本完成热更](./images/1689861796788.png)

我们解决了客户端的问题，绝大多数时候都能通过热更来更新客户端。少数情况下需要动到非脚本语言的代码，例如游戏大版本更新涉及到大功能模块的调整，或是网络库的更新。玩家的抱怨减少了，但在持续运营一段时间后，通过分析数据我们发现了新的问题——每次停服更新都会导致玩家在线人数骤降，频繁的停服对玩家的留存有明显的负面影响。

>这个问题是我个人几款游戏的经验，不代表大部分游戏的情况。事实上，我最近玩过的国内几款手游，都是处于每周一次停服维护的状态，偶尔会有临时的停服维护发公告，并没有解决上面这个问题。

我们需要不停服维护，热更服务器。如果是会话类游戏，那战斗服务器不需要热更，每局游戏时长有限，下一局使用新版本即可，只需要考虑逻辑服的热更；而非会话类游戏没有明显区分。

我们可以**仿照游戏客户端的做法，将大部分业务逻辑分拆出去，使用脚本语言编写**。例如使用 KBEngine 框架，用 C++ 编写底层架构，用 Python 编写业务逻辑，这在通常的更新迭代中就够用了。我以前使用过 Cocos 做客户端，那么服务器使用 JS 作为脚本语言，可以共用部分代码。同理，Lua本身就很适合做服务器脚本语言，且性能是这几个方案中最高的，一般全球同服方案最简单的实现，就是借助 Nginx+Lua 快速投入生产使用。

即使采用了上面的方案，我们还是需要定期停服维护服务器，因为那些非脚本语言的更新还是需要重新编译重新启动。有没有更好的方案？

滚动更新 （Rolling Update）能真正做到这一点，并且不需要引入任何脚本语言，最大化服务器的性能。具体方案如何？且待下一章 《微服务》 分解。


# 网络游戏架构的前世今生——微服务
来源：https://zhuanlan.zhihu.com/p/557412275

## 3.7 微服务
记得两年前，在知乎上看到一则回答，详细阐述了网络游戏不能应用微服务的原因。当时的我正好在 aws 研究微服务在游戏行业的应用，看到了不少日本游戏制作商应用微服务的案例，产生了想在知乎上回复的想法。但我知道网络上重拳出击没有任何意义，语言是苍白的，这些公司案例大部分都不能直接对外讲。而且最关键的一点是，我并不清楚内部的实现细节，在上海给数家游戏公司去讲游戏微服务时，我的内心是虚的，究竟怎么落地生产？工程上如何执行？那时候的我也不知道如何去实现游戏微服务。我只知道很多游戏公司用容易来测试、部署游戏后端，但容器并不是微服务——容器只是技术手段，微服务是理念架构。

是什么（what）、为什么（why）、怎么做（how），是我思考一个问题的基础逻辑，思考游戏微服务也是这个逻辑。在本章中我们重点解决是什么和为什么的问题，而怎么做是整个专栏的话题。

首先，微服务是什么？
我所理解的微服务是，将单一应用程序划分成一组小的服务，**服务满足低耦合、高内聚的特性**。每个服务运行在其独立的进程中，服务与服务间采用轻量级的通信机制互相沟通。每个服务都围绕着具体业务进行构建，并且能够独立地部署到生产环境、类生产环境等。

![微服务](./images/1689924270389.png)
关于微服务的定义还有很多阐述，例如有说“应尽量避免统一的、集中式的服务管理机制，对具体
的一个服务而言，应根据上下文，选择合适的语言、工具对其进行构建。”在我看来，这些表述是
表象，或者说是扩展后的微服务定义。但实则，微服务是对单体应用程序的一种拆分方式，我们上
文讲的网关服，也是从逻辑服中拆分的一种方式。这种拆分方式唯一需要满足的特点是——**低耦
合、高内聚**。

**服务与服务之间低耦合**。游戏组队功能故障，不影响正常匹配，匹配时不让玩家组队就可以。游戏
商店功能故障，不影响玩家查看自己的背包，例如炉石传说就是这样。游戏中有很多功能模块，传
统的写法是将多个功能模块包在一个应用程序内，牵一发而动全身，组队功能有一个崩溃点或是内
存泄漏，那就会导致匹配连带着崩溃无法使用，因为这两个模块是写在一个应用内的。而如果我们
在最开始，将功能模块拆分，彼此互不影响，通过 API 进行交互，那就是初步的低耦合。

![服务与服务间低耦合](./images/1689924882137.png)
你可能会认为，单体应用做好错误异常处理就不会有崩溃。理想是好的，但我从来没在任何一个商业游戏项目里看到过完美全覆盖的错误异常处理，工程上怎么保证程序的完美无瑕呢？况且就算应用做到了理想中的完美无瑕，来自操作系统层的偶发性问题导致的崩溃，又该如何规避呢？Linux 系统内核都不能做到完美无瑕不出问题。**要能意识到一切都有可能出错，为了容错而设计架构和编写程序，而不是盲目自信的认为程序不会出错**。


**服务内高内聚**。为了满足低耦合，是不是服务越多越好？并不是，需要满足服务内高内聚的特性。一个服务要精准定位它的用户场景，例如我们写一个组队服务，那这个服务的边界范围就是组队相关的所有功能，创建队伍、成为队长、踢出成员、移交队长、离开队伍等，这些功能聚合在一起形成组队服务，而不能再把这些功能点单独拆除去。为什么？拆的太零散会导致链路过于复杂、性能差、难维护等，失去了微服务的初衷。高内聚就是指，将强相关的所有功能写在一起，这些功能可以描述一个完整的用户场景，也就是可以完全独立进行使用，不依赖其他服务。

其次，游戏为什么微服务？有如下几个优势：

1. 稳定
2. 可独立部署
3. 精确缩放
4. 开发人员友好
5. 迭代速度快

前三点主要是**游戏运维、游戏测试层面考虑**。稳定性主要体现在低耦合之上。服务与服务间的低耦合，保证了 bug 的影响面最低。由于服务与服务之间是低耦合的，所以我们完全可以单独部署其中某一个服务，例如组队，测试部门就可以针对性的进行功能测试和性能测试，而不需要像单体应用那样每次都做整体性的测试。如果我们要测试组队匹配功能，那就需要部署组队+匹配两个服务，还是可以专注在测试的功能点本身之上，如果商店有什么严重 bug 导致完全无法使用，这个 bug 完全不会影响到组队匹配功能的测试与部署，这就是低耦合的好处。

精确缩放是指，我们可以精确的指定每个服务分别使用多少 CPU 多少内存，最大化利用服务器资源。如果这个服务支持多开——指能够水平扩展到任意数量，那我们还可以精确的缩放该服务的数量。即玩家少时开一个实例，玩家多了根据策略动态调整服务数量。

![根据实际用量自动扩缩容](./images/1689925034358.png)
>这里提到了不少点，例如限制服务资源、服务多开、服务单独测试，但并没有展开讲述具体怎么做。这些都是后文中会单独拆开讲述的。

后两点是**游戏开发层面考虑**。如果游戏工作室中只有一个服务器开发，那这两点是否成立存疑。如果项目不小，那我们至少需要一个后端团队来做服务器开发，那这时这两点随着项目功能越加越多，会体现的越来越明显。如果是做一个单体应用，开发者们不得不在同一个应用下干活，代码冲突是常事，几乎每个开发者都必须掌握各功能模块的实现，必须了解模块的启动顺序。这在项目后期维护时尤为痛苦。而如果我们使用微服务，那么每位开发可以单独负责几个服务的开发，开发小唐专门负责组队、匹配这两个服务的开发，需要调用其他服务时只需要看接口文档即可，不需要知道其他服务是谁写的，怎么实现的。同时，每个服务增加/修改/删除一个功能点，要比在一个大的单体项目中做同样的操作简单很多，编译、验证所需的时间明显更低。

>如果你用 C++ 开发过游戏后端的话，应该也饱受编译时间长的折磨，特别是公司没买联合编译的软件。

![开发间的工作相对独立，冲突较少](./images/1689925125351.png)
理论上，完全重写一个核心服务只需要两周时间。当然以我当前项目的实践经验，你可能需要给开发三周到四周，这也远比在一个大的单体应用内，重写一个核心功能模块轻松。事实上，到了项目后期，当需要在一个单体应用内重写一个功能模块是非常痛苦的一件事，不管需要多久时间，验证和测试的过程就会狠狠折磨一个开发——你需要保证写的功能正确的同时，其他功能也都能正常工作！

最后，让我们回顾一下上一章留下的一个尾巴，如何做滚动更新。对于那些支持多开（水平扩展）的服务，我们可以直接应用滚动更新。即更新时保留旧版本不动，新开一个新版本的服务，将网关路由指向新版本的服务，等到旧版本所有请求处理完毕时，再发出关闭信号，等最后扫尾工作做完，结束旧版本的生命周期。

服务如何能做到多开，例如组队服务如何做到多开？那些组队的状态信息如何存储？且待下一大章 《数据库方案演进》 分解。

>下期预告：
4. 数据库方案演进
4.1 通用数据库
4.2 自定义缓存
4.3 文档型数据库
4.4 列式数据库


# 网络游戏架构的前世今生——数据库
来源：https://zhuanlan.zhihu.com/p/559088037

## 4. 数据库方案演进
### 4.1 通用数据库
在开发游戏的最早期阶段，例如游戏原型制作，我们将所有数据直接保存在应用的内存中，随时存、随时取出来用，开发起来高效，程序性能也很高。当我们想要正式开始做一个游戏工程项目，我们不得不考虑一个问题，**游戏数据如何持久化？**

每次我们关闭一个应用，应用所申请的内存地址就被全部释放，也可以说内存中的数据全部丢失。我们必须在这之前保存下来其中的游戏数据——玩家相关的那部分数据，游戏进度等。这就是持久化的需求。

**数据库是持久化的方式之一**。数据库从来都不是开发者的必选项，而是存储数据的方式之一。我们可以将数据存在内存中，可以将数据存在文件中，可以将数据发送到其他应用中代为保存。如果我们需要这个“其他应用”具备一些好用的能力，例如按照特定的数据结构来组织存储，方便查询和修改，那就产生了数据库。数据库是一种存储方式，且这种存储方式是持久长期的。

>在这里我花了一些篇幅，或者说是”废话“，讲述什么是数据库，为什么用数据库。不少开发者使用数据库很熟练，远比我熟练，在怎么用数据库这条路上走得很远，但会忘记为什么要用数据库，陷入思维定式而开发。MVC？框架要求？别人也用？这些都不是原因。

在开发时，我喜欢先用内存做存储，正如上文所说，这样开发起来最快最流畅。等所有接口功能开发完毕，写下不同角度的测试用例，进行完善的功能边界测试。然后再把需要持久化的数据套上数据库接口，选择合适的数据库存储，最后做整体的测试验证。

**最简单的选择，也是最早期的数据库形态，就是关系型数据库**。MySQL、PostgreSQL 和 SQLite 是通常的选择，我们将需要持久化的数据整理成一张张的数据表，规整的存储在数据库中。

![关系型数据库的使用](./images/1689925940172.png)
这些关系型数据库，将数据按照特定的规则存储在文件中，按需读取。在本地测试时性能尚可，但当我们把应用部署好使用一段时间后，会发现关系型数据库的增删改查严重拖慢了我们接口的响应速度——在内存中是纳秒级的响应，而关系型数据库需要近百毫秒。**我们需要加速接口调用速度！**

**仿照其他应用的做法，我们可以使用 memcached、redis 等缓存数据库来缓存这些热数据**。数据库优先读写缓存中的数据，保证性能，然后异步保存到关系型数据库中，保证数据的一致性。

![引入redis做缓冲](./images/1689926026731.png)

**更进一步，我们可以移除背后的关系型数据库，直接把部分数据存储在缓存数据库中**。我们通常把相对临时的数据直接存在缓存数据库中，例如：匹配信息，排队信息。这样做，减少了保存两份数据的操作风险，使得数据操作更加快捷高效。

![redis直接做存储](./images/1689926063566.png)
我们发现，游戏业务中几乎不存在联表查询的需求，表与表之间相对独立。**有些游戏会只用 redis 作为数据库，完全把所有游戏数据存储在缓存中**。好处是简化了技术栈，坏处是，你需要额外考虑缓存数据库的备份、回滚问题，可能会带来额外的技术挑战和运维难度。

## 4.2 自定义缓存
**全盘使用缓存数据库如 redis 作为持久化方案过于激进**，备份与恢复会为游戏项目埋下雷，今后难以调整。更常见的做法是，**使用自定义缓存 buffer 在关系型数据库前挡一层**。当玩家的请求到来时，优先从本地内存中读取或修改数据，数据找不到再去查数据库获取数据。每隔五分钟，或十五分钟完成一次数据同步，从内存中将数据更新的内容写入数据库。

![每个模块管理本地缓存](./images/1689926143949.png)

也有单独拆出来，成为一个独立应用的自定义缓存。不需要仿照 redis 这种公开的缓存数据库设计通用接口，只需要按照项目需要编写特定的数据操作接口，即可把所有数据库操作移入一个完全独立的应用中。

![单独拆出一个独立应用做缓存](./images/1689926180038.png)
这两种方案是类似的，有一个明显的问题是，如果在数据库同步之前应用出现故障，则会丢失这期间内的数据更改，也就是**数据回档**的情况。

**我们可以仿照 MySQL 的做法，打印 redo log 到本地文件中**。只有当 redo log 打印成功，接口才允许返回结果给玩家。可以照搬 MySQL 的实现做出一整套防止崩溃机制，保证异常崩溃后数据不丢失不错乱。

![实现redo log，应对宕机](./images/1689926235957.png)
我们还能找到一些方案，但不管应用怎样的方案，使用自定义缓存的应用是有状态的。状态特性决定了，很难让这些应用模块自由的扩展收缩，想停就停想开就开（这里不考虑为此专门设计一套分布式内存数据库的场景）。**自定义缓存虽然解决了通用数据库的性能问题，但也埋下了无法自由扩展的坑。**

>我接触过的大部分游戏后端都有这样的问题。从传统的眼光来看，这不是大问题。我们可以停服维护，停服做垂直扩展或者水平扩展，大部分游戏都是这样设计的。

>下期预告：
4.3 文档型数据库
4.4 列式数据库


# 网络游戏架构的前世今生——拓展数据库
来源：https://zhuanlan.zhihu.com/p/559839404

## 4.3 文档型数据库
前文提到，大多数游戏开发者会在关系型数据库如 MySQL 和键值型数据库如 Redis 之间做选择，这同样也是其他行业后端开发的通用选择。这些数据库是游戏后端数据库的最佳选择吗？还有没有其他数据库选项，能够更快地响应数据请求，更快地持久化数据呢？

**文档型数据库非常契合游戏开发**。包括 MongoDB、DynamoDB 在内的文档型数据库，越来越受到开发者的欢迎，在不少前沿应用程序中都有他们的身影。当我们封装好一个文档型数据库的调用库，并放入我们游戏项目的组件库中，会发现，开发者们会慢慢把原先存储在 MySQL 或是 Redis 中的数据，逐渐修改为使用文档型数据库进行存储，开发者完全不需要知道背后的原理，只需要知道使用它又快又好用就可以。

**文档型数据库比 MySQL 更快！** 文档数据库作为后起之秀，技术层面上大多支持分布式集群存储，自动处理数据分片，相比于 MySQL 的本地存储，跨机器复制的方式更高效，扩展起来也更加方便。同步调用一次 MySQL 的接口，在正常表规模的情况下需要百毫秒的时间量级，而文档型数据库只需要十毫秒的时间量级，而且响应时间的抖动方差不大。

![MySQL vs MongoDB](./images/1689926495918.png)
**文档型数据库比 Redis 更适合存储对象！** 键值型数据库更适合存储一个个的键值对，而大部分需要保存的游戏数据都是一个个结构体，或者说是对象，这些存储对象的格式是松散的。很多时候我们在一次操作中只想修改一个或几个条目的值，而不想把整个对象都更新一遍。如果存储在键值型数据库中，那只能把这个存储对象通过特定的格式压缩存储，例如 json、bson 或是 protobuf。无论是读取还是修改，都需要整体进行，不断地加解码、转换格式，也不能对其中一个参数进行索引查询。而如果存储在文档型数据库中，则可以部分读取、部分写入，保留了对象的格式，方便程序使用，也节省了网络资源。

>如果考虑到跨进程间的同步问题，使用 Redis 锁的消耗远比在文档型数据库中用乐观锁的消耗大。可以说，在大部分场景下，结构体更适合存储在文档型数据库中。

当然，文档型数据库也有很明显的劣势。**文档型数据库对事务的支持不完全，存在性能上的缺陷**。在流程环节较多时，某一个环节出问题需要整体失败并回滚，传统的关系型数据库更适合这一场景。例如货币、充值等需要强一致性的系统，关系型数据库是更好的选择。

![复杂事物更适合使用关系型数据库](./images/1689926580201.png)
**文档型数据库不支持复杂的跨表级联查询**。幸运的是，游戏后端极少会遇到跨表查询的需求。很多时候需要跨表查询，是初期设计表结构时埋下的坑。所以这一点劣势在游戏后端并不明显。

```
- 不存在下述场景
SELECT account.name, trade.time FROM account,trade 
  WHERE account.id=trade.id;
```

**文档型数据库生态支持欠缺**。这一点才是文档型数据库没能广泛应用的主要原因。开发者对这类型数据库的掌握程度远不如传统数据库，出问题很难解决，这通常是实际项目中最现实的难题。现有的成熟框架，三方库，示例程序几乎都是基于传统数据库设计，如果借鉴甚至原版照抄一个框架引擎进行开发的话，自然不会构思应不应该使用文档型数据库。

如何解决这个问题？我们没能力改变生态环境，但有能力**封装**文档型数据库，以降低数据库的使用门槛，让大部分开发者只需要用封装好的库即可，而不需要学习文档型数据库的技术细节。从本地开发角度，为了开发阶段更好的测试，可以派生出纯内存的存储实现，生产环境再切换为正式的数据库。

```golang
// Storage 存储，提供增删改查能力
type Storage interface {
    // Create 创建一个新的存储对象
    // value 为符合格式的结构体
    Create(value interface{}) error


    // Delete 删除一个存储对象
    // value 为符合格式的结构体
    Delete(value interface{}, hash interface{}, args ...interface{}) error


    // Save 保存一个存储对象
    // value 为符合格式的结构体指针
    Save(value interface{}) error


    // First 获取符合要求的第一个存储对象
    // value 为符合格式的结构体指针
    First(value interface{}, hash interface{}, args ...interface{}) error


    // Find 获取所有符合要求的对象，性能远低于 First，请慎重使用
    // value 为符合格式的结构体切片地址 （注：&[]struct）
    Find(value interface{}, limit int64, expr string, args ...interface{}) error
}


// 利用内存实现一个类，方便开发人员本地调试
// 利用 MongoDB 实现一个类，生产环境应用
// 利用 DynamoDB 实现一个类，生产环境应用
// ……
// 根据配置文件选择当前环境需要使用哪一个持久化存储
```

**文档型数据库非常适合作为游戏数据持久化的默认存储**。先考虑把需要持久化的数据放入文档型数据库中，遇到一些特殊情况，例如需要强一致性保证或事务功能，再考虑用关系型数据库。

有些游戏虽然没有大规模的应用文档型数据库，但会引入文档型数据库解决一些数据存储难题。例如某大逃杀类的游戏，使用 MongoDB 实时存储游戏战斗服的数据，可以做到故障恢复、实时回滚，以及后续助力开挂玩家鉴别。

![MongoDB持久化战斗服对局信息](./images/1689926817248.png)
虽然这类数据库尚未在游戏行业大规模使用，但我相信这是未来的一大趋势，目前缺乏的只是迈过这一步的动力。

## 4.4 列式数据库
上面我们横向对比了在不同场景下，应该选择什么类型的数据库。接下来我们思考一个问题，当数据量持续增大，服务于游戏分析的数据应该存储在什么类型的数据库中？

一开始，我们将部分游戏数据进行分析处理，储存在关系型数据库中，用作每日报表和实时数据展示。数据量小的时候这一切很美妙，我们能很快的完成开发，代码也能很快的分析处理数据，生成报表展示的速度也很快。但随着游戏持续运营，数据表越来越大，我们在分析处理数据上遭遇了瓶颈，查询数据太慢了！

>很多游戏公司直接接入第三方的数据分析服务，没有这个烦恼。如果想要自己搭建一套游戏数据分析服务，出于更好的定制化和节省成本的目的，那下面的内容会有用。

很快我们找到了解决方案，**使用列式数据库存储这些需要分析处理的数据**。列式数据库是以列相关存储架构进行数据存储的数据，区别于 MySQL 这类行式数据库。

例如一张表存储玩家的姓名、年龄、注册时间等信息，如果我们把单个玩家的信息紧紧的放在一个存储块上，那就是行式存储，只要我们知道某个玩家的信息存储在哪个块上，就能检索到他的所有信息。但如果我们想统计一下游戏玩家的年龄分布，那我们需要遍历所有的存储块，获取整张表的信息，这太慢了。

而列式存储，是让每个项目按分区的方式存储在相同的存储块上。如果我们需要统计年龄分布，那就只需要检索年龄的相关存储块，极大地提升了性能效率。

![行与列的区别](./images/1689926896238.png)

根据这一原理，我们将业务不需要访问的数据直接存储到列式数据库中，另一部分后端业务要用的数据，拷贝出来一份存储到列式数据库中，专门用于数据分析。

![应用数据的分类存储](./images/1689926919719.png)
列式数据库非常适合批量数据处理，以及多行的及时查询。我们可以结合使用像 Clickhouse、Hive、HBase 这种分布式开源的数据库，或是采用一些商业数据仓库的解决方案。

>游戏数据分析是一个大话题，这里只指出这一个方向。我接触过的大游戏公司的项目里，有专门的数据分析团队负责数据分析服务的开发和产出。

最后，我们将游戏数据库方案总结为如下这张图，横向为业务逻辑开发选型，纵向为游戏持续运营后数据分析场景下的数据库迁移。

![业务数据库选型与数据分析选型](./images/1689926963560.png)


# 网络游戏架构的前世今生——数据架构
来源：https://zhuanlan.zhihu.com/p/561338038

## 5. 数据架构
数据架构，或者说是数据存储架构，其实包含了上文介绍的数据库方案。之所以先着重介绍了数据库方案，是因为这一块在游戏行业应用广泛，和业务联系紧密，避不开。而本篇中介绍的数据架构内容，大部分中小型游戏公司选择接入第三方平台，或是从略从简的做这些功能。通常意义上，这些不是游戏后端的核心内容，但我认为这些系统同样有价值，我将数据架构整理成下面这张图：

![数据架构](./images/1689927052687.png)
当我们把数据分析处理好存入数据库，运营、决策层和发行商希望部分数据能够实时展示出来，例如当前同时在线玩家、当前同时游戏对局数等，我们需要一个实时的展示系统；此外，他们还希望部分数据能够每天生成为报表，例如次日留存、七日留存、月留存、玩家访问漏斗等数据，我们需要每天制作类似的表格展示这些数据。**这些需求汇总成数据报表展示的需求**。

我们可以**选择一个 BI 展示工具**，例如图中画的 PowerBI 或是 Tableau，通过简单的图表配置，即可将数据库中抽象的数据以图表的形式具象化的展现出来。运营决策能够利用这些数据分析玩家行为，判断玩家更喜欢什么内容、不喜欢什么内容、想要什么内容，从而针对性的对游戏做出调整，以便于提高用户留存，提高游戏的商业价值。

![（图片来源于 PowerBI 官网）](./images/1689927110416.png)

我们的游戏前后端每时每刻都在输出日志信息，例如打印玩家当前在修改存档、匹配对战等。我们会将有用的信息都以日志的信息输出出来，帮助我们判断当前客户端、服务器正在执行什么操作，遇到了什么问题。如果没有这些日志，我们将很难弄清楚玩家遇到的 bug 是怎么产生的。所以**我们需要一套日志分析系统，来统一收集并展示日志**。

图中左侧给出了个人喜欢的 EFK 日志分析体系，早期是 ELK。这套体系的特点是完全免费，而且足够高效好用。

Elasticsearch 作为存储，能将半结构化的数据，例如日志数据，分布式的存储下来，同时支持高效的查询与聚合。

Filebeat 可以通过简单的配置，将输出到文件中，或是打印在标准输出（stdout）下，或是容器内的日志，统一规整好，分门别类的发送到 Elasticsearch 不同的索引（index）内。你可以将逻辑服、战斗服、数据库、客户端的日志分类放入不同的索引，方便查找。Fluentd 和 Filebeat 作用类似，在生产环境的 Kubernetes 集群中，考虑到性能因素，可以把它们替换为 Fluentbit，性能对比 Filebeat 有质的提升，当然相关筛选分类逻辑较弱，大部分场景足够使用，这是一个需要权衡的点。

>在Kubernetes 集群中，其他条件相同的情况下，每个 Filebeat 消耗了 2 GiB 左右的内存，而 Fluentbit 只需要消耗 10 MiB 左右的内存，他们整理好的日志是差不多的！

Kibana 是日志分析体系的可视化平台，当然也可以用作数据分析，它能将 Elasticsearch 存储的日志信息友好的展现出来。你可以通过时间，或者多种条件复合在一起进行筛选，快速定位到出问题时段的日志信息。当然它还支持其他丰富的功能，例如导出 csv，生命周期策略等，在这里就不赘述了。

![图片来源于Kibana官网](./images/1689927170942.png)
**仅仅看日志是不够的，我们还需要实时监控服务器的 CPU、内存、磁盘等指标，建立告警体系**。如果应用了微服务，我们还需要查看每个服务容器的 CPU、GC 等指标。我们可以提前设置异常阈值，例如 CPU 大于 80% 时告警，接入邮件、Slack 或是企业微信等通知工具，让运维人员及时处理。开发人员也能根据当时的各项指标状况，结合日志进行准确的 bug 原因分析，从而更好的修复 bug。

图中画了两套，一套是 Zabbix，这个是流行的老牌监控告警应用。我更喜欢使用另一套开源的方案——Prometheus+Grafana，开源以及天然亲和 Kubernetes 体系，成为了微服务监控告警的首选。使用 Webhook 能接入企业微信或是钉钉，快速通知到运维部门群内，得到快速的响应处理。

![（图片截取自 Grafana 官网）](./images/1689927219426.png)

**配置项也是数据架构的一环**。配置项属于半结构化或是非结构化的数据，在游戏里起到举足轻重的作用。游戏内角色属性、等级、关卡等数据，通常存储为二进制格式的文件，客户端和服务器都需要使用到它们。服务器也需要自己的配置文件，决定当前服务器的环境模式、连接哪个数据库、重试次数超时时间等。如果使用统一部署，例如使用 Kubernetes 容器集群，还需要部署配置文件，决定如何部署、版本信息、对外暴露哪些端口等。

这些配置文件可以存储在对象存储中，或是存储在本地的文件系统内。存储在对象存储的好处是可以在运行时远程查看当前的配置项，同样也可以为修复 bug 、解决玩家问题提供便利。

最后，**如果我们把这些数据系统中的数据整合到一起，那就是一个数据湖（data lake）**。我们可以自己写一个服务，实现所有数据存储的访问，将所有数据整合到一起进行分析。例如，我们可以将某一时段的数据全部整合到一起，发送给开发人员，全面的分析玩家遇到的问题；或是将数据加工后，放入机器学习模型，预测服务器的故障概率、玩家的行为模式等。这是一片广阔的数据舞台，里面的价值不可估量，也是未来的重要发展方向。

>有不少大厂推出了数据湖的第三方服务，在其他行业有不少应用案例。我没在真正的游戏项目中应用过，这里就不妄加阐述推测了。我当前的项目里所有数据架构都是自行搭建的，没有使用第三方平台，后续会尝试实现数据湖整合数据、挖掘数据价值，这是一件想想都觉得兴奋的事。


# 网络游戏架构的前世今生——总结
来源：https://zhuanlan.zhihu.com/p/563814047

## 6. 总结
我们将这一大章介绍的网络、计算、数据三大块内容结合在一起，就成了网络游戏的服务器架构。如果我们再加上图像动画的渲染部分，这部分通常由客户端引擎代劳，那就是完整的游戏架构。

回首再看，作为一个游戏开发者，我试图站在一个旁观者的视角来讲述游戏技术的变迁，但其实有不少内容还是“站了队”，讲了许多自己的见解和判断。在知晓并尊重其他技术实现方式的同时，我有着明确的技术倾向或者说是选择，正是这种选择促使我去淌出一条微服务架构的路。

幸运的是，我不是唯一坚持这条路的人。介绍微服务的那一篇曾讲过，最早萌生这个想法是受到日本多款游戏大作架构的启发，后来在上海自己参与的游戏项目中，也有接触到微服务架构。他们在这条路上比我走的更早，也有着更成熟的案例，但公开出来的经验见解极少，更是没有成熟的程序框架。我只知道这条路一定走的通，但却不知道为何去走？如何去走？如何让一群普通的程序员，开发出完善的微服务游戏服务器，稳定又要高效？

老牌游戏大厂也有过尝试。巨人网络曾在 Github 上放过一个 Golang 的微服务框架 go-service，现在再去访问已经 404。网易放在 Github 上的 pomelo，上一次维护是在三年多以前，纯 js 的框架，只适合运用在一些用量不大、对性能要求不严格的游戏上。TopFreeGames 放在 Github 上的 pitaya，据说是参考了网易的 pomelo 而构建的一个 Golang 微服务框架，至今还在维护，但文档支持方面做的很差，案例也很少，所以不温不火。大厂一定有着稳定高效的后端框架，使用了很多年，微服务对他们还剩多大的吸引力呢？

开源项目也有过尝试。mqant 是让我印象最深刻的游戏微服务框架，可以说我现在实现的框架一部分借鉴了 mqant 的思想。这样的框架有热度，也有一定的案例做支撑，但是还不够完善。或者说是，对于底层开发者而言，使用门槛还是高了些。

![mqant架构](./images/1689927593206.png)
![（图片来源于 mqant 文档）](./images/1689927608906.png)

我想做一套微服务框架，降低使用者的门槛，它能像开发单体应用那样开发，同时又支持分布式的部署。它的架构应该是这样：

![理想的游戏微服务架构](./images/1689927631143.png)
从第三章开始，我会详细阐述如何从零开始，一步步把整个框架搭建起来。最后，再分享几个成熟的游戏架构例子：

![某国内传统游戏架构](./images/1689927652967.png)
![某日本著名微服务游戏架构](./images/1689927674142.png)

>下一篇开始第二章《微服务的价值》，重点解决”为什么”的问题，也回答部分质疑“游戏服务器不能应用微服务”的经典问题。

>第二章将按下述顺序进行展开
1. 微服务真的慢吗
2. 解耦
3. 剥离有状态
4. 敏捷开发
5. 运维管理
6. 完美热更
7. 总结


# 【微服务】RPC、gRPC 和 C/S 架构的基本实现（Golang）
来源：https://blog.csdn.net/weixin_45651194/article/details/129930250

## 一、RPC
### 1.1 what & why need?
>一言以蔽之，RPC 是分布式系统的基石。

RPC(Remote Procedure Call)，中文名为远程过程调用。它最初由 Xerox 公司提出并对其定义为：
“RPC 是一种语言级别的通讯协议，它允许运行于一台计算机上的程序以某种管道作为通讯媒介，去调用另外一个地址空间”。

1. 从**类型**上说，RPC 是一种通讯协议；
2. 从**功能**上说，RPC 实现的功能是在一台机器上调用另一台机器的地址空间，该地址空间可能对应函数、变量等；
3. 从**实现手段**上说，RPC 需要借助计算机网络中的传输层来实现管道通讯。传输层的管道通信可以理解为通过 IP 地址和端口号来确定通信管道的两端。

随着互联网的发展，“客户端—服务器—数据库” 的单体架构（Monolithic Architecture）已无法满足日益复杂的业务逻辑和日渐增多的业务访问，因此需要转向微服务架构（Microservices Architecture）。以下是 Google Cloud Platform 中对微服务的定义：
>微服务架构（通常简称为微服务）是指开发应用所用的一种架构形式。通过微服务，可将大型应用分解成多个独立的组件，其中每个组件都有各自的责任领域。在处理一个用户请求时，基于微服务的应用可能会调用许多内部微服务来共同生成其响应。

![微服务](./images/1694163130217.png)
根据上述定义可知，对于微服务架构中的一个请求，各个服务间需要相互调用才能最终生成对应的响应，因此服务间的调用问题就是成为一个关键问题。而 RPC 正好能解决该问题，所以会有人说 **“要想搞懂微服务，先搞定RPC”**。在微服务架构中加入合适的 RPC 框架，不仅能降低微服务架构的开发成本，也能提高服务间的调用效率。

## 二、gRPC
### 2.1 一个通用开源框架
gRPC 官方文档中文版：开源中国（https://doc.oschina.net/grpc?t=57966 ）。似乎很久没有更新了，最新的英文文档请参考 docs（https://grpc.io/docs/ ）。

gRPC 最开始由 Google 开发，是一款语言中立、平台中立、**开源**的远程过程调用（RPC）系统。gRPC 基于 **HTTP/2** 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多路复用请求等特性。这些特性使得其在**移动设备**上表现更好，更省电和节省空间占用。根据 grpc.io 的数据，目前该框架已支持多种语言，包括 Java、C#、Go、C++ 、Dart、Python、Kotlin、PHP、Ruby、Objective-C 和 Node。

### 2.2 配置 gRPC 环境
* 在 go.mod 中配置 gRPC 核心库（会放在 `$GOPATH/pkg` 中）
```
$ go get google.golang.org/grpc

```

* 下载对应语言的代码生成插件（会放在 `$GOPATH/bin` 中）
```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

```

![bin文件夹](./images/1694163378747.png)

## 三、Golang 实现简单的 RPC 服务
### 3.1 编写 proto 文件并生成对应的 Go 代码

proto 文件的部分语法
1. `service` 服务名 { `rpc` rpc方法名(请求消息) `returns`(响应消息) { } }
2. `message` 消息名 { 变量类型 变量名 = 在消息中的位次; }

编写 server.proto 文件，声明语法为 proto3，`service` 类似于 Go 中的 `func`，`message` 类似于 Go 中的 `struct`。rpc 表示 ShakeHands 是 RPC 方法，接受请求 `ShakeReq`，返回结果 `ShakeRes`。
```protobuf
syntax = "proto3";

option go_package = ".;service"; // 生成的.go文件在哪个目录的哪个包下（用;隔开 目录;包名）

service ShakeHands {
    rpc ShakeHands(ShakeReq) returns (ShakeRes) {}
}

message ShakeReq {
    string requestName = 1; // 这是在定义变量在 message 中的位置
}
message ShakeRes {
    string responseMsg = 1;
}

```

然后进入 proto 文件所在目录，执行 protoc 命令。
```
$ cd path/to/proto/file
$ protoc --go_out=. server.proto
$ protoc --go-grpc_out=. server.proto

```

生成的文件如下，–go_out 表示 `.pb.go` 文件输出的目录位置，–go-grpc_out 表示 `_grpc.pb.go` 文件输出的目录位置。

![文件输出位置](./images/1694163630629.png)

### 3.2 实现服务端代码
主要流程如下：
1. 实现 `_grpc.pb.go` 文件中定义的服务（或叫 RPC 方法）。
2. 开启 TCP 端口。
3. 创建 gRPC 服务并在 gRPC 服务中注册实现了的服务。
4. 启动 gRPC 服务。

```golang
package main

import (
	"context"
	"fmt"
	"net"

	"google.golang.org/grpc"

	pb "gRPC/server/proto"
)

type server struct {
	pb.UnimplementedShakeHandsServer
}

func (s *server) ShakeHands(ctx context.Context, req *pb.ShakeReq) (res *pb.ShakeRes, err error) {
	return &pb.ShakeRes{ResponseMsg: "res: hello!" + req.RequestName}, nil
}

func main() {
	listen, _ := net.Listen("tcp", "127.0.0.1:9999")             // 开启tcp端口
	grpcServer := grpc.NewServer()                     // 创建grpc服务
	pb.RegisterShakeHandsServer(grpcServer, &server{}) // 在grpc服务中注册我们的服务
	err := grpcServer.Serve(listen)                    // 启动服务
	if err != nil {
		fmt.Println("server error!")
		return
	}
}

```

### 3.3 不带安全认证的客户端
客户端的逻辑很简单，就是建立与服务端的连接：

1. 拨号，发起连接。`grpc.Dial()` 服务端的 IP 地址和端口；
2. 通过 `_grpc.pb.go` 文件提供的方法创建客户端实例；
3. 调用 RPC 方法发起请求。

```golang
package main

import (
	"context"
	"fmt"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"

	pb "gRPC/server/proto"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:9999", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		fmt.Println("connection error!")
	}
	defer conn.Close()

	client := pb.NewShakeHandsClient(conn) // 建立连接
	resp, err := client.ShakeHands(context.Background(), &pb.ShakeReq{RequestName: "test"})
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Println(resp.GetResponseMsg())
}


```

### 3.4 进行 SSL/TLS 安全认证



# 详解golang中的context
来源：https://zhuanlan.zhihu.com/p/390970529

golang context 包 一开始只是 Google 内部使用的一个 Golang 包，在 Golang 1.7的版本中正式被引入标准库。

在学习 context 包之前，先看几种日常开发中经常会碰到的业务场景：

1. 业务需要对访问的数据库，RPC ，或API接口，为了防止这些依赖导致我们的服务超时，需要针对性的做超时控制。
2. 为了详细了解服务性能，记录详细的调用链Log。

上面两种场景在web中是比较常见的，context 包就是为了方便我们应对此类场景而使用的。

接下来, 我们首先学习 context 包有哪些方法供我们使用；接着举一些例子，使用 context 包应用在我们上述场景中去解决我们遇到的问题；最后从源码角度学习 context 内部实现，了解 context 的实现原理。

## Context 包
### Context 定义
context 包中实现了多种 Context 对象。Context 是一个接口，用来描述一个程序的上下文。接口中提供了四个抽象的方法，定义如下：
```golang
type Context interface {
  Deadline() (deadline time.Time, ok bool)
  Done() <-chan struct{}
  Err() error
  Value(key interface{}) interface{}
}
```

* Deadline() 返回的是上下文的截至时间，如果没有设定，ok 为 false
* Done() 当执行的上下文被取消后，Done返回的chan就会被close。如果这个上下文不会被取消，返回nil
* Err() 有几种情况:
	* 如果Done() 返回 chan 没有关闭，返回nil
	* 如果Done() 返回的chan 关闭了， Err 返回一个非nil的值，解释为什么会Done()
		* 如果Canceled，返回 "Canceled"
		* 如果超过了 Deadline，返回 "DeadlineEsceeded"
* Value(key) 返回上下文中 key 对应的 value 值

### Context 构造
为了使用 Context，我们需要了解 Context 是怎么构造的。

Context 提供了两个方法做初始化：
```golang
func Background() Context{}
func TODO() Context {}
```

上面方法均会返回空的 Context，但是 Background 一般是所有 Context 的基础，所有 Context 的源头都应该是它。TODO 方法一般用于当传入的方法不确定是哪种类型的 Context 时，为了避免 Context 的参数为nil而初始化的 Context。

其他的 Context 都是基于已经构造好的 Context 来实现的。一个 Context 可以派生多个子 context。基于 Context 派生新Context 的方法如下：
```golang
func WithCancel(parent Context) (ctx Context, cancel CancelFunc){}
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {}
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {}
```

上面三种方法比较类似，均会基于 parent Context 生成一个子 ctx，以及一个 Cancel 方法。如果调用了cancel 方法，ctx 以及基于 ctx 构造的子 context 都会被取消。不同点在于 WithCancel 必需要手动调用 cancel 方法，WithDeadline
可以设置一个时间点，WithTimeout 是设置调用的持续时间，到指定时间后，会调用 cancel 做取消操作。

除了上面的构造方式，还有一类是用来创建传递 traceId， token 等重要数据的 Context。
```golang
func WithValue(parent Context, key, val interface{}) Context {}
```

withValue 会构造一个新的context，新的context 会包含一对 Key-Value 数据，可以通过Context.Value(Key) 获取存在 ctx 中的 Value 值。

通过上面的理解可以直到，Context 是一个树状结构，一个 Context 可以派生出多个不一样的Context。我们大概可以画一个如下的树状图：

![树状图](./images/1694524402877.png)
一个background，衍生出一个带有traceId的valueCtx，然后valueCtx衍生出一个带有cancelCtx
的context。最终在一些db查询，http查询，rpc沙逊等异步调用中体现。如果出现超时，直接把这些异步调用取消，减少消耗的资源，我们也可以在调用时，通过Value 方法拿到traceId，并记录下对应请求的数据。

当然，除了上面的几种 Context 外，我们也可以基于上述的 Context 接口实现新的Context.

## 使用方法
下面我们举几个例子，学习上面讲到的方法。

### 超时查询的例子
在做数据库查询时，需要对数据的查询做超时控制，例如：
```golang
ctx = context.WithTimeout(context.Background(), time.Second)
rows, err := pool.QueryContext(ctx, "select * from products where id = ?", 100)
```

上面的代码基于 Background 派生出一个带有超时取消功能的ctx，传入带有context查询的方法中，如果超过1s未返回结果，则取消本次的查询。使用起来非常方便。为了了解查询内部是如何做到超时取消的，我们看看DB内部是如何使用传入的ctx的。

在查询时，需要先从pool中获取一个db的链接，代码大概如下：
```golang
// src/database/sql/sql.go
// func (db *DB) conn(ctx context.Context, strategy connReuseStrategy) *driverConn, error)

// 阻塞从req中获取链接，如果超时，直接返回
select {
case <-ctx.Done():
  // 获取链接超时了，直接返回错误
  // do something
  return nil, ctx.Err()
case ret, ok := <-req:
  // 拿到链接，校验并返回
  return ret.conn, ret.err
}
```

req 也是一个chan，是等待链接返回的chan，如果Done() 返回的chan 关闭后，则不再关心req的返回了，我们的查询就超时了。

在做SQL Prepare、SQL Query 等操作时，也会有类似方法：
```golang
select {
default:
// 校验是否已经超时，如果超时直接返回
case <-ctx.Done():
  return nil, ctx.Err()
}
// 如果还没有超时，调用驱动做查询
return queryer.Query(query, dargs)
```

上面在做查询时，首先判断是否已经超时了，如果超时，则直接返回错误，否则才进行查询。

可以看出，在派生出的带有超时取消功能的 Context 时，内部方法在做异步操作（比如获取链接，查询等）时会先查看是否已经
Done了，如果Done，说明请求已超时，直接返回错误；否则继续等待，或者做下一步工作。这里也可以看出，要做到超时控制，需要不断判断 Done() 是否已关闭。

### 链路追踪的例子
在做链路追踪时，Context 也是非常重要的。（所谓链路追踪，是说可以追踪某一个请求所依赖的模块，比如db，redis，rpc下游，接口下游等服务，从这些依赖服务中找到请求中的时间消耗）

下面举一个链路追踪的例子：
```golang
// 建议把key 类型不导出，防止被覆盖
type traceIdKey struct{}{}

// 定义固定的Key
var TraceIdKey = traceIdKey{}

func ServeHTTP(w http.ResponseWriter, req *http.Request){
  // 首先从请求中拿到traceId
  // 可以把traceId 放在header里，也可以放在body中
  // 还可以自己建立一个 （如果自己是请求源头的话）
  traceId := getTraceIdFromRequest(req)

  // Key 存入 ctx 中
  ctx := context.WithValue(req.Context(), TraceIdKey, traceId)

  // 设置接口1s 超时
  ctx = context.WithTimeout(ctx, time.Second)

  // query RPC 时可以携带 traceId
  repResp := RequestRPC(ctx, ...)

  // query DB 时可以携带 traceId
  dbResp := RequestDB(ctx, ...)

  // ...
}

func RequestRPC(ctx context.Context, ...) interface{} {
    // 获取traceid，在调用rpc时记录日志
    traceId, _ := ctx.Value(TraceIdKey)
    // request

    // do log
    return
}
```

上述代码中，当拿到请求后，我们通过req 获取traceId， 并记录在ctx中，在调用RPC，DB等时，传入我们构造的ctx，在后续代码中，我们可以通过ctx拿到我们存入的traceId，使用traceId 记录请求的日志，方便后续做问题定位。

当然，一般情况下，context 不会单纯的仅仅是用于 traceId 的记录，或者超时的控制。很有可能二者兼有之。

## 如何实现
知其然也需知其所以然。想要充分利用好 Context，我们还需要学习 Context 的实现。下面我们一起学习不同的 Context 是如何实现 Context 接口的，

### 空上下文
Background(), Empty() 均会返回一个空的 Context emptyCtx。emptyCtx 对象在方法 Deadline(), Done(), Err(), Value(interface{}) 中均会返回nil，String() 方法会返回对应的字符串。这个实现比较简单，我们这里暂时不讨论。

### 有取消功能的上下文
WithCancel 构造的context 是一个cancelCtx实例，代码如下：
```golang
type cancelCtx struct {
  Context

  // 互斥锁，保证context协程安全
  mu       sync.Mutex
  // cancel 的时候，close 这个chan
  done     chan struct{}
  // 派生的context
  children map[canceler]struct{}
  err      error
}
```

WithCancel 方法首先会基于 parent 构建一个新的 Context，代码如下：
```golang
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
  c := newCancelCtx(parent)  // 新的上下文
  propagateCancel(parent, &c) // 挂到parent 上
  return &c, func() { c.cancel(true, Canceled) }
}
```

其中，propagateCancel 方法会判断 parent 是否已经取消，如果取消，则直接调用方法取消；如果没有取消，会在parent的children 追加一个child。这里就可以看出，context 树状结构的实现。 下面是propateCancel 的实现：
```golang
// 把child 挂在到parent 下
func propagateCancel(parent Context, child canceler) {
  // 如果parent 为空，则直接返回
  if parent.Done() == nil {
    return // parent is never canceled
  }
  
  // 获取parent类型
  if p, ok := parentCancelCtx(parent); ok {
    p.mu.Lock()
    if p.err != nil {
      // parent has already been canceled
      child.cancel(false, p.err)
    } else {
      if p.children == nil {
        p.children = make(map[canceler]struct{})
      }
      p.children[child] = struct{}{}
    }
    p.mu.Unlock()
  } else {
    // 启动goroutine，等待parent/child Done
    go func() {
      select {
      case <-parent.Done():
        child.cancel(false, parent.Err())
      case <-child.Done():
      }
    }()
  }
}
```

Done() 实现比较简单，就是返回一个chan，等待chan 关闭。可以看出 Done 操作是在调用时才会构造 chan done，done 变量是延时初始化的。
```golang
func (c *cancelCtx) Done() <-chan struct{} {
  c.mu.Lock()
  if c.done == nil {
    c.done = make(chan struct{})
  }
  d := c.done
  c.mu.Unlock()
  return d
}
```

在手动取消 Context 时，会调用 cancelCtx 的 cancel 方法，代码如下：
```golang
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
  // 一些判断,关闭 ctx.done chan
  // ...
  if c.done == nil {
    c.done = closedchan
  } else {
    close(c.done)
  }

  // 广播到所有的child，需要cancel goroutine 了
  for child := range c.children {
    // NOTE: acquiring the child's lock while holding parent's lock.
    child.cancel(false, err)
  }
  c.children = nil
  c.mu.Unlock()

  // 然后从父context 中，删除当前的context
  if removeFromParent {
    removeChild(c.Context, c)
  }
}
```

这里可以看到，当执行cancel时，除了会关闭当前的cancel外，还做了两件事，① 所有的child 都调用cancel方法，② 由于该上下文已经关闭，需要从父上下文中移除当前的上下文。

### 定时取消功能的上下文
WithDeadline, WithTimeout 提供了实现定时功能的 Context 方法，返回一个timerCtx结构体。WithDeadline 是给定了执行截至时间，WithTimeout 是倒计时时间，WithTImeout 是基于WithDeadline实现的，因此我们仅看其中的WithDeadline
即可。WithDeadline 内部实现是基于cancelCtx 的。相对于 cancelCtx 增加了一个计时器，并记录了 Deadline 时间点。下面是timerCtx 结构体：

```golang
type timerCtx struct {
  cancelCtx
  // 计时器
  timer *time.Timer
  // 截止时间
  deadline time.Time
}
```

WithDeadline 的实现：
```golang
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
  // 若父上下文结束时间早于child，
  // 则child直接挂载在parent上下文下即可
  if cur, ok := parent.Deadline(); ok && cur.Before(d) {
    return WithCancel(parent)
  }

  // 创建个timerCtx, 设置deadline
  c := &timerCtx{
    cancelCtx: newCancelCtx(parent),
    deadline:  d,
  }

  // 将context挂在parent 之下
  propagateCancel(parent, c)

  // 计算倒计时时间
  dur := time.Until(d)
  if dur <= 0 {
    c.cancel(true, DeadlineExceeded) // deadline has already passed
    return c, func() { c.cancel(false, Canceled) }
  }
  c.mu.Lock()
  defer c.mu.Unlock()
  if c.err == nil {
    // 设定一个计时器，到时调用cancel
    c.timer = time.AfterFunc(dur, func() {
      c.cancel(true, DeadlineExceeded)
    })
  }
  return c, func() { c.cancel(true, Canceled) }
}
```

构造方法中，将新的context 挂在到parent下，并创建了倒计时器定期触发cancel。

timerCtx 的cancel 操作，和cancelCtx 的cancel 操作是非常类似的。在cancelCtx 的基础上，做了关闭定时器的操作
```golang
func (c *timerCtx) cancel(removeFromParent bool, err error) {
  // 调用cancelCtx 的cancel 方法 关闭chan，并通知子context。
  c.cancelCtx.cancel(false, err)
  // 从parent 中移除
  if removeFromParent {
    removeChild(c.cancelCtx.Context, c)
  }
  c.mu.Lock()
  // 关掉定时器
  if c.timer != nil {
    c.timer.Stop()
    c.timer = nil
  }
  c.mu.Unlock()
}
```

timeCtx 的 Done 操作直接复用了cancelCtx 的 Done 操作，直接关闭 chan done 成员。

### 传递值的上下文
WithValue 构造的上下文与上面几种有区别，其构造的context 原型如下：
```golang
type valueCtx struct {
  // 保留了父节点的context
  Context
  key, val interface{}
}
```

每个context 包含了一个Key-Value组合。valueCtx 保留了父节点的Context，但没有像cancelCtx 一样保留子节点的Context. 下面是valueCtx的构造方法：
```golang
func WithValue(parent Context, key, val interface{}) Context {
  if key == nil {
    panic("nil key")
  }
  // key 必须是课比较的，不然无法获取Value
  if !reflect.TypeOf(key).Comparable() {
    panic("key is not comparable")
  }
  return &valueCtx{parent, key, val}
}
```

直接将Key-Value赋值给struct 即可完成构造。下面是获取Value 的方法：
```golang
func (c *valueCtx) Value(key interface{}) interface{} {
  if c.key == key {
    return c.val
  }
  // 从父context 中获取
  return c.Context.Value(key)
}
```

Value 的获取是采用链式获取的方法。如果当前 Context 中找不到，则从父Context中获取。如果我们希望一个context 多放几条数据时，可以保存一个map 数据到 context 中。这里不建议多次构造context来存放数据。毕竟取数据的成本也是比较高的。

## 注意事项
最后，在使用中应该注意如下几点：

* context.Background 用在请求进来的时候，所有其他context 来源于它。
* 在传入的conttext 不确定使用的是那种类型的时候，传入TODO context （不应该传入一个nil 的context)
* context.Value 不应该传入可选的参数，应该是每个请求都一定会自带的一些数据。（比如说traceId，授权token 之类的）。在Value 使用时，建议把Key 定义为全局const 变量，并且key 的类型不可导出，防止数据存在冲突。
* context goroutines 安全。


# javapretty / game_server
来源：https://github.com/javapretty/game_server

通用游戏服务器框架，支持多进程、多线程部署，支持TCP、WebSocket、Http协议

服务器是异步非阻塞的多进程多线程架构，运行于CentOS7操作系统。服务器底层使用C++ 编写，高效稳定， 使用TCP协议进行进程间以及与客户端的通信。同时也支持UDP、Websocket、Http协议。 网络通信协议使用自定义格式，具备加密功能，安全高效。游戏数据库支持MongoDB和Mysql，可以选择自己想要的数据库。 逻辑层使用简单高效的Javascript进行开发，集成Google V8引擎解析脚本，保证脚本运行效率，有js经验的开发人员可以快速上手。

服务器分为LoginServer，GateServer，GameServer，MasterServer，DBServer，LogServer。

LoginServer：客户端发送账号密码到LoginServer，LoginServer去AccountDb验证， 验证失败断开连接，验证成功，则生成session， 根据玩家账号hash和gate在线人数选择GateIp,GatePort，发送给客户端。

GateServer：接受客户端发过来的连接，向LoginServer进行session验证，验证成功玩家可以正常登录， 然后就开始接收客户端消息，转发到Game，Master，类似网关中转器。

GameServer： 1、创建角色，角色登录，从DbServer加载数据返回给client，存储玩家数据
2、场景管理模块 3、AOI模块 4、定时器管理模块 5、怪物AI模块 6、Player管理模块 7、C++ 消息解析包装成js对象

MasterServer： 1、管理GameServer，玩家在GameServer进程间跳转 2、存储在线玩家数据，进行相关校验验证，存储每个玩家对应的gameserver 3、存储公共数据，例如帮派，好友，邮件，聊天，组队

V8：V8引擎管理模块，js代码解析，执行js代码，C++ 函数导出

DbServer： 1、建立mongodb引擎和存取数据接口 2、建立Mysql引擎和存取数据接口

LogServer: 1、实现写服务器文件日志功能 2、实现写玩家操作日志到Mysql Log数据库


# 13张图搞懂分布式系统服务注册与发现原理
来源：https://cloud.tencent.com/developer/article/1778134

在微服务架构或分布式环境下，服务注册与发现技术不可或缺，这也是程序员进阶之路必须要掌握的核心技术之一，本文通过图解的方式带领大家轻轻松松掌握。
引入服务注册与发现组件的原因

先来看一个问题，假如现在我们要做一个商城项目，作为架构师的你应该怎样设计系统的架构？你心里肯定在想：这还不容易直接照搬淘宝的架构不就行了。但在现实的创业环境中一个项目可能是九死一生，如果一开始投入巨大的人力和财力，一旦项目失败损失就很大。

作为一位有经验的架构师需要结合公司财力、人力投入预算等现状选择最适合眼下的架构才是王道。大型网站都是从小型网站发展而来，架构也是一样。

任何一个大型网站的架构都不是从一开始就一层不变的，而是随着用户量和数据量的不断增加不断迭代演进的结果。

在架构不断迭代演进的过程中我们会遇到很多问题，技术发展的本质就是不断发现问题再解决问题，解决问题又发现问题。

## 单体架构-应用和数据库共同部署
在系统建立之初可能不会有特别多的用户，将所有的业务打成一个应用包放在tomcat容器中运行，与数据库共用一台服务器，这种架构一般称之为单体架构。

![单体架构-应用和数据库共同部署](./images/1695441770719.png)
在初期这种架构的效率非常高，根据用户的反馈可以快速迭代上线。但是随着用户量增加，一台服务的内存和CPU吃紧，很容易造成瓶颈，新的问题来了怎么解决呢？

## 单体架构-应用与数据分离
随着用户请求量增加，一台服务器的内存和CPU持续飙升，用户请求响应时间变慢。这时候可以考虑将应用与数据库拆开，各自使用一台服务器，你看问题又解决了吧。

![单体架构-应用和数据库分离](./images/1695441838134.png)

突然有一天扫地阿姨不小心碰了电线，其中一台服务器掉电了，用户所有的请求都报错，随之而来的是一系列投诉电话。


## 集群部署
单实例很容易造成单点问题，比如遇到服务器故障或者服务能力瓶颈，那怎么办？聪明的你肯定想到了，用集群呀。

![应用集群部署](./images/1695441933451.png)

集群部署是指将应用部署在多个服务器或者虚机上，用户通过服务均衡随机访问其中的一个实例，从而使多个实例的流量均衡，如果一个实例出现故障可以将其下线，其他实例不受影响仍然可以对外提供服务。

随着用户数量快速增加，老板决定增加投入扩大团队规模。开发团队壮大后效率并没有得到显著的提高，以前小团队可以一周迭代上线一次，现在至少需要两到三周时间。

业务逻辑越来越复杂，代码间耦合很严重，修改一行代码可能引入几个线上问题。架构师意识到需要进行架构重构。


## 微服务架构
当单体架构演进到一定阶段后开发测试的复杂性都会成本增加，团队规模的扩大也会使得各自工作耦合性更严重，牵一发而动全身就是这种场景。

单体架构遇到瓶颈了，微服务架构就横空出世了。微服务就是将之前的单体服务按照业务维度进行拆分，拆分粒度可大可小，拆分时机可以分节奏进行。最佳实践是先将一些独立的功能从单体中剥离出来抽成一个或多个微服务，这样可以保障业务的连续性和稳定性。

![微服务架构](./images/1695442008961.png)
如上图将一个商用应用拆分为六个独立微服务。六个微服务可以使用Docker容器化进行多实例部署。

架构演化到这里遇到了一个难题，如果要查询用户所有的订单，用户服务可能会依赖订单服务，用户服务如何与订单服务交互呢？订单服务有多个实例该访问哪一个？

通常有几种解决办法：
### （1）服务地址硬编码
服务的地址写死在数据库或者配置文件，通过访问DNS域名进行寻址路由。

![服务元数据硬编码](./images/1695442100760.png)

服务B的地址硬编码在数据库或者配置文件中，服务A首先需要拿到服务B的地址，然后通过DNS服务器解析获取其中一实例的真实地址，最后可以向服务B发起请求。

如果遇到大促活动需要对服务实例扩容，大促完需要对服务实例进行下线，运维人员要做大量的手工操作，非常容易误操作。

### （2）服务动态注册与发现
服务地址硬编码还有一个非常致命的问题，如果一台实例挂了，运维人员可能不能及时感知到，导致一部分用户的请求会异常。

引入服务注册与发现组件可以很好解决上面遇到的问题，避免过多的人工操作。

## 架构演进总结
在单体架构中一个应用程序就是一个服务包，包内的模块通过函数方法相互调用，模型足够简单，根本没有服务注册和发现一说。

在微服务架构中会将一个应用程序拆分为多个微服务，微服务会部署在不同的服务器、不同的容器、甚至多数据中心，微服务间要相互调用，服务注册和发现成为了一个不可或缺的组件。


## 服务注册与发现基本原理
服务注册与发现是分为注册和发现两个关键的步骤。

**服务注册**：服务进程在注册中心注册自己的元数据信息。通常包括主机和端口号，有时还有身份验证信息，协议，版本号，以及运行环境的信息。

**服务发现**：客户端服务进程向注册中心发起查询，来获取服务的信息。服务发现的一个重要作用就是提供给客户端一个可用的服务列表。

### 服务注册
服务注册有两种形式：客户端注册和代理注册。

#### （1）客户端注册
客户端注册是服务自己要负责注册与注销的工作。当服务启动后注册线程向注册中心注册，当服务下线时注销自己。

![客户端注册](./images/1695442302031.png)
这种方式的缺点是注册注销逻辑与服务的业务逻辑耦合在一起，如果服务使用不同语言开发，那需要适配多套服务注册逻辑。

#### （2）代理注册
代理注册由一个单独的代理服务负责注册与注销。当服务提供者启动后以某种方式通知代理服务，然后代理服务负责向注册中心发起注册工作。

![代理注册](./images/1695442361116.png)
这种方式的缺点是多引用了一个代理服务，并且代理服务要保持高可用状态。


### 服务发现
服务发现也分为客户端发现和代理发现。

#### （1）客户端发现
客户端发现是指客户端负责向注册中心查询可用服务地址，获取到所有的可用实例地址列表后客户端根据==负载均衡==算法选择一个实例发起请求调用。

![客户端发现](./images/1695442456643.png)
这种方式非常直接，客户端可以控制负载均衡算法。但是缺点也很明显，获取实例地址、负载均衡等逻辑与服务的业务逻辑耦合在一起，如果服务发现或者负载平衡有变化，那么所有的服务都要修改重新上线。

#### （2）代理发现
代理发现是指新增一个路由服务负责服务发现获取可用的实例列表，服务消费者如果需要调用服务A的一个实例可以直接将请求发往路由服务，路由服务根据配置好的负载均衡算法从可用的实例列表中选择一个实例将请求转发过去即可，如果发现实例不可用，路由服务还可以自行重试，服务消费者完全不用感知。

![代理路由服务注册](./images/1695442495564.png)

### 心跳机制
如果服务有多个实例，其中一个实例出现宕机，注册中心是可以实时感知到，并且将该实例信息从列表中移出，也称为摘机。

如何实现摘机？业界比较常用的方式是通过心跳检测的方式实现，心跳检测有**主动**和**被动**两种方式。

#### （1）心跳机制-被动检测
**被动检测**是指服务主动向注册中心发送心跳消息，时间间隔可自定义，比如配置5秒发送一次，注册中心如果在三个周期内比如说15秒内没有收到实例的心跳消息，就会将该实例从列表中移除。

![心跳机制-被动检测](./images/1695442651294.png)
上图中服务A的实例2已经宕机不能主动给注册中心发送心跳消息，15秒之后注册就会将实例2移除掉。

#### （2）心跳机制-主动检测
主动检测是注册中心主动发起，每隔几秒中会给所有列表中的服务实例发送心跳检测消息，如果多个周期内未发送成功或未收到回复就会主动移除该实例。

![心跳机制-主动检测](./images/1695442732706.png)

## 业界常用的服务注册与发现组件对比
了解服务注册与发现的基本原理后，如果你要在项目中使用服务注册与发现组件，当面对众多的开源组件该如何进行技术选型？

在互联网公司里，有研发实力的大公司一般会选择自研或者基于开源组件进行二次开发，但是对于中小型公司来说直接选用一款开源软件会是一个不错的选择。

常用的注册与发现组件有eureka，zookeeper，consul，etcd等，由于eureka在2018年已经宣布放弃维护，这里就不再推荐使用了。

![业界开源组件](./images/1695442765978.png)
下面结合各个维度对比一下各组件。

| 组件      | 优点                                                                                                                 | 缺点                                                                      | 接口类型    | 一致性算法 |
| --------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ----------- | ---------- |
| zookeeper | 1.功能强大，不仅仅只是服务发现；2.提供watcher机制可以实时获取服务提供者的状态；3.广泛使用，dubbo等微服务框架已支持； | 1.没有健康检查；2.需要在服务中引入sdk，集成复杂度高；3.不支持多数据中心； | sdk         | Paxos      |
| consul    | 1.开箱即用，方便集成；2.带健康检查；3.支持多数据中心；4.提供web管理界面；                                            | 不能实时获取服务变换通知                                                  | restful/dns | Raft       |
| etcd      | 1.开箱即用，方便集成；2.可配置性强                                                                                   | 1.没有健康检查；2.需配合三方工具完成服务发现功能；3.不支持多数据中心；    | restful     | Raft       |
从整体上看consul的功能更加完备和均衡。接下来以consul为例详细介绍一下。

### Consul——值得推荐的服务注册与发现开源组件
#### 简单认识一下Consul
Consul是HashiCorp公司推出的开源工，使用Go语言开发，具有开箱即可部署方便的特点。Consul是分布式的、高可用的、 可横向扩展的用于实现分布式系统的服务发现与配置。

#### Consul有哪些优势？
* 服务注册发现：Consul提供了通过DNS或者restful接口的方式来注册服务和发现服务。服务可根据实际情况自行选择。
* 健康检查：Consul的Client可以提供任意数量的健康检查，既可以与给定的服务相关联，也可以与本地节点相关联。
* 多数据中心：Consul支持多数据中心，这意味着用户不需要担心Consul自身的高可用性问题以及多数据中心带来的扩展接入等问题。

#### Consul的架构图

![Consul架构图](./images/1695443083377.png)
Consul 实现多数据中心依赖于gossip protocol协议。这样做的目的：

* 不需要使用服务器的地址来配置客户端；服务发现是自动完成的。
* 健康检查故障的工作不是放在服务器上，而是分布式的。

#### Consul的使用场景
Consul的应用场景包括服务注册发现、服务隔离、服务配置等。

* **服务注册发现**场景中consul作为注册中心，服务地址被注册到consul中以后，可以使用consul提供的dns、http接口查询，consul支持health check。

* **服务隔离**场景中consul支持以服务为单位设置访问策略，能同时支持经典的平台和新兴的平台，支持tls证书分发，service-to-service加密。

* **服务配置**场景中consul提供key-value数据存储功能，并且能将变动迅速地通知出去，借助Consul可以实现配置共享，需要读取配置的服务可以从Consul中读取到准确的配置信息。



# 答疑 | 分布式和微服务的区别？
来源：https://zhuanlan.zhihu.com/p/138645236

## 1. 分布式和微服务有什么区别呢？
答：分布式的核心就一个字：拆。只要是将一个项目拆分成了多个模块，并将这些模块分开部署，那就算是分布式。

如何拆呢？有两种方式：水平拆分，或垂直拆分（也称为“横向拆分”和“垂直拆分”），具体如下：

**水平拆分**：根据“分层”的思想进行拆分。例如，可以将一个项目根据“三层架构”拆分成 表示层（jsp+servlet）、业务逻辑层（service）和数据访问层（dao），然后再分开部署：把表示层部署在服务器A上，把service和dao层部署在服务器B上，然后服务器A和服务器B之间通过dubbo等RPC进行进行整合（在左下角的“阅读原文”里有dubbo的视频课程，可以点击学习），如图所示。

![水平拆分](./images/1695452066846.png)

**垂直拆分**：根据业务进行拆分。例如，可以根据业务逻辑，将“电商项目”拆分成“订单项目”、“用户项目”和“秒杀项目”。显然这三个拆分后的项目，仍然可以作为独立的项目使用。像这种拆分的方法，就成为垂直拆分。

![垂直拆分](./images/1695452102441.png)

### 什么是微服务呢？
从名字就能知道，“微服务”就是非常微小的服务。

微服务可以理解为一种非常细粒度的垂直拆分。例如，以上“订单项目”本来就是垂直拆分后的子项目，但实际上“订单项目”还能进一步拆分为“购物项目”、“结算项目”和“售后项目”，如图。

![三个独立的微服务](./images/1695452212101.png)

现在看图中的“订单项目”，它完全可以作为一个分布式项目的组成元素，但就不适合作为微服务的组成元素了（因为它还能再拆，而微服务应该是不能再拆的“微小”服务，类似于“原子性”）。

## 总结

分布式：拆了就行。

微服务：细粒度的垂直拆分。



# 搞懂“负载均衡”，一篇就够了
来源：https://zhuanlan.zhihu.com/p/366204416

在互联网发展呢的今天，我们一般会把多台机器组成一个集群对外提供服务。然而，我们的网站对外提供的访问入口都是一个的，比如 http://www.baidu.com 那么当用户在浏览器输入 http://www.baidu.com 的时候如何将用户的请求分发到集群中不同的机器上呢，这就是负载均衡在做的事情。

![enter description here](./images/1695720645722.png)

## 负载均衡的分类
负载均衡就是一种计算机网络技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁碟驱动器或其他资源中分配负载，以达到最佳化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。那么，这种计算机技术的实现方式有多种。大致可以分为以下几种，其中最常用的是四层和七层负载均衡：

![enter description here](./images/1695720691854.png)
* 四层负载均衡四层负载均衡工作在OSI模型的传输层，由于在传输层，只有TCP/UDP协议，这两种协议中除了包含源IP、目标IP以外，还包含源端口号及目的端口号。四层负载均衡服务器在接受到客户端请求后，以后通过修改数据包的地址信息（IP+端口号）将流量转发到应用服务器。
* 七层负载均衡七层负载均衡工作在OSI模型的应用层，应用层协议较多，常用http、radius、dns等。七层负载就可以基于这些协议来负载。这些应用层协议中会包含很多有意义的内容。比如同一个Web服务器的负载均衡，除了根据IP加端口进行负载外，还可根据七层的URL、浏览器类别、语言来决定是否要进行负载均衡。

## 常见负载均衡算法
### 轮询(Round Robin)
请求到达后，将客户端发送到负载均衡器的请求依次轮流地转发给服务集群的某个节点。

优点：实现简单，每个集群节点平均分担所有的请求。

缺点：当集群中服务器硬件配置不同、性能差别大时，无法区别对待。引出下面的算法。

### 随机(Random)
随机选取集群中的某个节点来处理该请求，由概率论的知识可知，随着请求量的变大，随机算法会逐渐演变为轮询算法，即集群各个节点会处理差不多数量的请求。

优点：简单使用，不需要额外的配置和算法。

缺点：随机数的特点是在数据量大到一定量时才能保证均衡，所以如果请求量有限的话，可能会达不到均衡负载的要求。

### 加权
加权算法主要是根据集群的节点对应机器的性能的差异，给每个节点设置一个权重值，其中性能好的机器节点设置一个较大的权重值，而性能差的机器节点则设置一个较小的权重值。权重大的节点能够被更多的选中。它是和随机、轮训一起使用的。

优点：可以根据机器的具体情况，分配不同的负载，达到能者多劳。

缺点：需要额外管理加权系数。

### 最小连接数
主要是根据集群的每个节点的当前连接数来决定将请求转发给哪个节点，即每次都将请求转发给当前存在最少并发连接的节点。

优点：可以根据集群节点的负载情况来进行请求的动态分发，即机器性能好，处理请求快，积压请求少的节点分配更多的请求。避免某个节点因为处理超过自身所能承受的请求量而导致宕机或者响应过慢。

### hash
将对请求的IP地址或者URL计算一个哈希值，然后与集群节点的数量进行取模来决定将请求分发给哪个集群节点。它不是真正意义上的负载均衡，在某些意义上也是一个单点服务。

优点：实现简单

缺点：如果某个节点挂了，会使得一部分流量不可用。


## 负载均衡分类
### 1. 硬件负载均衡
常见的硬件有比较昂贵的F5和Array等商用的负载均衡器，它的优点简单，有专业的人负责；缺点就是 贵 如果你是土豪，可以考虑，但是对于规模较小的网络服务来说暂时还没有需要使用。

### 2. 软件负载均衡
目前使用最广泛的三种负载均衡软件Nginx/LVS/HAProxy，他们都是开源免费的负载均衡软件，这些都是通过软件级别来实现，所以费用较低。足以见得，搞软件的是多么不值钱！

### 3. 成熟的架构
负载均衡业界早已有成熟的架构，比较常用的有LVS+Keepalived、Nginx+Keepalived、HAProxy+Keepalived。


## 常见负载均衡软件
### Nginx

![enter description here](./images/1695720999619.png)
Nginx的负载均衡支持：
>rr：轮叫，轮流分配到后端服务器；
wrr：权重轮叫，根据后端服务器负载情况来分配；
lc：最小连接，分配已建立连接最少的服务器上；
wlc：权重最小连接，根据后端服务器处理能力来分配。

Nginx优点：
>1.简单：安装和配置比较简单、测试也简单
2.稳定：单机一般能支撑几万次的并发量
3.轻量：能ping通就就能进行负载功能
4.易用：明确的错误码、超时提醒
5.强大：负载均衡、反向代理、WEB容器等功能

Nginx缺点：
>1.仅能支持http、https和Email协议
2.对后端服务器的健康检查， 只支持通过端口来检测，不支持通过url来检测

>roundrobin：轮询，轮流分配到后端服务器； 
static-rr：根据后端服务器性能分配； 
leastconn：最小连接者优先处理； 
source：根据请求源IP，与Nginx的IP_Hash类似。

### LVS
LVS的负载均衡支持：
>roundrobin：轮询，轮流分配到后端服务器；
static-rr：根据后端服务器性能分配；
leastconn：最小连接者优先处理；
source：根据请求源IP，与Nginx的IP_Hash类似。

LVS优点：
>1.高效：工作在网络4层之上 仅作分发之用，没有流量的产生，这个特点也决定了它在负载均衡软件里的性能最强的 ，对内存和cpu资源消耗比较低
2.易用：配置性比较低，简化操作成本
3.稳定：本身抗负载能力很强，自身有完整的双机热备方案
4.应用广：因为LVS工作在4层，所以它几乎可以对所有应用做负载均衡，包括http、tcp、数据库、在线聊天室等

LVS缺点：
>1.不能做动静分离
2.大型网站LVS+Keepalived实施起来就比较复杂，配置成本高

### HAProxy
HAProxy负载均衡策略非常多，包括：

>roundrobin，表示简单的轮询，这个不多说，这个是负载均衡基本都具备的；
static-rr，表示根据权重，建议关注；
leastconn，表示最少连接者先处理，建议关注；
source，表示根据请求源IP，这个跟Nginx的IP_hash机制类似，我们用其作为解决session问题的一种方法，建议关注；
ri，表示根据请求的URI；
rl_param，表示根据请求的URl参数’balance url_param’ requires an URL parameter name；
hdr(name)，表示根据HTTP请求头来锁定每一次HTTP请求；
rdp-cookie(name)，表示根据据cookie(name)来锁定并哈希每一次TCP请求。

HAProxy的优点能够补充Nginx的一些缺点， 比如支持Session的保持，Cookie的引导；同时支持通过获取指定的url来检测后端服务器的状态。

HAProxy跟LVS类似，本身就只是一款负载均衡软件；单纯从效率上来讲HAProxy会比Nginx有更出色的负载均衡速度，在并发处理上也是优于Nginx的。

支持TCP协议的负载均衡转发，可以对MySQL读进行负载均衡，对后端的MySQL节点进行检测和负载均衡。


## 常见微服务框架的负载均衡
### Dubbo
支持4种算法（随机、轮询、活跃度、Hash一致性），而且算法里面引入权重的概念。

### SpringCloud Feign
支持很多种算法，包括：轮询、随机、最小连接、区域加权、重试以及ResponseTime加权。也可以自己实现负载均衡算法