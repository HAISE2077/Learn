---
title: Unity taikr
---

# 泰课在线[唐老狮]

# 一、Unity入门
## 第3章 Unity界面基础
### Scene场景和Hierarchy层级窗口
### Game游戏和Project工程
### Inspector检查和Console控制台
### 工具栏和父子关系
#### 工具栏
* File 重要选项：
BuildSetting（工程发布打包）

* Edit 重要选项：
Project Setting（工程各系统设置）
Preference（首选项，可以设置编程软件）

* GameObject 重要选项：
MoveToView、Align With View、Align View to Selected（几种快捷设置位置的功能）

#### 对象间的父子关系
1. 子对象会随着父对象的变化而变化
2. 子对象的Transform信息是相对于父对象的
3. Scene窗口上方的Pivot、Global空间的作用
	1. Pivot：相对于自己的中心点 Center：相对于父对象和子对象之间的中心点
	2. Global：相对于世界的坐标系 Local：相对于自己的坐标系


## 第4章 Unity工作原理
### 反射机制和游戏场景
#### 反射机制的体现
除了Transform这个表示位置的脚本外，还可以为GameObject关联各种C#脚本，让它按照代码逻辑处理事情。**而为`GameObject`添加C#脚本的这个过程，就是在利用反射 `new` 一个新的C#脚本对象 和 `GameObject`对象进行关联。**

#### 游戏场景的本质
游戏场景文件，后缀为`.unity`
本质就是一个配置文件
Unity有一套自己识别处理它的机制，但本质就是把场景对象相关信息读取出来，通过反射来创建各个对象关联各个脚本对象

### 预设体和资源包的导入导出
#### 预设体的本质
预设体文件，后缀为`.prefab`
本质就是一个配置文件

#### 资源包的导入导出
Asset文件夹下的资源都可以导入到资源包

## 第5章 Unity脚本基础
### 设置编程用工具
### 基本规则
#### 1.创建的规则
* 类名和文件名必须一致，否则不能挂载（因为反射机制创建对象，会通过文件名去找Type）
* 没有特殊需求就不同管命名空间
* 创建的脚本默认继承MonBehavior

#### 2.MonoBehavior基类
* 创建的脚本都默认继承MonoBehavior，继承了才能挂载在GameObject上
* ==继承了MonoBehavior的脚本不能new，只能挂载==
* 继承了MonoBehavior的脚本不要去写构造函数，因为不会去new它，写构造函数没有意义
* 继承了MonoBehavior的脚本可以在一个对象上挂载多个（如果没有加`[DisallowMutipleComponent]`特性）
* 继承了MonoBehavior的类也可以再次被继承，遵循面向对象继承多态的规则

#### 3.不继承MonoBehavior的类
* 没有继承MonoBehavior的类不能挂载在GameObject上
* 没有继承MonoBehavior的类，想怎么写就怎么写，有需要就自己new
* ==没有继承MonoBehavior的类，一般是单例模式的类（用于管理模块）或者数据结构类（用于存储数据）==
* 没有继承MonoBehavior的类，不必保留默认生成的生命周期函数

#### 关于继承MonoBehavior类的构造函数
虽然建议大家不在继承MonoBehavior的类中写构造函数

但是不意味着我们不能写，当我们在继承MonoBehavior的类中写无参构造函数时，你会发现在编辑模式下或者运行后，==只要该脚本挂载在场景中，那么该无参构造函数是会被自动执行的==。

因为Unity的工作原理中提到的反射机制，Unity实际上通过反射帮助我们实例化了该脚本对象，既然要实例化那么肯定是需要new的，只不过Unity中不需要我们自己new继承了MonoBehavior的类，只要挂载后Unity帮助我们做了这件事。

那么为什么不建议大家写构造函数呢？
1.Unity的规则就是，继承MonoBehavior的脚本不能new只能挂载
2.生命周期函数的Awake是类似构造函数的存在，当对象出生就会自动调用
3.写构造函数反而在结构上会破坏Unity设计上的规范

总结：
如果继承MonoBehavior的脚本想要进行初始化相关，可以在Awake或者Start中进行，搞清这两个生命周期函数的执行时机，根据需求选择在哪里进行初始化。

切记！！继承MonoBehavior的脚本不要new，不要new，不要new！！

#### 4.执行的先后顺序
#### 5.默认脚本内容

``` cmd
Editor\Data\Resources\ScriptTemplates
```

### 生命周期函数
#### 了解帧的概念
==游戏的本质就是一个死循环==
每一次循环处理游戏逻辑就会更新一次画面
之所以能看到画面在动是因为切换画面的速度到达一定程度时，人眼就认为画面是流畅的
==一帧就是执行一次循环==

fps（Frames Per Second）：每秒钟帧数
1s = 1000ms
60帧：1帧为 1000ms/60 = 16.66ms
30帧：1帧为 1000ms/30 = 33.33ms

人眼舒适放松时可视帧数是每秒24帧

游戏卡顿的原因：
跑1帧的游戏逻辑计算量过大，或者CPU不给力，不能在一帧的时间内处理完所有游戏逻辑

#### 生命周期函数的概念
所有继承MonoBehaivor的脚本，最终都会挂载到GameObject游戏对象上
生命周期函数就是该脚本对象依附的GameObejct对象从出生到消亡的整个生命周期中，会通过**反射**自动调用的一些特殊函数

#### 生命周期函数
注意：
生命周期函数的访问修饰符一般为`private`和`protected`
因为不需要在外部调用生命周期函数，都是Unity自动调用的

![生命周期函数](./img/生命周期函数.png)

在Unity打印信息的两种方式：
1. 没有继承MonoBehavior类的时候

``` c#
Debug.Log("");
Debug.LogError("");
Debug.LogWarning("");
```

2. 继承MonoBehavior类的时候，有一个现成的方法可以使用

``` c#
print("");
```

##### Awake()
出生时调用，一个对象只会调用一次

##### OnEnable()
挂载的GameObject对象每次激活时调用

##### Start()

##### FixedUpdate()
物理帧更新
固定间隔时间执行，间隔时间可以设置（在project setting的Time设置）
==主要用于物理更新==

##### Update()
逻辑帧更新
每帧执行

##### LaterUpdate()
每帧执行，于Update后执行
==一般用于处理摄像机位置更新==

##### OnDisable()
挂载的GameObject对象每次失活时调用

##### OnDestroy()
对象销毁时调用
挂载的GameObject对象被删除时调用

#### 生命周期函数支持继承多态

#### 不同对象的生命周期函数是在同一个线程中执行的吗？
答案：Unity中所有对象上挂载的生命周期函数都是在一个主线程中按先后执行的

理解：Unity会主动把场景上的对象，对象上挂载的脚本都统统记录下来，在主线程的死循环中，按顺序按时机的通过反射，执行记录的对象身上挂载的脚本的对应生命周期函数



### Inspector窗口可编辑的变量
Inspector窗口显示的可编辑内容就是脚本的成员变量

#### 知识点一：private、protected成员无法显示编辑

#### 知识点二：让private、protected成员也可以被显示
在变量前加上强制序列化字段特性 `[SerializeField]`
所有序列化就是把一个对象保存到一个文件或数据库字段中去

#### 知识点三：public成员可以显示编辑

#### 知识点四：让public成员无法显示编辑
在变量前加上字段特性：`[HideInspector]`

#### 知识点五：大部分类型都能显示编辑
字典Dictionary不能显示
自定义类型变量不能显示

#### 知识点六：让自定义类型可以被访问
加上序列化特性 `[System.SerializeField]`
字典Dictionary怎么样都不行

#### 知识点七：一些辅助特性
1. 分组说明特性 `[Header("分组说明")]`

2. 悬停注释特性 `[Tooltip("悬停注释")]`

3. 间隔特性 `[Space()]`
两个字段出现间隔

4. 修饰数值的滑条范围 `[Range(最小值,最大值)]`

5. 多行显示字符串 `[Multiline(行数)]`
默认不写参数显示3行
写参数就是对应几行

6. 滚动条显示字符串
默认不写参数就是超过3行显示滚动条
`[TextArea(3,4)]` 最少显示3行，最多4行，超过4行就显示滚动条

7. 为变量添加快捷方法
参数1：按钮的名字
参数2：方法名 ==不能有参数 不能有返回值==
`[ContextMenuItem("按钮的名字","方法名")]` 

8. 为方法添加特性 使其能够在Inspector中执行 `[ContextMenu(“测试函数")]`

#### 注意
1. 拖拽到GameObject对象后，再改变脚本中变量默认值，界面上不会改变
2. 运行中修改的信息不会保存


### MonoBehavior中的重要内容
#### 知识点一 重要成员
1. 获取依附的GameObject
`gameObject`

2. 获取依附的GameObject的位置信息 
`transform.position`//位置
`transform.eulerAngles`//角度
`transform.lossyScale`//缩放大小
//这种写法和上面是一样的
`gameObject.transform`

3. 获取脚本是否激活
`enable = true`

#### 知识点二 重要方法
得到依附对象上挂载的其他脚本
只要能获取到场景中别的对象或对象依附的脚本，那么就可以获取到它的所有信息

1. 得到自己挂载的单个脚本
==如果挂载了多个同类型的脚本，则无法知道获取的是哪个脚本==，一般也不会挂载多个同类型脚本
//根据脚本名获取
如果获取失败，默认返回`null`
`脚本名 变量名 = GetComponent("脚本名") as 脚本名`

//根据Type获取
`脚本名 变量名 = GetComponent(typeof(脚本名)) as 脚本名`

//根据泛型获取（建议使用泛型获取 因为不用二次转换）
`脚本名 变量名 = GetComponent<脚本名>()`

2. 得到自己挂载的多个脚本
`脚本名[] 变量名 = GetComponents<脚本名>()`

``` c#
List<脚本名> list = new List<脚本名>();
脚本名[] 变量名 = GetComponents<脚本名>(list);
```

3. 得到子对象挂载的脚本
==默认也会寻找自己身上是否挂载该脚本==
==即使是自己的子对象的子对象，也能找到==
默认参数：`false` 如果子对象失活则不会寻找
`脚本名 变量名 = GetComponentInChildren<脚本名>()`
参数：`true` 即使子对象失活也会寻找
`脚本名 变量名 = GetComponentInChildren<脚本名>(true)`

获取子对象的多个脚本
``` c#
`脚本名[] 变量名 = GetComponentsInChildren<脚本名>(true)`

List<脚本名> list = new List<脚本名>();
脚本名[] 变量名 = GetComponentsInChildren<脚本名>(true, list);
```

4. 得到父对象挂载的脚本
==默认也会寻找自己身上是否挂载该脚本==
==即使是自己的父对象的父对象，也能找到==
`脚本名 变量名 = GetComponentInParent<脚本名>()`

获取父对象的多个脚本
``` c#
`脚本名[] 变量名 = GetComponentsInParent<脚本名>()`

List<脚本名> list = new List<脚本名>();
脚本名[] 变量名 = GetComponetsInParent<脚本名>(list);
```

5. 尝试获取脚本
提供了更加安全的方法，如果得到则返回true
``` c#
List<脚本名> list = new List<脚本名>();
if(TryGetComponent<脚本名>(out list))
{
	...
}
```


## 第7章 Unity重要组件和api
### 最小单位GameObject
#### 知识点一 GameObject中的成员变量
1. 名字 `gameObject.name`
2. 是否激活 `gameObject.activeSelf`
3. 是否是静态 `gameObject.isStatic`
4. 层级 `gameObject.layer`
5. 标签 `gameObject.tag`
6. transform `gameObject.transform`

#### 知识点二 GameObject中的静态方法
1. 创建自带的几何体
`GameObject.CreatePrimitive(PrimitiveType.Cube)`	//创建方块

2. 查找单个对象 
==无法找到失活的对象==
==如果对象名重复，则无法确定找到的是哪一个==

//通过对象名查找
==效率比较低==，因为遍历对象
`GameObject 变量名 = GameObject.Find("对象名")`

//通过标签（tag）来查找
==效率比较低==，因为遍历对象
`GameObject 变量名 = GameObject.FindWithTag("标签名")`
`GameObject 变量名 = GameObject.FindGameObjectWithTag("标签名")`

3. 查找多个对象
==无法找到失活的对象==
==只能通过标签（tag）查找==
`GameObject[] 变量名 = GameObject.FindGameObjectsWithTag("标签名")`

几个是用得比较少的方法，是GameObject父类Object的方法
>Unity的Object不是万物之父object
Unity的Object，命名空间是UnityEngine
C#的object，命名空间是System

//可以找到场景中挂载的某一个脚本对象
==效率更低==，`GameObject.FindObjectOfType<>` 不仅要遍历对象，还要遍历对象上挂载的脚本， `GameObject.Find` 和 `GameObject.FindWithTag`只是遍历对象
`脚本名 变量名 = GameObject.FindObjectOfType<脚本名>()`

这些查找的方法效率低，可以直接在Editor拖动


4. 实例化对象（克隆对象）
克隆的对象：场景上的对象、预设体对象
`GameObject.Instantiate(想要克隆的对象)`

5. 删除对象 

==不仅可以删除对象，还可以删除脚本==
不会马上移除对象，只是给对象添加移除标识，之后会统一处理。一般情况下，==会在下一帧移除并在内存中移除==
==异步删除对象，降低卡顿几率==
`GameObject.Destroy(对象名)`
//第2个参数：延迟5秒
`GameObject.Destroy(对象名, 5)`
//删除脚本
`GameObject.Destroy(脚本对象)`

立刻删除对象
`GameObject.DestroyImmediate(对象名)`

切换场景不移除对象
默认情况下，切换场景，场景对象都会被删除掉
`GameObject.DontDestroyOnLoad(对象名)`
`GameObject.DontDestroyOnLoad(this.gameObject)`

#### 知识点三 GameObject中的成员方法
1. 创建空物体
`GameObject 变量名 = new GameObject();`
//名字
`GameObject 变量名 = new GameObject("名字");`
//加脚本
`GameObject 变量名 = new GameObject(typeof(脚本名1),typeof(脚本名2));`

2. 为对象添加脚本
`脚本名 变量名 = 对象名.AddComponent<脚本名>();`

3. 得到脚本
`this.GetComponent();`
`this.gameObject.GetComponent();`

4. 比较标签
`if(this.gameObject.CompareTag("标签名"))`

5. 设置激活失活
`this.gameObject.SetActive(true);`
`this.gameObject.SetActive(false);`

下面的方法了解即可，不建议使用，因为效率较低
``` c#
//通过广播或者发送消息的形式 让自己或别人执行某些行为方法
//会去遍历脚本本身的方法
this.gameObject.SendMessage("方法名");
this.gameObject.SendMessage("方法名",参数);

//广播行为，让自己和子对象执行
this.gameObject.BroadcastMessage("方法名");

//向父对象和自己发送消息 并执行
this.gameObject.SendMessageUpwards("方法名");
```

### 时间相关Time
主要用于游戏中参与位移、计时、时间暂停等
#### 知识点一 时间缩放比例
1. 时间停止
`Time.Scale = 0`

2. 恢复正常
`Time.Scale = 1`

3. 2倍速
`Time.Scale = 2`

#### 知识点二 帧间隔时间
==主要用来计算位移==，路程 = 时间 * 速度
最近的一帧 用了多长时间（秒）
1. 受Time.Scale影响的帧间隔时间
`Time.deltaTime`

2. 不受Time.Scale影响的帧间隔时间
`Time.unscaledDeltaTime`

如果希望 游戏暂停就不动的，就使用`Time.deltaTime`
如果希望 不受游戏暂停，就使用`Time.unscaledDeltaTime`

#### 知识点三 游戏开始到现在的时间
==主要用来计时==
1. 受Time.Scale影响
`Time.time`

2. 不受Time.Scale影响
`Time.unScaledTime`

#### 知识点四 物理帧间隔时间 FiexedUpdate
1. 受Time.Scale影响
`Time.fixedDeltaTime`

2. 不受Time.Scale影响
`Time.fixedUnscaledDeltaTime`

#### 知识点五 帧数
从开始到现在游戏跑了多少帧（次循环）
`Time.frameCount`

### Transform作用
GameObject的位移、旋转、缩放、父子关系、坐标转换	等相关操作都由它处理

### Transform：位置和位移
#### 知识点一 必备知识点 Vector3基础
Vector3主要是用来表示三维坐标系中的 一个点 或者 一个向量

1. 基本计算 + - * /
2. 常用的（静态）(==世界坐标系的朝向==)
``` c#
Vector3.zero	//000
Vector3.one		//111
Vector3.right	//100
Vector3.left	//-100
Vector3.forward	//001
Vector3.back	//00-1
Vector3.up		//010
Vector3.down	//0-10
```

3. 常用的方法
计算两个点之间的距离
`float Vector3.Distance(Vector3 a,Vector3 b)`

#### 知识点二 位置
1. 相对世界坐标系
`this.transform.position`
通过position得到的位置是相对于 世界坐标系原点的位置
可能和hierarchy面板上显示的不一样
因为如果对象有父子关系，并且父对象位置不在原点，那么和面试上肯定不一样

2. 相对父对象
`this.transform.localPosition`
如果想要和hierarchy面板上显示的一致，那么一定是通过localPosition来进行设置的

position和localPosition一致的情况：
* 父对象的坐标就是原点
* 对象没有父对象

==注意：位置的赋值不能直接改变x、y、z，会报错，只能整体改变==
``` c#
this.transform.position = new Vector3(0,0,0);
this.transform.localPosition = new Vector3(0,0,0);
```

3. 当前对象的各朝向
``` c#
this.transform.right
this.transform.left
this.transform.forward
this.transform.back
this.transform.up
this.transform.down
```

#### 知识点三 位移
坐标系下的位移计算公式
路程 = 方向 * 速度 * 时间

1. 方式一 自己计算
目标位置 = 当前位置 + 路程
``` c#
int velocity = 1;
this.transform.position += this.transform.forward * velocity * Time.deltaTime;
```

2. 方式二 API
==注意：一般使用API来进行位移==
参数一：位移多少（路程 = 方向 * 速度 * 时间）
参数二：相对坐标系（默认相对自己的坐标系 `Space.Self`）
``` c#
int velocity = 1;
//在自己的坐标系下，朝自己前方移动
this.transform.Translate(Vector3.forward * velocity * Time.deltaTime);
//在世界坐标系，朝世界的前方移动
this.transform.Translate(Vector3.forward * velocity * Time.deltaTime,Space.World);
//在世界坐标系，朝自己的前方移动
this.transform.Translate(this.transform.forward * velocity * Time.deltaTime,Space.World);

//在自己的坐标系下，向this.transform.forward移动（一定不会让物体这样移动，会很奇怪）
//因为this.transform.forward是世界坐标系下的方向
this.transform.Translate(this.transform.forward * velocity * Time.deltaTime)
```

### Transform：角度和旋转
#### 知识点一 角度
1. 相对世界坐标系
`this.transform.eulerAngles`

2. 相对父对象
`this.transform.localEulerAngles`
如果想要和hierarchy面板上显示的一致，那么一定是通过localEulerAngles来进行设置的

==注意：设置角度和设置位置一样，不能单独设置x、y、z，需要一起设置==
``` c#
this.transform.eulerAngles= new Vector3(0,0,0);
this.transform.localEulerAngles= new Vector3(0,0,0);
```

#### 知识点二 旋转
1. 方式一 自己计算

2. 方式二 API
==注意：一般使用API来进行位移==

``` c#
//自转
//参数一：旋转角度（旋转速度* 时间）
//参数二：相对坐标系（默认相对自己的坐标系 `Space.Self`）
this.transform.Rotate(new Vector3(0,10,0) * Time.deltaTime);
this.transform.Rotate(new Vector3(0,10,0) * Time.deltaTime,Space.World);

//相对于某个轴转
//参数一：相对的轴
//参数二：旋转角度（旋转速度* 时间）
//参数三：相对坐标系（默认相对自己的坐标系 `Space.Self`）
this.transform.Rotate(Vector3.up, 10*Time.deltaTime);
//相对于世界的轴转
this.transform.Rotate(Vector3.up, 10*Time.deltaTime,Space.World);

//相对于某个点转
//参数一：相对的点
//参数二：相对的点的轴
//参数三：旋转角度（旋转速度* 时间）
this.transform.RotateAround(Vector3.zero, Vector3.up, 10*Time.deltaTime);
```

### Transform：缩放和看向
#### 知识点一 缩放
1. 相对世界坐标系
`this.transform.lossyScale`

2. 相对本地坐标系（父对象）
`this.transform.localScale`

==注意：设置角度和设置位置一样，不能单独设置x、y、z，需要一起设置==
==相对于世界坐标系的缩放大小lossyScale 只能得，不能改==
一般修改缩放大小，只能改`localScale`
==Unity没有提供缩放的API==

#### 知识点二 看向
让一个对象的面可以一直看向某一个点或某个对象

``` c#
//看向一个点 相对于世界坐标系
this.transform.LookAt(Vector3.zero);

//看向一个对象（transform）
Transform objTransform;
this.transform.LookAt(objTransform);
```

### Transform：父子关系
#### 知识点一 获取和设置父对象
1. 获取父对象
`this.transform.parent`

2. 设置父对象
`this.transform.parent = null`

3. 通过API设置父对象
``` c#
this.transform.SetParent(null);

//参数一：父对象transform
//参数二：是否保留在世界坐标系的位置、角度、缩放
//	true：保留世界坐标系下的状态，和父对象的进行计算，得到本地坐标系的信息
//	false：直接把世界坐标系下的状态 直接赋值到 本地坐标系中
this.transform.SetParent(null,true)
```

#### 知识点二 移除所有子对象
`this.transform.DetachChildren()`
==只能断绝第一层的父子关系，不能断绝子对象和孙对象之间的关系==

#### 知识点三 获取子对象transform
1. 按名字查找子对象
==可以找到失活的子对象==，`GameObject.Find()`找不到失活的对象
==找不到子对象的子对象==
`this.transform.Find("子对象名字")`

2. 遍历子对象
``` c#
//子对象的数量（包括失活的，不算孙对象的数量）
this.transform.childCount;

//通过索引得到子对象
this.transfotm.GetChild(0);

for(int i=0;i<this.transform.childCount;++i)
{
	Transform child = this.transfotm.GetChild(i);
}
```

#### 知识点四 子对象的操作
1. 判断自己的父对象是谁
`if(this.transform.IsChildOf(父对象Transform)) {}`

2. 得到自己作为子对象的编号
`this.transform.GetSiblingIndex()`

3. 把自己设置为第一个子对象
`this.transform.SetAsFirstSibling()`

4. 把自己设置为最后一个子对象
`this.transform.SetAsLastSibling()`

5. 设置自己作为子对象的编号
==即使超出范围或负数，也不会报错，则会指定作为最后一个子对象==
`this.transform.SetSiblingIndex(编号)`

### Transform：坐标转换
#### 知识点一 世界坐标转换本地坐标
可以帮助我们大概判断一个相对位置
1. 世界坐标系的点 转换为 本地坐标系的点
受到缩放影响
`this.transform.InverseTransformPoint(Vector3.forward)`

2. 世界坐标系的方向 转换为 本地坐标系的方向
不受缩放影响
`this.transform.InverseTransformDirection(Vector3.forward)`

受缩放影响
`this.transform.InverseTransformVector(Vector3.forward)`

#### 知识点二 本地坐标转换世界坐标（比较重要 释放技能等）
1. 本地坐标系的点 转换为 世界坐标系的点
受到缩放影响
`this.transform.TransformPoint(Vector3.forward)`

2. 本地坐标系的方向 转换为 世界坐标系的方向
不受缩放影响
`this.transform.TransformDirection(Vector3.forward)`

受到缩放影响
`this.transform.TransformVector(Vector3.forward)`

### 输入相关Input
注意：输入相关肯定是写在Update里的

#### 知识点一 鼠标在屏幕位置
屏幕的原点是在屏幕左下角
往右是x轴
往上是y轴
返回值是Vector3，z轴一直是0
``` c#
Input.mousePosition
```

#### 知识点二 检测鼠标输入
``` c#
//1.鼠标按下的一瞬间 进入
//参数：0左键 1右键 2中键
if(Input.GetMouseButtonDown(0)) {}

//2.鼠标抬起的一瞬间 进入
//参数：0左键 1右键 2中键
if(Input.GetMouseButtonUp(0)) {}

//3.鼠标长按、按下、抬起都会进入
//参数：0左键 1右键 2中键
if(Input.GetMouseButton(0)) {}

//4.中键滚动
//返回值Vector3的y -1往下滚动 0没有滚动 1往上滚动
Input.mouseScrollDelta
```

#### 知识点三 检测键盘输入

``` c#
//1.键盘按下
//	传入枚举KeyCode的重载
if(Input.GetKeyDown(KeyCode.W)) {}
//	传入字符串的重载
//(传入的字符串不能是大写的)
if(Input.GetKeyDown("s")) {}

//3.键盘抬起
if(Input.GetKeyUp(KeyCode.W)) {}

//4.键盘长按
if(Input.GetKey(KeyCode.W)) {}

```

#### 知识点四 检测默认轴输入
Unity提供了更方便的方法来帮助控制对象的位移和旋转
``` c#
//1.键盘AD按下时，返回float -1到1之间的变化
Input.GetAxis("Horizental");

//2.键盘SW按下时，返回float -1到1之间的变化
Input.GetAxis("Vertical");

//3.鼠标横向移动时，返回-1到1，从左到右
Input.GetAxis("Mouse X");

//4.鼠标竖向移动时，返回-1到1，从上到下
Input.GetAxis("Mouse Y");

//5.GetAxisRaw 和 GetAxis 使用方式相同
//	返回值不同 只有 -1 0 1 不会有中间值
Input.GetAxisRaw("Horizental");
```

#### 其他
``` c#
//1.是否有任意键或鼠标长按
if(Input.anyKey) {}

//2.是否有任意键或鼠标按下
if(Input.anyKeyDown) {}

//3.这一帧的键盘输入
Input.inputString

//和任意键配合使用
if(Input.anyKeyDown) 
{
	print(Input.inputString);
}
```

#### 手柄输入
``` c#
//1.得到连接的手柄的所有按钮名字
string[] keys = Input.GetJoystickNames();

//2.某一个手柄键按下
if(Input.GetButtonDown("X")) {}

//3.某一个手柄键抬起
if(Input.GetButtonDown("Y")) {}

//4.某一个手柄键长按
if(Input.GetButton("A")) {}

```

#### 移动设备触摸相关
``` c#
if(Input.touchCount > 0)
{
	Touch t1 = Input.touches[0];
	
	//手指位置
	t1.position
	//相对上次位置的变化
	t1.deltaPosition
}

//2.是否启用多点触控
Input.multiTouchEnabled = false;
```

#### 陀螺仪
``` c#
//1.开启陀螺仪
Input.gyro.enabled = true;

//2.重力加速度
Input.gyro.gravity

//3.旋转速度
Input.gyro.rotationRate

//4.当前旋转的四元数
//	例如 用这个角度信息来控制场景上的一个3D物体收到重力影响，手机怎么动，它就怎么动
Input.gyro.attitude
```

### 屏幕相关Screen
#### 知识点一 静态属性
常用的
``` c#
//1.当前屏幕分辨率
Resolution r = Screen.currentResolution;
// 分辨率的宽
r.width
// 分辨率的高
r.height

//2.屏幕窗口当前宽高
//	(一般写代码使用这个)
Screen.width
Screen.height

//3.屏幕休眠模式
Screen.sleepTimeout = SleepTimeout.NeverSleep;
```

不常用的
``` c#
//运行时是否全屏

```

#### 知识点二 静态方法
``` c#
//1.设置分辨率 一般移动设备不使用
//	参数三：是否全屏
Screen.SetResolution(1920,1080,true);
```

### 摄像机Camera
#### Camera可编辑参数
常用的
1. Clear Flags
如何清除背景
//	Skybox：天空盒（多用在3D游戏）
//	Solid Color：颜色填充（多用在2D游戏）
//	Depth Only：只画该层，背景透明
//	Don't Clear：不移除，覆盖渲染，不会擦除上一帧的图像

2. Culling Mask
选择性渲染部分层级(Layer)，可以指定只渲染对应层级的对象
取消勾选对应的层级，就看不到对应层级的对象

3. Projection
透视方式
//	Perspective：透视模式

//	Orthographic：正交摄像机（一般用于2D游戏）

4. Clipping Planes
裁剪平面距离

5. Depth
渲染顺序上的深度
只有一个Camera是没有作用的
数字越小越先被渲染

6. Target Texture
渲染纹理
可以把摄像机画面渲染到一张图上（主要用于小地图的制作）
可以右键创建Render Texture，再拖到Target Texture

7. Occlusion Culling
是否启用剔除遮挡
如果有些对象被挡住了，就可以不渲染，节约计算

不常用的
8. Viewport Rect

9. Redering path

10. Allow HDR

11. Allow MSAA

12. Allow Dynamic Resolution

13. Target Display

#### Camera代码相关
##### 知识点一 重要静态成员
``` c#
//1.获取摄像机
//	获取主摄像机(此方法场景上必须要有tag为Main Camera的camera)
Camera.main

//	获取摄像机数量
Camera.allCamerasCount

//	得到所有摄像机
Camera[] allCameras = Camera.allCameras;

//2.渲染相关委托
//	摄像机剔除前处理的委托
Camera.onPreCull += (c) => {};
//	摄像机渲染前处理的委托
Camera.onPreRender += (c) => {};
//	摄像机渲染后处理的委托
Camera.onPostRender += (c) => {};
```

##### 知识点二 重要成员
``` c#
//1.hierarchy界面上的参数 都可以在Camera中获取到
Camera.main.depth

//2.世界坐标转屏幕坐标
//	xy轴对应屏幕坐标，z轴表示离Camera的距离
//	可以用来做头顶血条功能
Vector3 v = Camera.main.WorldToScreenPoint(this.transform.position);

//3.屏幕坐标转世界坐标
Camera.main.ScreenToWorldPoint(Input.mousePosition)
//改变z轴
//	之所以改变z轴，是因为z轴默认是0
//	转换过去的世界坐标的点，永远都是视口相交的点
//	改变了z轴，转换过去的世界坐标的点，就是摄像机前方多少单位的横截面上的世界左边点
Vector3 v = Input.mousePosition;
v.z = 5;
Camera.main.ScreenToWorldPoint(v);
```

## 第8章 核心系统
### 光源系统基础

### 物理系统之碰撞检测
#### 知识点一 碰撞产生的必要条件
两个物体都有碰撞器，至少一个物体有刚体

#### 刚体
**刚体会利用体积进行碰撞计算，模拟真实的碰撞效果，产生力的作用**

1. Mass
质量

2. Drag
空气阻力
==影响位移的阻力==

3. Angular Drag
根据扭矩旋转对象时，影响对象的空气阻力大小
==影响旋转的阻力==

4. Use Gravity
是否收到重力影响

5. Is Kinematic
如果启用此选项，则对象将不会被物理引擎驱动，只能通过transform进行操作。
对于移动平台，或者如果要动画化附加了HingeJoint的刚体，此属性将非常有用。

6. Interpolate
插值运算
让刚体物体运动更加平滑
（适用于物理帧长的时候）

//None：不应用插值运算
//Interpolate：根据前一帧的变换来平滑变换
//Extraploate：差值运算，根据下一帧的估计变化来平滑变换

7. Collision Detection
碰撞检测模式

//Discrete：离散检测（==默认==）
对场景中的所有其他碰撞体使用离散碰撞检测
==最不耗性能，也是最相对不准确的==

//Continuous：连续检测
==对物理性能有很大影响，如果没有快速对象的碰撞问题，请将其保留为Discrete设置==
对动态碰撞体（具有刚体）使用离散碰撞检测
对静态碰撞体（没有刚体）使用连续碰撞检测

//Continuous Dynamic：连续动态检测
==性能消耗高==
==最准确的==
==用于快速移动的对象==
对设置为连续碰撞检测和连续动态碰撞检测的对象使用连续碰撞检测，还将对静态物体（没有刚体）使用连续碰撞检测，对于其他所有碰撞体，使用离散碰撞检测

//Continuous Speculaive：连续推测检测

性能消耗关系（性能消耗越高越准确）：
Continuous Dynamic > Continuous Speculaive > Continuous > Discrete

![enter description here](./img/刚体参数3.png)

8. Constraints：约束
对刚体运动的限制

//Freeze Position：有选择地停止刚体沿世界X、Y、Z轴的移动

//Freeze Rotation：有选择地停止刚体沿世界X、Y、Z轴旋转

##### 刚体的休眠
Unity为了节约性能，让刚体停止计算（==坑点==）
``` c#
//判断刚体是否休眠
if(rigidbody.IsSleeping())
{
	//唤醒刚体
	rigidbody.WakeUp();
}
```

#### 碰撞器
**碰撞器表示物体的体积（形状）**

##### 3D碰撞器种类
1. 盒状

2. 球状

3. 胶囊

4. 网格

5. 环状

5. 地形

##### 共同参数
1. Is Trigger
是否是触发器

2. Material
物理材质，可以确定碰撞体和其他对象碰撞时的交互（表现）方式
（例如篮球反弹、在冰面上滑动等）

3. Center
碰撞体在对象局部空间中的中心位置

##### 常用碰撞器
盒状
球状
胶囊

##### 异形物体使用多种碰撞器组合
刚体对象的子对象碰撞器参与碰撞检测

##### 不常用碰撞器
1. Mesh Collider：网格碰撞器

2. Wheel Collider：环状碰撞器
赛车游戏用得比较多

3. Terrain Collider：地形碰撞器

#### 物理材质
1. 创建物理材质
右键

2. 物理材质参数
	* Dynamic Friction：在移动时使用的摩擦力。
	  通常为0-1的值，值为0就像冰一样，值为1将使对象迅速静止（除非用很大的力或重力推动对象）
	* Static Friction：静止在表面时使用的摩擦力
	  通常为0-1的值，值为0就像冰一样，值为1将导致对象很难移动
	* Bounciness：表面的弹性
	  值为0将不会反弹，值为1表示在反弹时不会产生任何能量损失
	* Friction Combine：两个碰撞对象的摩擦力的组合方式
	* Bounce Combine：两个碰撞对象的弹性的组合方式 


#### 碰撞检测函数
注意：碰撞和触发响应函数 属于 特殊的生命周期函数，也是通过反射调用
##### 知识点一 物理碰撞检测响应函数
``` c#
// 碰撞刚开始时
void OnCollisionEnter(Collision collision ) 
{
	//1.碰撞到的对象的碰撞器的信息
	collision.collider
	
	//2.碰撞到的对象的GameObject
	collision.gameObject
	
	//3.碰撞到的对象的transform
	collision.transform
	
	//4.触碰点数相关
	collision.contactCount
	//接触点 具体坐标
	ContactCount[] pos = collision.contacts;
}

// 碰撞结束分离时
void OnCollisionExit(Collision other) {}

// 两个物体相互解除摩擦时 会不停地调用
void OnCollisionStay(Collision other) {}
```

##### 知识点二 触发器响应检测函数
``` c#
//接触刚开始时
void OnTriggerEnter(Collider other) {}

//接触结束分离时
void OnTriggerExit(Collider other) {}

//一直接触时 会不停调用
void OnTriggerStay(Collider other) {}
```

##### 知识点三 要明确何时会响应函数
1. 只要挂载的对象能和别的对象产生碰撞或者触发，那么对应的6个函数都能被响应

2. 如果是异形物体，刚体在父对象上，如果在子对象上挂载脚本检测碰撞是不行的，必须挂载到父对象上

##### 知识点四 碰撞和触发响应函数都可以写成虚函数

#### 刚体加力
让其有一个速度 朝一个方向移动
加力后 物体是否停止 是由刚体的阻力和扭矩阻力决定的

##### 知识点一 刚体自带添加力的方法
``` c#
//1. 首先获取刚体组件
Rigidbody rigidBody = this.GetComponent<RigidBody>();

//2.添加力
//	相对世界坐标系 向世界的前方移动
rigidBody.AddForce(Vector3.forward * 10);
//	相对世界坐标系 向自己的前方移动
rigidBody.AddForce(this.transform.forward * 10);
//	相对本地坐标系
rigidBody.AddRelativeForce(Vector3.forward * 10);

//3.添加扭矩力，让其旋转
//	相对世界坐标系
rigidBody.AddTorque(Vector3.up * 10);
//	相对本地坐标系
rigidBody.AddRelativeTorque(Vector3.up * 10);

//4.直接改变速度
//速度方向是相对于世界坐标系的
rigidBody.velocity = Vector3.forward * 5;

//5.模拟爆炸效果
// 只影响调用函数的脚本的物体
//	参数一：力的大小
//	参数二：爆炸中心
//	参数三：爆炸半径
rigidBody.AddExplosionForce(10, Vector3.zero,10);
```

##### 知识点二 力的几种模式
动量定理：Ft = mv
v = Ft / m
F：力
t：时间
m：质量
v：速度
``` c#
//1.Acceleration 加速度模式
// 给物体添加一个持续的加速度，忽略其质量，默认为1
rigidBody.AddForce(Vector3.forward * 10, ForceMode.Acceleration);

//2.Force
// 给物体添加一个持续的力，与物体的质量有关
rigidBody.AddForce(Vector3.forward * 10, ForceMode.Force);

//3.Impulse
// 给物体添加一个瞬间的力，与物体的质量有关，忽略时间，默认为1
rigidBody.AddForce(Vector3.forward * 10, ForceMode.Impulse);

//4.VelocityChange
// 给物体添加一个瞬时速度，忽略其质量和时间，默认为1
rigidBody.AddForce(Vector3.forward * 10, ForceMode.VelocityChange);
```

##### 知识点三 力场脚本
Unity自带的Constant Force脚本
比较适用于场景

### 音效系统

## 入门实践 TankGame
### 必备知识点 场景切换和退出游戏
#### 知识点一 场景切换
需要引用命名空间 `UnityEngine.SceneManagement`
需要把场景加载到场景列表中：File -> Build Settings ->拖动/Add Open Scenes
场景列表的第一个场景是打包后运行的默认场景
``` c#
//切换场景
SceneManagement.LoadScene("场景名字");
```

#### 知识点二 退出游戏
在编辑模式下没有作用，一定是发布游戏后才有作用
``` c#
Application.Quit();
```

### 必备知识点 鼠标隐藏锁定相关
#### 知识点一 隐藏鼠标
`Cursor.visible = false;`

#### 知识点二 锁定鼠标
``` c#
//None：不锁定
//Locked：锁定，鼠标会被限制在屏幕的中心点，还会被隐藏
//Confined：限制在窗口范围内
Cursor.lockState = CursorLockMode.None;
//锁定
```

#### 知识点三 设置鼠标图片
``` c#
Texture2D tex;

//参数一：光标图片
//参数二：偏移位置，相对图片左上角
//参数三：平台支持的光标模式（硬件或软件）
Cursor.SetCursor(tex, Vector2.zero, CursorMode.Auto);

```

### 必备知识点 随机数和Unity自带委托相关
#### 知识点一 随机数
Unity中的随机数
``` c#
UnityEngine.Random

//整形 左包含 右不包含
int num = Random.Range(0,100);
//浮点型 左包含 右包含
float num = Random.Range(1f,99f);
```

C#中的随机数
``` c#
System.Random

Random random = new Random();
random.Next()
```

#### 知识点二 委托
Unity的委托
需要命名空间 `UnityEngine.Events`
``` c#
UnityAction u_ac = ()=>{};

UnityAction<int,float> u_ac2 = (i,f) =>{};


```

C#的委托
``` c#
System.Action ac = ()=>{};

System.Action<int,float> ac2 = (i,f) =>{};

System.Func<float> func = ()=>{ return 1f; };

System.Func<int,float> func2 = (i)=>{ return 1f; };
```

### 必备知识点 模型资源的导入
#### 模型由什么构成
骨（骨骼）：非必须，有动作的模型才需要

肉（网格面片）：必须，决定了模型的轮廓

皮（贴图）：必须，决定了模型的颜色效果

#### Unity支持的模型格式
==官方推荐的是fbx格式的模型==

其他格式虽然支持，但是不推荐
.dae
.3ds
.dxf
.obj

#### 如何指导美术导出模型
Unity官网有指导手册
https://docs.unity.cn/cn/2019.4/Manual/CreatingDCCAssets.html

重点规则：
==Unity中模型面朝向朝模型坐标系的Z轴==
==缩放大小单位要注意==

#### 学习阶段在哪获取模型资源
AssetStore
淘宝
等

获取的资源一般两种形式
1. Unity资源包
2. 原生的模型贴图资源（需要进行一些设置）

### 需求分析
完成以下学习：
Unity入门
数据持久化——PlayerPrefs
UI——GUI

# 二、Unity基础
## 第2章 基础知识点
### 3D数学--基础
#### Mathf 数学计算公共类
##### 知识点一 Mathf和Math
Math是C#中封装好的用于数学计算的工具==类==，位于`System`命名空间
Mathf是Unity中封装好的用于数学计算的工具==结构体==，位于`UnityEngine`命名空间

##### 知识点二 Mathf和Math的区别
Mathf不仅包含Math中的方法，还多了适用于游戏开发的方法

##### 知识点三 Mathf中的常用方法（一般只计算一次）
1. PI

2. Abs（绝对值）

3. CeilToInt（向上取整）
1.3 -> 2

4. FloorToInt（向下取整）
==强制转换都是向下取整的==
1.3 -> 1

5. Clamp（钳制函数）
``` c#
Mathf.Clamp(10, 11, 20);	//11
Mathf.Clamp(21, 11, 20);	//21
Mathf.Clamp(15, 11, 20);	//15
```

6. Max（取最大值）

7. Min

8. Pow（一个数的n次幂）

9. RoundToInt（四舍五入）

10. Sqrt（返回一个数的平方根）

11. IsPowerOfTwo（判断一个数是否是2的n次方）
==可以用来判断这个数的二进制某一个是否是1==

12. Sign（判断正负数）
返回值 float
整数 1
负数 -1

##### 知识点四 Mathf中的常用方法（一般不停计算）
1. Lerp（插值运算）
==可以用来做跟随、缓动==

`result = Mathf.Lerp(start, end, t)`

t为插值系数，取值范围 0~1
result = start + (end - start)* t

* 插值运算用法一
	每帧改变start值——变化速度先快后慢，位置无限接近，但是不会得到end位置
``` c#
int start = 0;
start = Mathf.Lerp(start, 10, Time.deltaTime);
```

* 插值运算用法二
	每帧改变t值——变化速度匀速，位置每帧接近，当 t>=1 时，得到结果
``` c#
float time = 0;
int start = 0;
int result = 0;

time += Time.deltaTime;
result = Mathf.Lerp(start, 10, time);
```


#### 三角函数
##### 1.角度和弧度
1. 角度和弧度都是度量角的单位

角度：1°
弧度：1 radian

圆一周的角度：360°
圆一周的弧度：2π radian

2. 角度和弧度的转换关系
π rad = 180°
1 rad = （180 / π)°  => 57.3°
1° = (π / 180) rad  => 0.01745 rad

由此可以得出：
弧度 * 57.3			= 对应角度
角度 * 0.01745	 = 对应弧度

##### 知识点一 弧度和角度的相互转化
``` c#
//弧度转角度
float rad = 1;
float anger = rad * Mathf.Rad2Deg;

//角度转弧度
rad = anger * Mathf.Deg2Rad;
```

##### 2.三角函数
三角函数是初等函数之一，包括正弦函数、余弦函数、正切函数等

sin（正弦函数）：对边 / 斜边
cos（余弦函数）：邻边 / 斜边

##### 知识点二 三角函数
注意：Mathf的三角函数的相关函数，传入的参数需要是==弧度值==
``` c#
Mathf.Sin(30* Mathf.Deg2Rad);
Mathf.Cos(60* Mathf.Deg2Rad);
```

##### 3.三角函数曲线

##### 练习 让物体反复曲线移动

``` c#
public float moveSpeed_ = 5;
//左右曲线移动变化的速度
public float horizontalSpeed_ = 2;
//左右曲线移动的距离
public float horizontalDis_ = 2;

float time_ = 0;

void Update()
{
	transform.Translate(Vector3.forward * moveSpeed_ * Time.deltaTime);
	
	time += Time.deltaTime * horizontalSpeed_ ;
	transform.Translate(Vector3.right * horizontalDis_ * Time.deltaTime * Mathf.Sin(time));
}
```

##### 4.反三角函数
反三角函数是初等函数之一，包括反正弦函数、反余弦函数等

==作用：通过反三角函数计算正弦值或余弦值的对应弧度值==

##### 知识点三 反三角函数
注意：返回的是==弧度值==
``` c#
int deg = Mathf.Asin(0.5f) * Mathf.Rad2Deg;
deg = Mathf.Acos(0.5f) * Mathf.Rad2Deg;
```

#### Unity中的坐标系
##### 世界坐标系
原点：世界的中心点

轴向：==三个轴向是固定的==

``` c#
this.transform.position
this.transform.rotation
this.transform.eulerAngles
this.transform.lossyScale
```

##### 物体坐标系
原点：物体的中心点（建模时决定的）

轴向：
物体==右方==为`x`轴正方向
物体==上方==为`y`轴正方向
物体==前方==为`z`轴正方向

相对父对象的物体坐标系
``` c#
this.transform.localPosition
this.transform.localRotation
this.transform.localEulerAngles
this.transform.localScale
```

##### 屏幕坐标系
原点：屏幕左下角

轴向：
==右方==为`x`轴正方向
==上方==为`y`轴正方向

最大宽高：
`Screen.width`
`Screen.height`

``` c#
Input.mousePosition
Screen.width
Screen.height
```

##### 视口坐标系
原点：屏幕左下角

轴向：
==右方==为`x`轴正方向
==上方==为`y`轴正方向

特点：
左下角为 (0, 0)
右上角为 (1, 1)
和屏幕坐标系类似，将坐标单位化

摄像机上的视口范围
Camera对象的Inspector面板的==Viewport Rect==参数

##### 坐标转换
世界转本地
``` c#
this.transform.InverseTransformDirection
this.transform.InverseTransformPoint
this.transform.InverseTransformVector
```

本地转世界
``` c#
this.transform.TransformDirection
this.transform.TransformPoint
this.transform.TransformVector
```

世界转屏幕
``` c#
Camera.main.WorldToScreenPoint
```
屏幕转世界
``` c#
Camera.main.ScreenToWorldPoint
```

世界转视口
``` c#
Camera.main.WorldToViewportPoint
```
视口转世界
``` c#
Camera.main.ViewportToWorldPoint
```

视口转屏幕
``` c#
Camera.main.ViewportToWorldPoint
```
屏幕转视口
``` c#
Camera.main.WorldToViewportPoint
```

### 3D数学--向量 Vector3
#### 向量模长和单位向量
标量：有数值大小，没有方向

向量：有数值大小，有方向的矢量

##### 知识点一 向量
``` c#
Vector3 v3 = new Vector3(x, y, z);
Vector2 v2 = new Vector2(x, y);
```

1. 位置——代表一个点
``` c#
this.transform.position
```

2. 方向——代表一个方向
``` c#
this.transform.forward
this.transform.up
```

##### 知识点二 两点决定一向量
==终点减去起点==
AB向量 = B向量 - A向量

##### 知识点三 零向量和负向量
零向量：(0, 0, 0)
`Vector3.zero`

负向量：
负向量和原向量大小相等
负向量和原向量方向相反
``` c#
Vector3 A = new Vectro3(1, 2, 3);
Vector3 E_A = -A;
```

##### 知识点四 向量模长
向量的模长：向量的长度，两个点的距离

公式：A向量(x, y, z) = √ x² + y² + z²
``` c#
Vector3 A = new Vector3(1, 2, 3);

//相对原点的模长
A.magnitude

Vector3 B = new Vector3(6, 2, 1);
Vector3 AB = B - A;
AB.magnitude

//获得两点之间的距离，与模长有关联
Vector3.Distance(A, B);
```

##### 知识点五 单位向量
模长为1的向量
任意一个向量经过归一化就是单位向量

==只需要方向，不想让模长影响计算结果时使用单位向量==

归一化公式：
A向量(x, y, z)模长 = √ x² + y² + z²
单位向量 = (x/模长, y/模长, z/模长)

``` c#
AB.normalized
```

#### 向量加减乘除
1. 加法
向量A(x1, y1, z1)
向量B(x2, y2, z2)

A + B = (x1+x2, y1+y2, z1+z2)

加法的意义：
位置 + 位置 = 位置：两个位置相加没有任何意义

**向量 + 向量 = 向量**
==向量相加，首尾相连==

**向量 + 位置 = 位置**
==位置和向量相加 = 平移位置==

2. 减法
减法的意义：
**位置 - 位置 = 向量**
==两点决定一向量，终点 - 起点==

**向量 - 向量 = 向量**
==向量相减，头连头，尾指尾==

**位置 - 向量 = 位置**
相当于往向量的相反方向（负向量）平移位置
==位置减向量 = 平移位置==

向量 - 位置：没有任何意义

3. 乘除
向量只会和标量进行乘除
向量A(x, y, z)
标量a

A* a = (x * a, y * a, z * a)
A/ a = (x / a, y / a, z / a)

向量 乘除 标量 = 向量
向量 乘除 正数，==方向不变，放大缩小模长==
向量 乘除 负数，==方向相反，放大缩小模长==
向量 * 0 = 零向量


向量加法——主要用于**位置平移**和向量计算
向量减法——主要用于**位置平移**和向量计算
向量乘除——主要用于**模长放大缩小**

#### 向量点乘
1. 点乘计算公式
向量A(x1, y1, z1)
向量B(x2, y2, z2)
A·B = x1 * x2 + y1 * y2 + z1 * z2

向量 · 向量 = 标量

2. 点乘几何意义
点乘可以得到一个向量
在自己向量上==投影的长度==

==点乘结果 > 0 ：两个向量夹角为锐角==
==点乘结果 = 0 ：两个向量夹角为直角==
==点乘结果 < 0 ：两个向量夹角为钝角==

==可以用来判断敌方的大致位置==
##### 知识点一 通过点乘判断对象方位
``` c#
float dotRest = Vector3.Dot(transform.position, 
	target.position-transform.position);
if(dotRes > 0) {}
else if(dotRes < 0) {}
```

##### 知识点二 通过点乘推导公式算出夹角

3. 点乘公式推导
	1. cosβ = 直角边 / 单位向量B模长
		直角边 = 单位向量B模长 * cosβ
					+
	2. 直角边 = 单位向量A · 单位向量B
					↓
		cosβ * 单位向量B模长 = 单位向量A · 单位向量B
					↓
		cosβ = 单位向量A · 单位向量B
					↓
==推出结果：β = Acos(单位向量A · 单位向量B)==

``` c#
//第一种方法
//	1.用单位向量算出点乘结果
float dotRes = Vector3.Dot(transform.forward,
	(target.position - transform.position).normalized);

//	2.用反三角函数得出结果
Mathf.Acos(dotRest) * Mathf.Rad2Deg

//第二种方法
Vector3.Angle(transform.forward, target.position - transform.position);
```

##### 练习 发现入侵者
当一个物体B在物体A前方45度角的扇形范围内，并且离A只有5米距离时，在控制台打印“发现入侵者”
``` c#
public Transform target_;

void Update()
{
	//方法一
	if(Vector3.Distance(transform.position, target.position) <= 5)
	{
		//1.算出点乘结果
		float dotRes = Vector3.Dot(transform.forward, 
			(target.position - transform.position).normalized);
		//2.通过反三角函数
		if(Mathf.Acos(dotRes) * Mathf.Rad2Deg <= 22.5f)
		{
			print("发现入侵者");
		}
	}
	
	//方法二
	if(Vector3.Distance(transform.position, target.position) <= 5 && 
		Vector3.Angle(transform.forward, target.position - transform.position) <= 22.5f)
	{
		print("发现入侵者");
	}
}
```


#### 向量叉乘
==向量 × 向量 = 向量==

向量A(Xa, Ya, Za)
向量B(Xb, Yb, Zb)
A × B = (YaZb - ZaYb, ZaXb - XaZb, XaYb - YaXb)

**记忆：新向量的X，A和B的X不会参与计算，从Y开始；新向量的Y，A和B的Y不会参与计算，从Z开始；新向量的Z，A和B的Z不会参与计算，从X开始**

##### 知识点一 叉乘计算
``` c#
Transform A;
Transform B;

Mathf.Cross(A.position, B.position);
```

##### 知识点二 叉乘几何意义
A × B得到的向量同时垂直于A和B
A × B得到的向量垂直于A和B组成的平面
A × B = -(B × A)

假设向量A和B都在XZ平面上
A × B
y > 0，证明B在A右侧
y < 0，证明B在A左侧

``` c#
Transform A;
Transform B;

void Update()
{
	Vector3 cross = Mathf.Cross(A.position, B.position);
	if(cross.y > 0)
	{
		print("B在A的右侧");
	}
	else
	{
		print("B在A的左侧");
	}

	Vector3 cross = Mathf.Cross(B.position, A.position);
	if(cross.y > 0)
	{
		print("A在B的右侧");
	}
	else
	{
		print("A在B的左侧");
	}
}

```

##### 练习
1. 判断一个物体B的位置在另一个物体A的位置的左上，左下，右上，右下哪个方位
``` c#
Transform A;
Transform B;

void Update()
{
	float dotRes = Vector3.Dot(A.forward, B.position - A.position);
	Vector3 crossRes = Vector3.Cross(A.forward, B.position - A.position);

	//判断前后
	if( dotRes >= 0)
	{
		if( crossRes.y >= 0)
		{
			print("右上");	
		}
		else
		{
			print("左上");
		}
	}
	else
	{
		if( crossRes.y >= 0)
		{
			print("右下");	
		}
		else
		{
			print("左下");
		}
	}
}
```

2. 当一个物体B在物体A左前方20度角或右前方30度角的扇形范围内，并且离A只有5米距离时，在控制台打印“发现入侵者”
``` c#
Transform A;
Transform B;

void Update()
{
	if(Vector3.Distance(A.position, B.position) <= 5)
	{
		Vector3 crossRes = Vector3.Cross(A.forward, B.position - A.position);
		if(crossRes.y >= 0 && 
			Vector3.Angle(A.forward, B.position - A.position) <= 30)
		{
			print("右侧30度，发现入侵者");
		}
		else if(crossRes.y < 0 && 
			Vector3.Angle(A.forward, B.position - A.position) <= 20)
		{
			print("左侧20度，发现入侵者");
		}
	}
}
```

#### 向量插值运算
##### 线性插值
==用于跟随运动，摄像机跟随==

`Vector3.Lerp(start, end, t)`
对两个点进行插值计算，t的取值范围0~1

公式：==result = start + (end - start) * t==

线性插值的应用：
* **每帧改变start的值，位置无限接近，但不会得到end位置（先快后慢）**
* **每帧改变t的值，当t>=1时得到结果（匀速）**
注意：当t>=1时改变目标位置，会直接瞬移到目标位置，所以目标位置改变后需要清空t

``` c#
Transform target_;
Transform A_;
Transform B_;

//记录B位置、时间
Vector3 startPos_;
float time_;
//记录目标位置
Vector3 targetNowPos_;

void Start()
{
	startPos_ = B_.position;
	targetNowPos_ = target_.position;
}

void Update()
{
	//每帧改变start的值，位置无限接近，但不会得到end位置（先快后慢）
	A_.position = Vector3.Lerp(A_.position, target_.position, Time.deltaTime);
	
	//每帧改变t的值，当t>=1时得到结果（匀速）
	//注意：当t>=1时改变目标位置，会直接瞬移到目标位置，所以目标位置改变后需要清空t
	if(targetNowPos_ != target_.position)
	{
		targetNowPos_ = target_.position;
		time = 0;
		startPos_ = B_.position;
	}
	time += Time.deltaTime;
	B.position = Vector3.Lerp(startPos_, targetNowPos_, time);
}

```

##### 球形插值
==用于曲线运动，模拟太阳运动弧线、导弹运动弧线==

`Vector3.Slerp(start, end, t)`
对两个向量进行插值计算，t的取值范围0~1

线性插值和球形插值的区别：
线性：直接平移，直线轨迹
球形：直接旋转，弧形轨迹

``` c#
Transform A_;

float time_;
void Update()
{
	time_ += Time.deltaTime;
	A_.position = Vector3.Slerp(Vector.right * 10, Vector3.forward * 10, time_);
}
```

##### 练习
1. 用线性插值相关知识，实现摄像机跟随（不能将摄像机设置为对象子物体），摄像机一直在物体的后方4米，向上偏7米的高度
``` c#
float zOffset = 4;
float yOffset = 7;

Transform target;
Vector3 targetPos;

float moveSpeed;

void Start()
{
	
}

void Update()
{
	if(targetPos != target.position + -target.forward * zOffset * target.up * yOffset)
	{
		targetPos = target.position + -target.forward * zOffset * target.up * yOffset;
	}
	
	this.transform.position = Vector3.Lerp(this.transform.position, 
		targetPos, Time.deltaTime * moveSpeed);
	
	this.transform.LookAt(target);
}
```

2. 通过球形插值模拟太阳的升降变化
``` c#
Transform A;

void Update()
{
	time += Time.deltaTime;
	A.position = Vector3.Slerp(Vector3.right * 10, 
		Vector3.left * 10 + Vector3.up * 0.1f, time * 0.01f);
}
```


### 3D数学--四元数 Quaternion
#### 为何使用四元数
##### 1. 欧拉角
由三个角度(x, y, z)组成，在特定坐标系下用于描述物体的旋转量

空间中的任意旋转都可以分解成绕
三个互相垂直轴的三个旋转角组成的序列

##### 欧拉角旋转约定
heading - pitch - bank
是一种最常用的旋转序列约定

Y-X-Z约定
heading	 ：物体绕自身的对象坐标系的==Y轴==，旋转的角度
pitch		：物体绕自身的对象坐标系的==X轴==，旋转的角度
bank	   ：物体绕自身的对象坐标系的==Z轴==，旋转的角度

##### Unity中的欧拉角
==Inspector窗口==中的Rotation就是欧拉角

`this.transform.eulerAngles`就是欧拉角角度

##### 欧拉角的优缺点
优点：
直观、易理解
存储空间小（三个数表示）
可以进行从一个方法到另一个方向旋转大于180°的角度

缺点：
==同一旋转的表示不唯一==
==万向节死锁==

##### 2. 万向节死锁
当某个特定轴达到某个特殊值
==绕一个轴旋转可能会覆盖住另一个轴的旋转==，==从而失去一维自由度==

==Unity中X轴达到90°时，会产生万向节死锁==

##### 总结
因为欧拉角存在一些缺点
==1.同一旋转的表示不唯一（90°和450°）==
==2.万向节死锁==

而四元数旋转不存在万向节死锁问题
因此在计算机中往往使用四元数来表示三维空间中的旋转信息

#### 四元数是什么
##### 四元数的构成
四元数是简单的超复数
由实数加上三个虚数单位组成
主要用于在三维空间中表示旋转

四元数原理包含大量数学相关知识，较为复杂，比如：复数、四维空间等
因此此处我们只对其基本构成和基本公式进行讲解，想深入可以从数学层面了解

一个四元数包含一个标量和一个3D向量 [ w, v ]，w为标量，v为3D向量
[ w, (x, y, z) ]

对于给点的任意一个四元数：==表示3D空间中的一个旋转量==

**轴-角对**
在3D空间中，任意旋转都可以表示 绕着某个轴旋转一个旋转角得到

==注意：该轴并不是空间中的x，y，z轴，而是任意一个轴==

对于给定旋转，假设为绕着n轴，旋转β°，n轴为(x, y, z)，
那么可以构成四元数为
**四元数Q = [cos(β / 2), sin(β / 2)n]**
==**四元数Q = [cos(β / 2), sin(β / 2)x,  sin(β / 2)y,  sin(β / 2)z]**==

四元数Q则表示绕着轴n，旋转β°的旋转量

##### Unity中的四元数
`Quaternion`是Unity中表示四元数的结构体

##### Unity中的四元数初始化方法
轴角对公式初始化
==四元数Q = [cos(β / 2), sin(β / 2)x,  sin(β / 2)y,  sin(β / 2)z]==
``` c#
Quaternion q = new Quaternion(sin(β/2)x,sin(β/2)y,sin(β/2)z, cos(β/2));
//使用较少
```

轴角对方法初始化
四元数 Q = Quaternion.AngleAxis(角度, 轴)
``` c#
Quaternion q = Quaternion.AngleAxis(60, Vector3.right);
//一般使用该方法
```

``` c#
//使用较少
Quaternion q = new Quaternion(Mathf.Sin(30 * Mathf.Deg2Rad), 0, 0, 
	Mathf.Cos(30 * Mathf.Deg2Rad));

Quaternion q = Quaternion.AngleAxis(60, Vector3.right);

this.transform.rotation = q;
```

##### Unity中的四元数和欧拉角的相互转换
``` c#
//1. 欧拉角转四元数
Quaternion q = Quaternion.Euler(60, 0, 0);

//2. 四元数转欧拉角
q.eulerAngles
```

##### 四元数弥补欧拉角的缺点
==四元数相乘代表旋转四元数==

1. **同一旋转的表示不唯一**
四元数旋转后，转换后的欧拉角始终是 -180~180°

2. **万向节死锁**
通过四元数旋转对象可以避免万向节死锁

``` c#
void Update()
{
	//Vector3.up是自己坐标系的
	this.transform.rotation *= Quaternion.AngleAxis(1, Vector3.up);
}
```

#### 四元数常用方法
##### 单位四元数
`Quaternion.identity`

==单位四元数表示没有旋转量（角位移）==
当角度为0或360°时，对于给定轴都会得到单位四元数

[1, (0,0,0)] 和 [-1, (0,0,0)] 都是单位四元数，表示没有旋转量

``` c#
//可以用来实例化游戏对象
Instantiate(gameObject, Vector3.zero, Quaternion.identity);
```

##### 插值运算
`Quaternion.Lerp`
`Quaternion.Slerp`

四元数的Lerp和Slerp只有一些细微差别，Slerp的效果会好一点

==Lerp的效果相比Slerp更快，但如果旋转范围比较大，效果就会比较差==
==所以建议使用Slerp进行插值运算==

``` c#
Transform target;
Transform A;
Transform B;

Quaternion startRotation;
Quaternion targetRotation;
float time;
void Update()
{
	//先快后慢
	A.rotation = Quaternion.Slerp(A.rotation, target.rotation, Time.deltaTime);
	
	//匀速
	if(targetRotation != target.rotation)
	{
		targetRotation = target.rotation;
		startRotation = B.rotation;
		time = 0;
	}
	time += Time.deltaTime;
	B.rotation = Quaternion.Slerp(startRotation, targetRotation, time);
}
```

##### 向量方向 转换为 对应的四元数角度
`Quaternion.LookRotation(面朝向量)`

LookRotation方法可以将传入的面朝向量 转换为对应的 四元数角度信息

>例如：
当人物的面朝向想要改变时，只需要把目标面朝下传入LookRotation方法，便可以得到目标四元数角度信息，之后将人物四元数角度信息改为目标的信息即可达到转向

``` c#
Transform target;

void Update()
{
	this.transform.ratation = 
		Quartenion.LookRotation(target.position - this.transform.position);
}
```

##### 练习
1. 利用四元数的LookRotation方法，实现LookAt的效果
``` c#
//扩展方法
static class Tools
{
	public static void MyLookAt(this transform obj, Transform target)
	{
		Vector3 vec = target.position - obj.position;
		obj.rotation = Quaternion.LookRotation(vec);
	}
}

//调用
void Update()
{
	this.transform.MyLookAt(target);
}
```

2. 将之前的摄像机移动的练习题中的LookAt换成LookRotation实现并且通过Slerp来缓慢看向玩家
``` c#
float zOffset = 4;
float yOffset = 7;

Transform target;
Vector3 targetPos;

float moveSpeed;

Quartenion targetQuartenion;

float roundTime;
void Start()
{
	
}

void Update()
{
	if(targetPos != target.position + -target.forward * zOffset * target.up * yOffset)
	{
		targetPos = target.position + -target.forward * zOffset * target.up * yOffset;
	}
	
	this.transform.position = Vector3.Lerp(this.transform.position, 
		targetPos, Time.deltaTime * moveSpeed);
	
	//this.transform.LookAt(target);
	
	//先快后慢
	targetQuartenion = Quartenion.LookRotation(target.position - 
		this.transform.position);
	this.transform.rotation = Quartenion.Slerp(this.transform.rotation, 
		targetQuartenion, Time.deltaTime);
	
	//匀速
	if(targetQuartenion != Quartenion.LookRotation(target.position - 
		this.transform.position))
	{
		targetQuartenion = Quartenion.LookRotation(target.position - 
		this.transform.position);
		startQuartenion = this.transform.rotation;
		roundTIme = 0;
	}
	roundTime += Time.deltaTime;
	this.transform.rotation = Quartenion.Slerp(startQuartenion, targetQuartenion, 
		roundTime);
}
```

#### 四元数计算
##### 四元数相乘
==两个四元数相乘得到一个新的四元数==
==代表两个旋转量的叠加==
相当于==旋转==

==注意：旋转相对的坐标系是物体自身坐标系==

``` c#
Quaternion q1 = Quaternion.AngleAxis(20, Vector3.up);
this.transform.rotation *= q1;
```

##### 四元数乘向量
四元数乘向量得到一个新的向量
==可以将指定的向量旋转 对应的四元数的旋转量==
相当于==旋转向量==

``` c#
Vector3 v = Vector3.forward;

v = Quaternion.AngleAxis(45, Vector.up) * v;
//！！！不能写成这样，会报错！！！
//v = v * Quaternion.AngleAxis(45, Vector.up);
```

##### 练习
1. 模拟飞机发射不同类型子弹的方法。
	* 单发
	* 双发
	* 扇形
	* 环形

Player.cs
``` c#
enum E_FireType
{
	one,
	two,
	sector,
	round
}

class Player: MonoBehavior
{
	E_FireType type = E_FireType.one;
	
	public GameObject bullect;
	
	public int roundNum = 4;
	
	void Start()
	{
		
	}
	
	void Update()
	{
		if(Input.GetKeyDown(KeyCode.Alpha1))
		{
			type = E_FireType.one;
		}
		else if(Input.GetKeyDown(KeyCode.Alpha2))
		{
			type = E_FireType.two;
		}
		else if(Input.GetKeyDown(KeyCode.Alpha3))
		{
			type = E_FireType.sector;
		}
		else if(Input.GetKeyDown(KeyCode.Alpha4))
		{
			type = E_FireType.round;
		}
		
		if(Input.GetKeyDown(KeyCode.Space))
		{
			Fire();
		}
	}
	
	void Fire()
	{
		switch(type)
		{
			case E_FireType.one:
			{
				Instantiate(bullect, this.transform.position, this.transform.rotation);
			}
			break;
			
			case E_FireType.two:
			{
				//往中心点左边平移
				Instantiate(bullect, 
					this.transform.position - this.transform.right * 0.5f,
					this.transform.rotation);
				//往中心点右边平移
				Instantiate(bullect, 
					this.transform.position + this.transform.right * 0.5f,
					this.transform.rotation);
			}
			break;
			
			case E_FireType.sector:
			{
				//往左旋转20°
				Instantiate(bullect, this.transform.position, 
					this.transform.rotation * Quaternion.AngleAxis(-20, Vector3.up));
			
				//中间
				Instantiate(bullect, this.transform.position, this.transform.rotation);
				
				//往右旋转20°
				Instantiate(bullect, this.transform.position, 
					this.transform.rotation * Quaternion.AngleAxis(20, Vector3.up));
			}
			break;
			
			case E_FireType.round:
			{
				float angel = 360 / roundNum;
				for(int i=0; i<roundNum; ++i)
				{
					Instantiate(bullect, this.transform.position, 
						this.transform.rotation * 
							Quaternion.AngleAxis(i * angle, Vector3.up));
				}
			}
			break;
		}
	}
}
```

Bullect.cs
``` c#

class Bullect: MonoBehavior
{
	public float moveSpeed = 10;
	
	public float duration = 5;

	void Start()
	{
		Destroy(this.gameObject, duration);
	}
	
	void Update()
	{
		this.transform.Translate(Vector3.forward * moveSpeed * Time.deltaTime);
	}
}

```

2. 实现摄像机跟随效果
	* 摄像机在人物斜后方，通过角度控制倾斜率
	* 通过鼠标滚轮可以控制摄像机距离人物的距离（有最大最小限制）
	* 摄像机看向人物头顶上方一个位置（可调节）
	* Vector3.Lerp实现相机跟随人物
	* Quaternion.Slerp实现相机朝向过渡效果
``` c#
class CameraMove : MonoBehavior
{
	public Transform target;
	
	//头顶正上方的偏移位置
	public float headOffsetH = 1;
	
	//倾斜角度
	public float angle = 45;

	//摄像机离观测点的距离
	public float dis = 5;
	public float minDis = 3;
	public float maxDis = 10;

	//当前摄像机的位置
	Vector3 nowPos;
	
	Vector3 nowDir;

	void Start()
	{
		
	}
	
	void Update()
	{
		//通过鼠标滚轮可以控制摄像机距离人物的距离
		dis += Input.GetAxis("Mouse ScrollWheel");
		dis = Mathf.Clamp(dis, minDis, maxDis);
	
		//向头顶偏移位置
		nowPos = target.position + target.up * headOffsetH;
	
		//往后方偏移角度
		nowDir = Quaternion.AngleAxis(angle, target.right) * -target.forward
		nowPos += nowDir;
		//偏移位置
		nowPos *= dis;
		
		//摄像机跟随人物
		this.transform.position = Vector3.Lerp(this.transform.position,
			nowPos, Time.deltaTime);
		
		//摄像机看向人物
		this.transform.rotation = Quaternion.Slerp(this.transform.rotation, 
			Quaternion.LookRotation(-nowDir), Time.deltaTime);
		
	}
}
```

### MonoBehavior重要内容
#### 延迟函数
##### 知识点一 什么是延迟函数
会延时执行的函数
是MonoBehavior提供的方法

##### 知识点二 延迟函数的使用
1. **延迟函数**
`Invoke`
参数一：函数名字符串
参数二：延迟时间，秒为单位

注意：
* 延时函数参数一传入的是函数名字符串
* ==被延时的函数无法传入参数==，可以使用无参函数调用被延时的函数
* 函数名必须是该脚本上声明的函数

2. **延迟重复执行函数**
`InvokeRepeating`
参数一：函数名字符串
参数二：第一次执行的延迟时间
参数三：之后每次执行的间隔时间

注意：
* 延时函数参数一传入的是函数名字符串
* ==被延时的函数无法传入参数==，可以使用无参函数调用被延时的函数
* 函数名必须是该脚本上声明的函数

3. **取消延迟函数**
	* 取消该脚本的所有延迟函数执行
	`CancelInvoke()`

	* 取消指定的延迟函数执行
	`CancelInvoke("函数名")`
	==只要是同名的函数，都会取消==
	==找不到函数也不会报错==

4. **判断是否有延迟函数**
	`if( IsInvoking() ) {}`
	`if( IsInvoking("函数名") ) {}`

##### 知识点三 延迟函数受对象失活和销毁的影响
==脚本依附对象失活或脚本失活，延迟函数可以继续执行==
==脚本依附对象销毁或脚本移除，延迟函数无法继续执行==

##### 练习
1. 计时器
方法一：InvokeRepeating
``` c#
int time = 0;

void Start()
{
	InvokeRepeating("DelayFunc", 0, 1);
}

void DelayFunc()
{
	time += 1;
}
```

方法二：Invoke
``` c#

int time = 0;

void Start()
{
	DelayFunc();
}

void DelayFunc()
{
	time += 1;
	Invoke("DelayFunc", 1);
}

```

2. 用两种方式延时销毁一个指定对象
``` c#
//方法一：Destroy
Destroy(this.gameObject, 5);

//方法二：Invoke
Invoke("DelayDestroy", 5);

void DelayDestroy()
{
	Destroy(this.gameObject);
}
```


#### 协程
##### 知识点一 Unity是否支持多线程
Unity支持多线程
==但是新开的线程无法访问Unity的相关对象的内容==，会报错

==注意：Unity中的多线程 要记得关闭==

**实际开发中，可以将复杂计算放入线程中**
**可以用线程处理可能会卡顿的场合中**

##### 知识点二 协程是什么
主要作用：
将代码分时执行，不卡主线程
简单理解，是把可能会让主线程卡顿的耗时逻辑分时分步执行

**主要使用场景**：
* ==异步加载文件==
* ==异步下载文件==
* ==场景异步加载==
* ==批量创建时防止卡顿==

##### 知识点三 协程和线程的区别
新开的线程是独立的管道，和主线程并行执行
新开的协程是在原线程上开启，进行逻辑分时分步执行

##### 知识点四 协程的使用
继承MonoBehavior的类都可以开启协程函数

1. **声明协程函数**
协程函数2个关键点
	* 返回值为`IEnumerator`类型及其子类
	* 函数通过`yield return 返回值;` 进行返回

``` c#
IEnumerator MyCoroutine(int i, string str)
{
	print(i);
	//等待5秒
	yield return new WaitForSeconds(5f);
	print(str);
}
```

2. **开启协程函数**
	* 常用开启方式

``` c#
//第一种方法
StartCoroutine(MyCoroutine(1, "str"));

//第二种方法
IEnumerator ie = MyCoroutine(1, "str");
StartCoroutine(ie);
```

3. **关闭协程**
	* 关闭所有协程
	 `StopCoroutinues()`

	* 关闭指定协程
``` c#
Coroutine c1 = StartCoroutine(MyCoroutine(1, "str"));
Coroutine c2 = StartCoroutine(MyCoroutine(1, "str"));
Coroutine c3 = StartCoroutine(MyCoroutine(1, "str"));

//关闭指定协程
StopCoroutine(c1);
```

##### 知识点五  yield return 不同内容的含义
1. **下一帧执行**
在`Update`和`LateUpdate`之间执行
``` c#
yield return 数字;

yield return null;
```

2. **等待指定秒后执行**
在`Update`和`LateUpdate`之间执行
``` c#
yield return new WaitForSeconds(秒);
```

3. **等待下一个固定物理帧更新时执行**
在`FixedUpdate`和`碰撞检测相关函数`之后执行
做==截图功能==时会使用
``` c#
yield return new WaitForFixedUpdate();
```

4. **等待摄像机和GUI渲染完成后执行**
在`LateUpdate`之后的渲染相关处理完毕后执行
``` c#
yield return new WaitForEndOfFrame();
```

5. **一些特殊类型对象，比如异步加载相关函数返回的对象**
一般在`Update`和`LateUpdate`之间执行
异步加载资源
异步加载场景
网络加载

6. **跳出协程**
``` c#
yield break;
```


##### 知识点六 协程受对象和脚本组件失活和销毁的影响
协程开启后，
物体和脚本组件销毁后，协程不执行
物体失活后，协程不执行
==脚本组件失活后，协程继续执行==

##### 练习
1. 使用协程制作一个计时器

``` c#
IEnumerator Clock()
{
	int time = 0;

	while(true)
	{
		++time;
		yield return new WaitForSeconds(1f);
	}
}

StartCoroutine(Clock());
```

2. 请在场景中创建十万个随机位置的立方体，让其不会明显卡顿

``` c#
IEnumerator CreateCube(int num)
{
	for(int i=0; i<num; ++i)
	{
		GameObject obj = GameObject.CreatePrimitive(Primitive.Cube);
		obj.transform.position = 
			new Vector3(Random.Range(-100,100), Random.Range(-100,100),
				Random.Range(-100,100));
		if(i % 1000 == 0)
		{
			yield return null;
		}
	}
}
```

#### 协程原理
##### 知识点一 协程的本质
协程可以分成两部分
1. ==协程函数本体==
2. ==协程调度器==

协程本体就是一个能够中间暂停返回的函数
协程调度器是Unity内部实现的，会在对应的时机继续执行协程函数

Unity只实现了协程调度部分
协程的本体本质上就是一个C#迭代器方法

##### 知识点二 协程本体是迭代器方法的体现
1. **协程函数本体**
我们也可以自己执行迭代器函数内容
``` c#
IEnumerator ie = MyCoroutineFunc();

//会执行遇到下一个yield return为止的代码逻辑
ie.MoveNext();

//得到 yield return 返回的对象
ie.Current;


while(ie.MoveNext())
{
	ie.Current;
}

```

2. **协程调度器**
利用迭代器函数返回的内容来进行之后的处理
比如Unity的协程调度器，==根据yield return返回的内容==决定了下一个在何时继续执行迭代器函数的下一个部分

##### 练习
请不使用Unity自带的协程调度器开启协程，
通过迭代器函数实现每隔一秒执行函数中的一部分逻辑

``` c#
IEnumerator MyCoroutineFunc()
{
	int time = 0;
	while(true)
	{
		print(time);
		yield return 1;
		++time;
	}
}


class YieldReturnTime
{
	//记录下次执行的迭代器接口
	public IEnumerator ie_;
	//记录下次执行的时间
	public float time_;
}

class CoroutineMgr : MonoBehavior
{
	CoroutineMgr instance_;
	public CoroutineMgr Instance => instance_;
	
	//存储迭代器的容器
	List<YieldReturnTime> list = new List<YieldReturnTime>();
	
	void Awake()
	{
		instance_ = this;
	}
	
	void Update()
	{
		//为了避免在循环时 从列表里移除内容 我们可以倒着遍历
		for(int i=list.Count-1; i>=0; --i)
		{
			//到达时间
			if(list[i].time_ <= Time.time)
			{
				if(list[i].ie_.MoveNext())
				{
					if(list[i].ie_.Current is int)
					{
						//下一次执行的时间
						list[i].time_ = Time.time + (int)list[i].ie_.Current;
					}
					else
					{
						//不是int型则从列表中移除
						list.RemoveAt(i);
						//加到别的列表
						.......
					}
				}
				else
				{
					//从列表中移除
					list.RemoveAt(i);
				}
			}
		}
	}
	
	public void MyStartCoroutine(IEnumerator ie)
	{
		if(ie.MoveNext())
		{
			if(ie.Current is int)
			{
				YieldReturnTime y = new YieldRetuenTime();
				y.ie_ = ie;
				//下一次执行的时间
				y.time_ = Time.time + (int)ie.Current;
				
				list.Add(y);
			}
		}
	}
}

```


### Resources资源动态加载
#### 特殊文件夹
##### 知识点一 工程路径获取
注意：
==该方式获取的路径，一般情况下只在编辑模式下使用==
我们不会在实际发布后，还使用该路径
==游戏发布过后，该路径就不存在了==
``` c#
Application.dataPath

// 打印至Assets文件夹
// ..../.../.../Assets
```

##### 知识点二 Resources 资源文件夹
注意：
**需要在Assets手动创建Resources文件夹**
==Resources文件夹可以有多个==，也可以在Editor等文件夹里
Resources文件夹的内容都会打包在一起，会自动整合
**`Resources.Load()`重复加载同一资源时，不会浪费内存，只会浪费性能**。（因为同一资源已经加载到内存中了，所以不会浪费内存，只是每次重复加载时都会去寻找，所以浪费性能）

路径获取：
==一般不获取路径，只能使用`Resources`相关API进行加载==

作用：
* 通过`Resources`相关API动态加载的资源需要放在其中
* 该文件夹下的所有文件都会被打包出去
* 打包时Unity会对其压缩加密
* 该文件夹打包后只读，只能通过`Resources`相关API加载

##### 知识点三 StreamingAssets 流动资源文件夹
注意：
**需要在Assets手动创建Resources文件夹**

路径获取：
``` c#
Application.StreamingAssetsPath
```

作用：
* 打包出去不会被压缩加密，可以任意操作
* 在移动平台只读，在PC平台可读可写
* 可以放入一些需要自定义动态加载的初始资源

##### 知识点四 persistentDataPath 持久数据文件夹
注意：
==不需要手动创建==
不在Assets文件夹中

路径获取：
``` c#
Application.persistentDataPath 
```

作用：
* 所有平台可读可写
* 一般用于放置动态下载或动态创建的文件，游戏中创建或获取的文件都放在其中

##### 知识点五 Plugins 插件文件夹
注意：
**需要在Assets手动创建**

路径获取：
一般不获取

作用：
* 不同平台的插件相关文件放入其中，例如Android和IOS

##### 知识点六 Editor 编辑器文件夹
注意：
**需要在Assets手动创建**

路径获取：
一般不获取
如果要获取，可以用工程路径拼接
`Application.dataPath + /Editor`

作用：
* 开发Unity编辑器时，编辑器相关脚本放入其中
* 该文件夹的内容不会打包出去

##### 知识点七 StandardAssets 默认资源文件夹
注意：
**需要在Assets手动创建**

路径获取：
一般不获取

作用：
* 一般Unity自带资源都放在该文件夹下
* 代码和资源优先被编译

#### Resources资源同步加载
##### 知识点一 Resources资源动态加载的作用
1. 通过代码动态加载Resources文件夹下指定路径资源
2. 避免繁琐的拖拽操作

##### 知识点二 常用资源类型
1. 预设体对象——GameObject
2. 音效文件——AudioClip
3. 文本文件——TextAsset
4. 图片文件——Texture
5. 其他类型——动画文件、模型等

注意：
==预设体对象加载需要实例化==
==其他资源加载一般直接用==

##### 知识点三 资源同步加载 普通方法
1. 预设体对象
创建在场景上，需要实例化

* 第一步：**加载预设体的资源文件**
==本质上是加载配置数据在文件中==
==不用加后缀==
``` c#
//返回Unity的Object
Object obj = Resources.Load("文件夹名/资源名");
```

* 第二步：**加载配置文件后，实例化**
``` c#
Instantiate(obj);
```


2. 音效资源
``` c#
public AudioSource audioSource;

Object obj = Resources.Load("文件夹名/音效文件名");

//相当于把音效文件拖拽到AudioSource组件里
audioSource.clip = obj as AudioClip;
```

3. 文本资源
支持的格式：
.txt
.xml
.bytes
.json
.html
.csv
...

``` c#
Object obj = Resources.Load("文件夹名/文本文件名");

TextAsset ta = obj as TextAsset;

//文本内容
ta.text;

//字节数据组
ta.bytes;
```

4. 图片
``` c#
Teture tex = Resources.Load("文件夹名/图片文件名") as Texture;

void OnGUI()
{
	GUI.DrawTexture(new Rect(0,0,100,100), tex);
}
```

5. 其他类型

6. 资源同名怎么处理？
`Resources.Load()`加载同名资源时，无法准确加载出想要的内容

可以使用其他API：
* 加载指定类型的资源
``` c#
//图片类型
Resources.Load("路径/文件名", typeof(Texture));
```

* 加载指定名字的所有资源
用得较少
``` c#
Object[] objs = Resources.LoadAll("路径/文件名");

foreach(Object item in objs)
{
	if(item is Texture)
	{
		
	}
	else if
}
```

##### 知识点四 资源同步加载 泛型方法

``` c#
Texture tex = Resources.Load<Texture>("路径/文件名");
```

#### Resources资源异步加载
##### 知识点一 Resources异步加载是什么
Resources同步加载中如果加载过大的资源可能会造成程序卡顿（从硬盘读取到内存，是需要计算的）
越大的资源耗时越长，就会造成掉帧卡顿

Resources异步加载就是内部新开一个线程进行资源加载，不会造成主线程卡顿

##### 知识点二 Resources加载方法
注意：
异步加载不能马上得到资源，==至少要等一帧==

1. **通过异步加载中的完成事件监听，使用加载的资源**
``` c#
Resources.LoadAsync("路径/文件名");

Texture tex_;
//加载结束
void LoadOver(AsyncOperation ao)
{
	tex_ = (ao as ResourceRequest).asset as Texture;
}

ResourceRequest rq = Resources.LoadAsync<Texture>("路径/文件名");
//事件监听
rq.completed += LoadOver;

void OnGUI()
{
	if(tex_!=null)
	{
		//操作
		...
	}
}
```

2. **通过协程，使用加载的资源**
``` c#
Texture tex_;

IEnumerator Load()
{
	ResourceRequest rq = Resources.LoadAsync<Texture>("路径/文件名");
	//方法一：
	{
		//unity会判断资源是否加载完毕
		yield return rq;
		//资源加载完毕
		tex_ = rq.asset as Teture;
	}
	
	//方法二：
	{
		//判断加载是否结束
		while( !rq.isDone )
		{
			//加载进度（不会特别准确）
			print(rq.priority);
			yield return null;
		}
		//资源加载完毕
		tex_ = rq.asset as Teture;
	}
}

StartCroutine(Load());
```

##### 总结
1. 完成事件监听异步加载
优点：写法简单
缺点：只能在资源加载结束后进行处理
”==线性加载==“

2. 协程异步加载
优点：可以在协程中处理复杂逻辑，比如同时加载多个资源，比如进度条更新
缺点：写法稍微复杂
“==并行加载==”

##### 练习
写一个简单的资源管理器，提供统一的方法给外部用于资源异步加载，外部可以传入委托，用于当资源加载结束时使用资源

``` c#
class ResourcesMgr
{
	static ResourcesMgr instance_ = new ResourceMgr();
	public static ResourcesMgr Instance => instance_;
	
	ResourcesMgr() {}
	
	public void LoadRes<T>(string name, UnityAction<T> callback) where T:Object
	{
		ResourceRequest rq = Resources.LoadAsync<T>(name);
		rq.completed += (ao)=>
		{
			callback((ao as ResourceRequest).asset as T);
		};
	}
}


//调用
Texture tex_;
ResourceMgr.Instance.LoadRes<Texture>("路径/文件名", (obj)=>{
	tex_ = obj;
});
```

#### Resources资源卸载
##### 知识点一 Resources资源重复加载会浪费内存吗
Resource资源加载一次后，就会一直存放在内存中作为缓存
第二次加载时发现缓存中存在该资源，会直接取出来使用

所以多次重复加载资源不会浪费内存，但是会浪费性能（需要去查找）

##### 知识点二 如何手动释放掉缓存中的资源
1. **卸载指定资源**
`Resources.UnloadAsset(对象)`
注意：
==该方法不能释放`GameObject`对象，因为会用于实例化对象。即使是还没实例化的对象，也不能卸载。都会报错==
只能用于一些不需要实例化的内容，比如文本、音效、图片等
一般情况下，很少单独使用

2. **卸载未使用的资源**
注意：
一般在过场时和GC一起使用，或内存到达瓶颈时使用

``` c#
Resources.UnloadUnusedAssets();
GC.Collect();
```

### 场景异步切换
##### 知识点一 场景同步切换
``` c#
SceneManager.LoadScene("场景名");
```

1. **场景同步切换的缺点**
==在切换场景时，Unity会删除当前场景上的所有对象，并且去加载下一个场景的相关信息。==
如果当前场景的对象过多或下一个场景的对象过多，这个过程会非常耗时，让玩家感受到卡顿
异步可以解决这个问题

##### 知识点二 场景异步切换
1. **通过事件回调函数异步加载**
``` c#
AsyncOperation ao = SceneManager.LoadSceneAsync("场景名");

//加载结束回调
ao.completed += (ao)=>{
	
};
```

2. **通过协程异步加载**
注意：加载场景会把当前场景上没有特别处理的对象都删除，所以协程中的部分逻辑，可能执行不了。
解决思路：让处理场景加载脚本挂载的对象，过场景时不被删除
``` c#
IEnumerator LoadScene(string name)
{
	AsyncOperation ao = SceneManager.LoadSceneAsync(name);
	
	//进度条
	//第一种：
	{
		//进度不准确，一般不会用
		while(!ao.isDone)
		{
			print(ao.progress);
			yield return null;
		}
		
		//立刻循环后，填满进度条
	}
	
	//第二种：
	//自己定义规则
	{
		//加载场景结束，进度条更新20%
		
		//加载场景中的怪物等，进度条更新20%
		
		//...
		
		//加载结束，进度条填满
	}
	
	yield return ao;
	
	//只有调用DontDestroyOnLoad(this.gameObject)才能执行到这里
	print("异步加载结束");
}

//调用
DontDestroyOnLoad(this.gameObject);
StartCroutine(LoadScene(name));
```

##### 练习
写一个简单的场景管理器，提供统一的方法给外部用于场景异步切换，外部可以传入委托，用于当场景加载结束时执行某些逻辑
``` c#
class MySceneMgr
{
	static MySceneMgr instance_ = new MySceneMgr();
	public MySceneMgr Instance => instance_;
	
	MySceneMgr() {}
	
	public void LoadScene(string name, UnityAction callback)
	{
		AsyncOperation ao = SceneManager.LoadSceneAsync(name);
		
		ao.completed += (ao)=>{
			callback();
		};
	}
}
```

### LineRenderer
#### 知识点一 LineRenderer是什么
LineRenderer是Unity提供的一个用于画线的组件
可以在场景中绘制线段

一般用于：
1. 绘制攻击范围
2. 武器红外线
3. 辅助功能
4. 其他画线功能

#### 知识点二 LineRenderer参数相关
1. **Loop**
是否终点起始自动相连

2. **Positions**
线段的点

3. **线段宽度曲线调整**

4. **Color**
颜色变化

5. **Corner Vertices**
（角定点，圆角）
此属性指示在一条线中绘制角时使用了多少额外的点。增加此值，使线角看起来更加圆

6. **End Cap Vertices**
（终端顶点，圆角）
终点圆角

7. **Use World Space**
是否使用世界坐标系

8. **Material**
线使用的材质球

#### 知识点三 LineRenderer代码相关
``` c#
//动态添加一个线段
GameObject line = new GameObject();
line.name = "Line";

LineRenderer lr = line.AddComponent<LineRenderer>();

//首尾相连
lr.loop = true;

//开始结束宽
lr.startWidth = 0.02f;
lr.endWidth = 0.02f;

//开始结束颜色
lr.startColor = Color.white;
lr.endColor = Color.green;

//设置材质
Material m = Resources.Load<Material>("...");
lr.material = m;

//设置点
// 1.一定要先设置点的个数
lr.positionCount = 4;
// 2.设置每个对应点的位置
//	不设置的话默认是0, 0, 0
lr.SetPositions(new Vector3[] {
	new Vector3(0, 0, 0),
	new Vector3(0 ,0, 1),
	new Vector3(0, 0, 2)
});
//设置第几个点的位置
lr.SetPositions(3, new Vector3(0, 0, 2));

//是否使用世界坐标系（决定了线段是否随对象移动而移动）
lr.useWorldSpace = false;//随着对象移动

//是否受光照影响，会接受光数据，进行着色器计算
lr.generateLighingData = true;
```

#### 练习
1. 写一个方法，传入中心点，传入一个半径，用LineRenderer画一个圆
``` c#
void DrawLineRenderer(Vector3 centerPos, float r, int pointNum)
{
	GameObject obj = new GameObject();
	obj.name = "Circle";
	LineRenderer lr = obj.AddComponent<LineRenderer>();
	lr.loop = true;
	lr.positionCount = pointNum;

	//每个点间隔的度数
	float angle = 360f / pointNum;
	
	//得到每一个点
	for(int i=0; i<=pointNum; ++i)
	{
		//半径上的点（偏移中心点）
		Vector3 radius_pos = centerPos + Vector3.forward * r;
		
		//旋转点
		radius_pos = Quaternion.AngleAxis(angle * i, Vector3.up);
		
		lr.SetPosition(i, radius_pos);
	}
}
```

2. 请实现在Game窗口长按鼠标，用LineRenderer画出鼠标移动的轨迹
``` c#
LineRenderer lr;

Vector3 mousePos;

void Start()
{
	
}

void Update()
{
	if(Input.GetMouseButtonDown(0))
	{
		GameObject obj = new GameObject();
		lr = obj.AddComponent<LineRenderer>();
		lr.loop = false;
		lr.startWidth = 0.5f;
		lr.endWidth = 0.5f;
		lr.positionCount = 0;
	}

	if(Input.GetMouseButton(0))
	{
		lr.positionCount += 1;
		
		//得到鼠标坐标
		mousePos = Input.mousePosition;
		mousePos.z = 10;
		
		//鼠标转世界坐标
		Vector3 worldPos = Camera.main.ScreenToWorldPoint(mousePos);
		
		lr.SetPosition(lr.positionCount - 1, worldPos);
	}
}
```

### 物理系统之范围检测
#### 知识点一 什么是范围检测
游戏中瞬时的攻击范围判断一般会会使用范围检测

举例：
1. 玩家在前方5米处释放一个地刺魔法，在此范围的对象都会收到范围伤害
2. 玩家攻击，在前方1米原型范围内对象都收到伤害
等

类似==这种没有实体，只想要检测在指定某一范围的逻辑==，就可以使用范围检测

#### 知识点二 任何进行范围检测
* 必备条件：
==想要被范围检测到的对象，必须具备碰撞器==

注意：
1. 范围检测API，只有当执行该句代码时，才进行一次范围检测，是==瞬时==的
2. 范围检测API，并不会真正产生一个碰撞器，只是碰撞判断计算而已

范围检测API
1. **盒状范围检测**
参数一：立方体中心点
参数二：立方体三边大小
参数三：立方体角度Quaternion
参数四：检测指定层级（默认检测所有层）
参数五：是否忽略触发器（默认使用UseGlobal）
	* UseGlobal：使用全局设置
	* Collide：检测触发器
	* Ignore：忽略触发器
返回值：在该范围内的触发器

``` c#
Collider[] colliders = Physics.OverlapBox
(
	Vector3.zero, 
	new Vector3(1, 2, 3), 
	Quaternion.AngleAxis(45, Vector3.up),
	1 << LayerMask.NameToLayer("UI") | 1 << LayerMask.NameToLayer("Default"),
	QueryTriggerInteraction.UseGlobal
);
```

>！！！重要知识点：！！！
关于层级LayerMask
通过名字得到层级编号 `LayerMask.NameToLayer`
每一个层级编号，都是对应位为1的二进制数
==需要通过位运算，可以选择想要的检测层级==
好处是一个int就可以表示所有想要检测的层级信息
>层级编号共有32位，0~31
>是一个int数
>每一个编号代表的都是二进制的一位

另一个盒装检测API：
参数：传入一个数组进行存储
返回值：碰撞到的碰撞器数量
``` c#
Collider[] colliders = new Collider[]();
int num = Physics.OverlapBoxNonAlloc
(
	Vector3.zero,
	new Vector3(1, 2, 3),
	colliders,
	Quaternion.AngleAxis(45, Vector3.up),
	1 << LayerMask.NameToLayer("UI") | 1 << LayerMask.NameToLayer("Default"),
	QueryTriggerInteraction.UseGlobal
);
```

2. **球形范围检测**
参数一：中心点
参数二：球半径
参数三：检测指定层级
参数四：是否忽略触发器
返回值：在该范围内的触发器
``` c#
Collider[] colliders = Physics.OverlapSphere
(
	Vector3.zero,
	5,
	...,
	...
);
```

另一个球形检测API：
参数：传入一个数组进行存储
返回值：碰撞到的碰撞器数量
``` c#
Collider[] colliders = new Collider[]();
int num = Physics.OverlapSphereNonAlloc
(
);
```

3. **胶囊状范围检测**
==可以用两个中心点决定胶囊的倾斜角度==
参数一：半圆一中心点
参数二：半圆二中心点
参数三：半圆半径
参数四：层级
参数五：是否忽略触发器
``` c#
Collider[] colliders = Physics.OverlapCapsule
(
	Vector3.zero,
	Vector3.up,
	5,
	...,
	...
);
```

另一个胶囊状检测API：
参数：传入一个数组进行存储
返回值：碰撞到的碰撞器数量

#### 练习
世界坐标原点有一个立方体，WASD键可以控制其前后移动和旋转
请结合所学知识实现：
1. 按J键在立方体面朝前方1米处进行立方体范围检测
2. 按K键在立方体前面5米范围内进行胶囊范围检测
3. 按L键在立方体脚下为原点，半径10米内进行球形范围检测
``` c#
class Test : MonoBehavior
{
	public float moveSpeed;
	public float roundSpeed;
	
	void Start()
	{
	
	}
	
	void Update()
	{
		this.transform.Translate(Vector3.forward * Time.deltaTime * moveSpeed *
			Input.GetAxis("Vertical"));
		this.transform.Rotate(Vector3.up, roundSpeed * Time.deltaTime *
			Input.GetAxis("Horizontal"));
			
		if(Input.GetKeyDown(KeyCode.J))
		{
			Physics.OverlapBox(
				this.transform.position + this.transform.forward, 
				Vector3.one,
				this.transform.rotation,
				1 << LayerMask.NameToLayer("Monster"));
		}
		if(Input.GetKeyDown(KeyCode.K))
		{
			Physics.OverlapCapsule(
				this.transform.position,
				this.transform.position + this.transform.forward * 5,
				0.5f,
				1 << LayerMask.NameToLayer("Monster"));
		}
		if(Input.GetKeyDown(KeyCode.L))
		{
			Physics.OverlapSphere(
				this.transform.position,
				10,
				1 << LayerMask.NameToLayer("Monster"));
		}
	}
}
```

### 物理系统之射线检测
#### 知识点一 什么是射线检测
物理系统中，目前学习的物体相交判断：
1. 碰撞检测——必备条件：刚体、碰撞器
2. 范围检测——必备条件：碰撞器

如果是以下这些场景：
1. **鼠标选择场景的物体**
2. **FPS射击游戏（无弹道，不产生实际的子弹对象进行移动）**
等一些需要判断一条线和物体的碰撞情况

射线检测就是用来解决这些问题的
可以在指定点发射一个指定方向的射线
判断该射线与哪些碰撞器相交，得到对应对象

#### 知识点二 射线对象
1. **3D世界中的射线**
参数一：起点
参数二：方向（一定记住：不是两点决定射线方向，而是直接代表方向向量）

``` c#
//假设有一条射线：
//起点为(1, 0, 0)，
//方向为世界坐标z轴正方向。

Ray r = new Ray(Vector3.right, Vector3.forward);

//成员：
//1. 起点
r.origin
//2. 方向
r.direction
```

2. **摄像机发射的射线**
起点：屏幕位置，
方向：摄像机视口方向
``` c#
//鼠标选择场景的物体
// 参数：坐标
Ray r = Camera.main.ScreenPointToRay(Input.mousePosition);
```

注意：
单独的射线没有实际的意义
需要用射线集合物体系统进行射线碰撞判断

#### 知识点三 碰撞检测函数
注意：
射线检测也是==瞬时==的，执行代码时进行一次射线检测

1. **最原始的射线检测**
如果碰撞到对象，返回true

参数一：射线
参数二：检测的最大距离，超过距离就不检测
参数三：检测指定层级（默认检测所有层）
参数四：是否忽略触发器（默认使用UseGlobal）
	* UseGlobal：使用全局设置
	* Collide：检测触发器
	* Ignore：忽略触发器
``` c#
//准备一条射线
Ray r = new Ray(Vector3.zero, Vector3.forward);

//射线碰撞检测
bool isCollide = Physics.Raycast(r, 100, 
	1 << LayerMask.NameToLayer("Monster"),
	QueryTriggerInteraction.UseGlobal);
	
```

* 另一种重载：
不必传入射线，直接传入起点和方向，也可以用于判断
参数一：起点
参数二：方向
参数三：检测的最大距离，超过距离就不检测
...
``` c#
bool isCollide = Physics.Raycast(Vector3.zero, Vector3.forward, 100, 
	1 << LayerMask.NameToLayer("Monster"),
	QueryTriggerInteraction.UseGlobal);
```

2. **获取射线相交的单个物体的信息**
  物体信息结构体 `RaycastHit`
成员：
1. 碰撞器 `.collider`
2. 碰撞到的点 `.point`
3. 法线信息 `.normal`
4. 碰撞到的对象的位置 `.transform.position`
5. 碰撞到的对象离自己的距离 `.distance`

参数一：射线
参数二：`out RaycastHit`
参数三：距离
参数四：检测指定层级
参数五：是否忽略触发器
``` c#
Ray r = new Ray(Vector3.zero, Vector3.forward);

RaycastHit hitInfo;

if(Physics.Raycast(r, out hitInfo, 100))
{
	
}
```

* 另一种重载：
不必传入射线，直接传入起点和方向，也可以用于判断

3. **获取射线相交的多个物体的信息**
可以得到碰撞到的多个对象
==如果没有，就是容量为0的数组==
数组顺序：
==最后碰撞到的对象在第一位==

参数一：射线
参数二：距离
参数三：检测指定层级
参数四：是否忽略触发器
``` c#
Ray r = new Ray(Vector3.zero, Vector3.forward);

RaycastHit[] hitInfos = Physics.RaycastAll(r, 100);
```

* 另一种重载：
不必传入射线，直接传入起点和方向，也可以用于判断

* 另一个函数：
返回碰撞的数量，通过out得到数据
``` c#
Ray r = new Ray(Vector3.zero, Vector3.forward);

RaycastHit[] hitInfos = new RaycastHit[]();

int num = Physics.RaycasrNonAlloc(r, out hitInfos, 100);
```

#### 知识点四 使用时注意的问题
注意：
距离、层级两个参数都是int型
当传入参数时，一定要明确传入的参数是距离还是层级

#### 练习
1. 请用资料给的资源，实现鼠标点击场景上一面墙，在点击的位置创建子弹特效和弹孔
``` c#
class Test : MonoBehavior
{
	void Update()
	{
		if(Input.GetMouseButtonDown(0))
		{
			//射线检测
			Ray r = Camera.main.ScreenPointToRay(Input.mousePosition);
			RaycastHit hitInfo;
			if(Physics.Raycast(r, out hitInfo, 100))
			{
				//创建特效
				GameObject obj = Instantiate(Resources.Load(".../..."));
				//位置
				obj.transform.position = hitInfo.point + hitInfo.normal * 0.2f;
				//角度
				obj.transform.roation = Quaternion.LookRotation(hitInfo.normal);
				Destroy(obj, 0.8f);
				
				//创建弹孔
				obj = Instantiate(Resources.Load(".../..."));
				//位置
				obj.transform.position = hitInfo.point + hitInfo.normal * 0.2f;
				//角度
				obj.transform.roation = Quaternion.LookRotation(hitInfo.normal);
				Destroy(obj, 1.8f);
			}
		}
	}
}
```

2. 场景上有一个平面，有一个立方体，当鼠标点击选中立方体时，长按鼠标左键，可以拖动立方体在平面上移动，点击鼠标右键取消选中
``` c#
class Test : MonoBehavior
{
	public Transform sel;
	RaycastHit hitInfo;

	void Update()
	{
		//选中
		if(Input.GetMouseButtonDown(0))
		{
			if(Physics.Raycast(Camera.main.ScreenPointToRay(Input.mousePosition),
				out hitInfo,100))
			{
				//选中对象
				sel = hitInfo.transform;
				print(sel.name);
			}
		}
		
		//拖动物体
		if(Input.GetMouseButton(0) && sel != null)
		{
			if(Physics.Raycast(Camera.main.ScreenPointToRay(Input.mousePosition),
				out hitInfo,100,
				1 << LayerMask.NameToLayer("Floor")))
			{
				sel.position = hitInfo.point + Vector3.up * offset;
			}
		}
		
		//取消选中
		if(Input.GetMouseButtonDown(1))
		{
			print(sel.name);
			sel = null;
		}
	}
}
```

# 三、Unity核心
## 第2章 认识模型的制作流程
### 模型的制作过程
* 美术工种——**建模**
	* 第一步：建模
	* 第二步：展UV
	* 第三步：材质和纹理贴图
* 美术工种——**动作**
	* 第四步：骨骼绑定
	* 第五步：动画制作

### 第一步：建模
通过建模软件用一个个面片将模型制作出来

* **面片**：3点构成一个面，面的最小单位是三角形，可以使用3点以上的面但是建议使用三角面
* **网格信息**：一般指的是模型的顶点面片信息

即，==建模其实就是用三角面组装拼凑泥人==

### 第二步：展UV
==展开模型的网格到平面上产生映射关系==

* **UV（纹理贴图坐标）**：有U轴和V轴，类似空间中的xyz轴。纹理贴图坐标中的每一个点都和3D模型上的位置信息是相互联系的。

展UV就好像是把做好的3D模型平铺到一张2D图片上，并把这张2D图片上对应位置的信息和3D模型上位置信息对应起来，为之后给模型上色做准备

即，展UV就好像是把一张纸做的立方体拆开变成一张纸

### 第三步：材质和纹理贴图
==使用UV信息画贴图选择材质出效果==

* **纹理**：一张2D图片
* **贴图**：把纹理通过UV坐标映射到3D物理表面
* **纹理贴图**：代指模型的颜色信息，UV信息等等

* **材质**：模型的表现，通过纹理贴图提供的信息使用不同的着色器算法，呈现不同的表现效果。
比如：金属、玻璃、塑料、透明等

* **着色器**：决定材质的表现效果，shader就是着色器

### 第四步：骨骼绑定
* **骨骼绑定**：为模型定义骨骼信息，定义骨骼控制哪些网格信息

### 第五步：动画制作
利用骨骼的旋转来制作3D动画

在一条时间轴上，制作关键帧的位置，通过一些规则决定从上一个关键帧到这一个关键帧的变化应该如何过渡。不停地制作关键帧就可以制作出最终的动画效果。

## 第3章 2D相关
### 图片导入设置
#### 图片导入概述
##### 知识点一 Unity支持的图片格式
* BMP：是Windows操作系统的标准图像文件格式，特点是几乎不进行压缩，占磁盘空间大
* TIF：基本不损失图片信息的的图片格式，确定是体积大

* JPG：一般指JPEG格式，属于有损压缩格式，能够让图像压缩在很小的存储空间，一定程度上会损失图片数据，无透明通道
* PNG：无损压缩算法的位图格式，压缩比高，生成文件小，有透明通道

* **TGA：支持压缩，使用不失真的压缩算法，还支持编码压缩。体积小，效果清晰，兼备BMP的图像质量和JPG的体积优势，有透明通道**
* PSD：是PhotoShop专用的格式，通过第三方工具可以直接将PSD界面转为UI界面

其他：
EXR、GIF、HDR、IFF、PICT等

==其中Unity最常用的格式是：
JPG、PNG、TGA==

##### 知识点二 图片设置的6大部分
1. 纹理类型
2. 纹理形状
3. 高级设置
4. 平铺拉伸
5. 平台设置
6. 预览窗口

#### 纹理类型设置
#### 纹理形状设置
#### 纹理高级设置
#### 纹理平铺拉伸设置
#### 纹理平台打包相关设置

### Sprite

### 2D物理系统
#### 刚体
##### 知识点一 学习2D物理系统的前提
学习2D物理系统之前先学习Unity入门的3D物理系统

##### 知识点二 2D物理系统中的刚体组件
刚体是物理系统中用于帮助我们进行模拟物理碰撞中力的效果的

2D刚体和3D刚体基本是一样的，
最大的区别是对象只会在XY平面中移动，并且只在垂直于该平面的轴上旋转

##### 知识点三 参数相关
1. **Material**
物理材质，在刚体上设置了物理材质，如果子物体有碰撞器但是没有设置材质则会通用刚体的物理材质。
如果不设置，将使用在Physics 2D窗口中设置的默认材质。
物理材质的优先级：
	1. 2D碰撞器上指定的2D物理材质
	2. 2D刚体上指定的2D物理材质
	3. Physics 2D窗口指定的默认2D物理材质

2. **Simulated**
如果希望2D刚体以及所有子对象2D碰撞器和==2D关节==都能模拟物理效果，则勾选

3. **Collision Detection**
-Discrete：离散检测算法，只会用新位置进行计算，速度过快时会穿过
-Continous：连续检测算法，计算量更大，但是不会发生穿过的情况

4. **Sleeping Mode**
对象处于静止状态时进入睡眠模式
Nerver Sleep：从不休眠，会一直进行检测计算，性能消耗较大	
Start Awake：最初处于唤醒状态
Start Asleep：最初处于睡眠状态，	但是可以被碰撞唤醒

5. **Interpolate**
物理更新间隔之间的插值运算
None：不应用移动平滑
Interploate：根据前一帧进行平滑处理
Extraploate：根据后一帧位置进行平滑处理

刚体类型
1. **Dynamic**

2. **Kinematic**
运动学类型
不受力的影响，只能通过代码让其动起来
能和Dynamic 2D刚体产生碰撞，但是不会动，只会进入碰撞检测函数，因此没有质量、摩擦系数等属性。
==性能消耗较低==，主要通过代码来移动旋转

**Similuated**
启用时，会充电一个无限质量的不可移动对象，可以和所有2D刚体产生碰撞
当Use Full Kinematic Contacts禁用时，只会和Dynamic 2D刚体碰撞

**Use Full Kinematic Contacts**
如果希望能和所有2D刚体碰撞，则勾选
不启用则不会和Kinematic 2D刚体和Static 2D刚体碰撞

3. **Static**
完全不同的需要检测碰撞的对象
相当于是无限质量不可移动的对象
==性能消耗最小==，只能和Dynamic 2D刚体碰撞

##### 知识点四 如何选择不同类型的刚体
Dynamic 动态刚体：
受力的作用，需要移动、需要碰撞的对象

Kinematic 运动学刚体：
通过刚体API移动的对象，不受力的作用，但想要进行碰撞检测的对象

Static 静态刚体：
不移动、不受力作用的静态物体，但是想要进行碰撞检测的对象

##### 知识点五 刚体API

``` c#
//加力
Rigidbody2D r = this.GetComponent<Rigidbody2D>();
r.AddForce(new Vector2(0, 100));

//速度
r.velocity = new Vector2(1, 0);
```

#### 碰撞器

#### 物理材质

#### 恒定力

#### 区域效应器

#### 浮力效应器

#### 点效应器

#### 平台效应器

#### 表面效应器


### SpriteShape

### Tilemap

## 第4章 动画基础
### Animation动画窗口
### Animator动画状态机

## 第5章 2D动画
### 序列帧动画
### 骨骼动画——2D Animation
### 骨骼动画——Spine

## 第6章 模型导入相关
### Model模型页签
### Rig操纵（骨骼）页签
### Animation动画页签
### Materials材质纹理页签

## 第7章 3D动画相关
### 3D动画的使用
### 动画分层和遮罩
### 动画1D混合
### 动画2D混合
### 动画子状态机
### 动画IK控制
### 动画目标匹配
### 状态机行为脚本
### 状态机复用
### 角色控制器

## 第8章 导航寻路相关

---
# Unity中的UI系统——UGUI
## UGUI基础——六大基础组件
Hierarchy面板右键 -> UI
UI中的所有内容都是UGUI

* **Canvas对象**上依附的组件：
Canvas：画布组件，主要用于渲染UI控件
Canvas Scaler：==画布分辨率自适应组件==
Graphic Raycaster：射线事件交互组件，主要用于控制射线响应相关
RectTransform：UI对象位置锚点控制组件，==主要用于控制位置和对齐方式==

* **EventSystem对象**上依附的组件：
主要用于监听玩家操作
EventSystem：玩家输入事件响应系统
Standalone Input Module：独立输入模块组件

### Canvas 画布组件
#### Canvas作用
是UGUI中所有UI元素能够被显式的根本，主要负责渲染自己的所有UI子对象

==如果UI控件对象不是Canvas对象的子对象，则UI控件对象将不能被渲染==

#### 场景中可以有多个Canvas对象
==如果没有特殊需求，一般情况下场景一个Canvas即可==

#### Canvas组件的3种渲染方式
* **Screen Space - Overlay**
	==屏幕空间，覆盖模式，UI始终在前==
参数：
==Pixel Perfect==：是否开启无锯齿精确渲染（性能换效果）
==Sort Order==：排序层编号（用于控制多个Canvas时的渲染先后顺序）

* **Screen Space - Camera**
	==屏幕空间，摄像机模式，3D物体可以显示在UI之前==
参数：
==Render Camera==：用于渲染UI的摄像机（如果不设置将类似覆盖模式）
**不推荐使用主摄像机，需要用专门的摄像机**
==Plane Distance==：UI平面在摄像机前方的距离，类似整体Z轴的感觉
==Sorting Layer==：所在排序层
==Order in Layer==：排序层的序号

* **World Space**
	世界空间，3D模式

### Canvas Scaler 画布分辨率自适应组件

### Graphic Raycaster 射线事件交互组件

### RectTransform UI对象位置锚点控制组件
继承于Transform，是专门处理UI元素位置大小相关的组件

Transform只处理==位置、角度、缩放==，RectTransform在此基础上加入了矩形相关，将UI元素当做一个矩形来处理，加入了==中心点、锚点、长宽==等属性，==目的是更加方便地控制其大小以及分辨率自适应中的位置适应==

#### 参数：
**Pivot**：
==轴心点==（中心点），取值范围0~1

**Anchors**：
相对父矩形==锚点==
Min是矩形锚点范围X和Y的最小值
Max是矩形锚点范围X和Y的最大值
取值范围0~1

PosX、PosY、PosZ：
轴心点（中心点）相对锚点的位置

Width、Height：
矩形的宽、高

Left、Top、Right、Bottom：
矩形边缘相对于锚点的位置（==当锚点分离时会出现这些内容==，X、Y变成Left、Top，Width、Height变成Right、Bottom）

Rotation：
围绕轴心点旋转的角度

Scale：
缩放大小

Blueprint Mode（蓝图模式）按钮：
启用后，编辑旋转和缩放不会影响矩形，只会影响显示内容
（很少使用）

Raw Edit Mode（原始编辑模式）按钮：
启用后，改变轴心和锚点值不会改变矩形位置
（很少使用）

==锚点和轴心点快捷设置面板==：
鼠标左键点击九宫格布局可以快捷设置==锚点==
按住==shift==点击鼠标左键可以同时设置==轴心点==（相对自身矩形）
按住==alt==点击鼠标左键可以同时设置位置

### EventSystem 玩家输入事件响应系统

### Standalone Input Module 独立输入模块组件


## UGUI基础——三大基础控件
### Image 图像控件

### Text 文本控件

### RawImage 原始图像控件

## UGUI基础——组合控件
### Button 按钮控件
### Toggle 开关控件
### InputField 文本输入控件
### Slider 滑动条控件
### ScrollBar 滚动条控件
### ScrollView 滚动视图控件
### Dropdown 下拉列表控件

## UGUI基础——图集制作

## UGUI进阶
### UI事件监听接口

### EventTrigger事件触发器

### 屏幕坐标转UI相对坐标

### 遮罩Mask

### 模型和粒子显示在UI之前

### 异形按钮

### 自动布局组件

### 画布组 Canvas Group



---
# Unity数据持久化——JSON

---
# Unity网络开发基础
## 网络开发必备理论
## 网络协议
## 网络通信—Socket—TCP
## 网络通信—Socket—UDP
## 网络通信—文件传输FTP
## 网络通信—超文本传输HTTP
## 消息处理


---
# 【唐老狮】Unity中的MVC思想（框架）
## MVC基础
Model View Controller
模型、视图、控制器

Model：数据
View：界面
Controller：业务逻辑

==主要用于UI系统逻辑==

优点：降低耦合、方便修改、逻辑更清晰
缺点：脚本变多、体量变大、流程变复杂

## MVC变形——MVX概念
### MVX是什么

### MVP

### MVVM

### MVE

## PureMVC框架



# 【唐老狮】Unity程序基础框架（重置版）
一般游戏框架会包含且不限于：
* 资源管理
* 场景管理
* 输入管理
* 音频管理
* 网络模块
* 存储和持久化
* 物理引擎集成
* 日志调试工具
* 等

许多游戏框架都是基于==设计模式==和==软件架构思想==，根据开发需求进行设计的
设计模式：
软件架构思想：是一种关于组织和设计软件系统的理念和原则（比如MVC和ECS）

市面上有很多第三方框架：GameFramework、PureMVC、Unity中较新的Dots系统

## 单例模式基类
### 不继承MonoBehavior的单例模式基类
``` c#
class BaseSingleton<T> where T:class,new()
{
	private static T instance_;
	
	public static T Instance
	{
		get
		{
			if(instance_ == null)
				instance_ = new T();
			return instance_;
		}
	}
}
```

#### 潜在的安全问题
1. 构造函数问题：构造函数可在外部调用，可能会破坏唯一性
2. 多线程问题：当多个线程同时访问管理器时，可能会出现共享资源的安全访问问题

### 继承MonoBehavior的单例模式基类
#### 注意事项
1. 继承MonoBehavior的脚本不能new！
2. 继承MonoBehavior的脚本一定得依附在GameObject上

#### 实现挂载式的单例模式基类
``` c#
class MonoSingleton<T> : MonoBehavior where T:MonoBehavior
{
	private static T instance_;
	
	public static T Instance => instance_;
	
	protected virtual void Awake()
	{
		instance_ = this as T;
	}
}
```

==不建议使用==
**因为很容易被破坏单例模式的唯一性**
1. 挂载多个脚本
2. 切换场景回来时：由于场景放置了挂载脚本的对象，回到该场景时，又会有一个该单例模式对象
3. 通过代码动态地添加多个该脚本

#### 实现自动挂载式的单例模式基类
``` c#
class MonoAutoSingleton<T> : MonoBehavior where T:MonoBehavior
{
	private static T instance_;
	
	public static T Instance
	{
		get
		{
			if(instance_ == null)
			{
				//动态创建
				//动态挂载
				// 创建空物体
				GameObject obj = new GameObject();
				// 为空物体改名
				obj.name = typeof(T).ToString();
				// 动态挂载对应的单例模式脚本
				instance_ = obj.AddComponent<T>();
				// 过场景时不移除对象，保证它在整个生命周期中都存在
				DontDestroyOnLoad(obj);
			}
			return instance_;
		}
	}
}
```

==推荐使用==
无需挂载
无需动态创建
无需关系切场景时

#### 潜在的安全问题
1. 构造函数问题
继承MonoBehavior的函数，不用new，不必担心公共构造函数
2. 多线程问题
Unity主线程的相关内容，不允许其他线程访问
3. 重复挂载问题
手动重复挂载
代码重复添加
需要人为干涉，定规则，或通过代码逻辑强制处理

### 【安全问题】唯一性问题——构造函数
不继承Mono的单例模式基类
唯一性问题：
``` c#
MonoSingleton<Test> testSingleton = new MonoSingleton<Test>();

Test test = new Test();
```

#### 解决
1. 将单例模式基类变成抽象类
``` c#
class abstract BaseSingleton<T> where T:class,new()
{
	private static T instance_;
	
	public static T Instance
	{
		get
		{
			if(instance_ == null)
				instance_ = new T();
			return instance_;
		}
	}
}
```
2. 规定继承单例模式基类的类必须实现私有的无参构造函数（可能会报错）

3. 在单例模式基类通过反射来调用私有构造函数实例化对象
利用`Type`中的`GetConstructor(约束条件, 绑定对象, 参数类型, 参数修饰符)`方法来获取私有无参构造函数
``` c#
ConstructorInfo constructor = typeof(T).GetConstructor(
	BindingFlags.Instance | BindingFlags.NonPublic,//表示成员私有方法
	null,				//表示没有绑定对象
	Type.EmptyTypes,	//表示没有参数
	null				//表示没有参数修饰符
);
```

``` c#
using System.Reflection;

class BaseSingleton<T> where T:class
{
	private static T instance_;
	
	public static T Instance
	{
		get
		{
			if(instance_ == null)
			{
				//反射
				Type type = typeof(T);
				ConstructorInfo info = type.GetConstructor(
					BindingFlags.Instance | BindingFlags.NonPublic,
					null,
					Type.EmptyTypes,
					null);
				if(info == null)
				{
					Debug,LogError("没有私有无参构造函数");
				}
					
				instance_ = info.Invoke(null) as T;
			}
			return instance_;
		}
	}
}
```

### 【安全问题】唯一性问题——重复挂载
继承Mono的单例模式基类
唯一性问题：
1. 手动挂载多个相同的单例模式脚本
2. 代码动态添加多个相同的单例模式脚本

#### 解决
对于挂载式的单例模式脚本：
1. 同个对象重复挂载（但在其他对象也可以挂）
**为脚本添加特性** `[DisallowMultipleComponent]`
2. 修改代码逻辑
**判断如果存在对象则移除脚本**
``` c#
class MonoSingleton<T> : MonoBehavior where T:MonoBehavior
{
	private static T instance_;
	
	public static T Instance => instance_;
	
	protected virtual void Awake()
	{
		if(instance_ != null)
		{
			//移除该脚本
			Destroy(this);
			return;
		}
		instance_ = this as T;
		//不移除对象
		DontDestroyOnLoad(this.gameObject);
	}
}
```
对于自动挂载式的单例模式脚本：
1. 制定使用规则，不允许使用手动挂载或代码添加

### 【安全问题】线程安全——是否加锁
#### 解决多线程并发带来的问题
1. 不继承Mono的单例模式基类
建议加锁，避免以后使用多线程出现问题，比如在处理网络通信模块、复杂算法模块等
``` c#
class BaseSingleton<T> where T:class,new()
{
	private static T instance_;
	
	//用于加锁的对象
	protected static readonly object lockObj = new object();
	
	public static T Instance
	{
		get
		{
			//双重判断,解决性能问题
			if(instance_ == null)
			{
				lock(lockObj)
				{
					if(instance_ == null)
						instance_ = new T();
				}
			}
			return instance_;
		}
	}
}
```

2. 继承Mono的单例模式基类
可加可不加，建议不加
因为Unity主线程的内容不允许被其他线程访问

### 【补充】饿汉 和 懒汉
#### “懒汉”单例模式
主要特点是在属性或者方法中进行判空后再实例化。

这个懒字体现在：==这种单例模式只会在第一次使用时才创建事例，而不是应用程序启动时就创建。==一种“敌不动我不动”，“催一下动一下”的感觉

它的好处也在于此，因为==只有当我们代码中需要用到某个单例模式对象时才会去实例化分配内存。==

在商业项目中，游戏系统是非常多的，内存开销也是较大的，懒汉模式的==延迟实例化特点==可以帮助我们在一定程度上缓解一丁点的内存压力。（因为用才分配，不用不分配）

#### “饿汉”单例模式
这个饿字的体现在：==这种单例模式在以下几种情况下会直接初始化==

1. ==创建该类型实例时==（比如外部手动new一个对象，但是并没有使用该单例）
2. ==访问静态成员时==（比如访问其他静态内容，但是并没有使用该单例）
3. ==使用反射获取该类型时==
4. ==加载程序集时==（可能会，受诸多因素影响）

也就是说在这些时刻无论是否使用该事例，都会直接初始化

相当于不管我“吃不吃”，我就要创建它，先放在那，一种“饥不择食”的感觉。

虽然看起来“饿汉”没有“懒汉”那么好，但实际上，“饿汉”最大的优点就是“懒汉”的缺点，因为“饿汉”不用在实例化时考虑线程安全问题，它==具有天生的线程安全，C#会确保在多线程环境中这种静态成员的初始化只发生一次==。而“懒汉”在我们之前讲解的课程当中，大家已经感受到了，我们需要考虑多线程的并发访问问题。

#### 如何选择
在实际开发当中，我们==使用“懒汉”单例模式更多，因为它延迟性实例化的特点的诱惑力更强==。

## 公共Mono模块
必备知识点
* 委托
* 生命周期函数
* 协程

### 主要作用
* 让==不继承MonoBehavior的脚本==也能：
	1. 利用帧更新或定时更新处理逻辑
	2. 利用协程处理逻辑
	3. 可以统一执行管理帧更新或定时更新相关逻辑（继承MonoBehavior的脚本也可以）



* 统一执行管理以提升一定性能：
由于==Unity中过多脚本中的过多帧更新或定时更新函数会对性能有一定影响==，还可以对它们进行==集中化管理==，**减少Unity中多脚本的帧更新或定时更新函数的数量，从而提升一定的性能**

### 基本原理
实现一个继承MonBehavior的单例模式基类的公共Mono管理器脚本，
在其中实现
1. 通过==事件==或==委托==管理 不继承MonoBehavior脚本的相关更新函数
2. 提供==协程==**开启**或**关闭**的方法

使不继承MonoBehavior脚本也可以：
1. 利用帧更新或定时更新函数处理逻辑
2. 利用协程处理逻辑
3. 可以统一执行管理帧更新或定时更新相关逻辑

### 具体实现
1. 创建自动挂载式的MonoBehavior单例模式基类
2. 实现Update、FixedUpdate、LateUpdate生命周期函数
3. 声明对应事件或委托用于存储外部函数，并提供添加和移除的方法，从而达到让不继承MonoBehavior的脚本可以执行帧更新或定时更新的目的
4. 声明协程开启关闭函数，从而达到让不继承MonoBehavior的脚本可以执行协程的目的

``` c#
class MonoMgr : MonoAutoSingleton<MonoMgr>
{
	private event UnityAction fixedUpdateEvent;
	private event UnityAction updateEvent;
	private event UnityAction lateUpdateEvent;

	//添加和移除FixedUpdate帧更新监听函数
	public void AddFixedUpdateListener(UnityAction func)
	{
		fixedUpdateEvent += func;
	}
	public void RemoveFixedUpdateListener(UnityAction func)
	{
		fixedUpdateEvent -= func;
	}

	//添加和移除Update帧更新监听函数
	public void AddUpdateListener(UnityAction func)
	{
		updateEvent += func;
	}
	public void RemoveUpdateListener(UnityAction func)
	{
		updateEvent -= func;
	}
	
	//添加和移除LateUpdate帧更新监听函数
	public void AddLateUpdateListener(UnityAction func)
	{
		lateUpdateEvent += func;
	}
	public void RemoveLateUpdateListener(UnityAction func)
	{
		lateUpdateEvent -= func;
	}

	void FixedUpdate()
	{
		fixedUpdateEvent?.Invoke();
	}
	
	void Update()
	{
		updateEvent?.Invoke();
	}
	
	void LateUpdate()
	{
		lateUpdateEvent?.Invoke();
	}
}

```

### 测试
``` c#
class Test ; BaseSingleton<Test>
{
	Test() {}

	//Update
	void MyUpdate() {}

	public void CanStartUpdate()
	{
		MonoMgr.Instance.AddUpdateListener(MyUpdate);
	}
	
	public void CanStop()
	{
		MonoMgr.Instance.RemoveUpdateListener(MyUpdate);
	}
	
	//协程函数
	Coroutine coroutine_;
	IEnumerator MyCoroutine()
	{
		yield return new WaitForSeconds(3f);
	}
	
	public void CanStartCoroutine()
	{
		coroutine_ = MonoMgr.Instance.StartCoroutine(MyCoroutine());
	}
	
	public void CanStopCoroutine()
	{
		MonoMgr.Instance.StopCoroutine(coroutine_);
	}
}

//调用
void Update()
{
	Test.Instance.CanStartUpdate();
	
	Test.Instance.CanStartCoroutine();
}
```

## 缓存池（对象池）模块
必备知识
* C#的垃圾回收机制（GC）
* C#的泛型数据结构类Dictionary、List、Stack、Queue
* Unity的GameObject
* Unity的Resources

### 问题举例
1. 对象的频繁创建
频繁地实例化对象会带来一定的性能开销
2. 对象的频繁销毁
对象的频繁销毁会造成大量的内存垃圾，会造成GC的频繁触发，会带来卡顿感，影响体验

### 主要作用
**优化资源管理，提高程序性能**。
主要==通过重复利用已经创建的对象==，避免频繁地创建和销毁，从而减少系统的内存分配和垃圾回收带来的开销。

### 基本原理
用一个“柜子”中的“各种抽屉”来装“东西”，
用的时候就去拿（没有就创造，有就获取），
不用时就还回去（将“东西”分门别类地放入“抽屉”中）

### 具体实现
1. 继承 不继承Mono的单例模式基类
2. 声明“柜子”（Dictionnary）和“抽屉”（List、Stack、Queue等）容器
3. 取东西的方法
	3-1. 有抽屉并且抽屉里有东西，直接获取
	3-2. 没有抽屉或抽屉里没东西，创造
4. 还东西的方法
	4-1. 有抽屉，直接还
	4-2. 没有抽屉，创建抽屉再还
5. 清空柜子的方法
	切换创建时，对象都会被移除，这时应该清空柜子，
	否则会出现内存泄漏，并且下次取东西都会出问题。	
``` c#
class PoolMgr : BaseSingleton<PoolMgr>
{
	//柜子-抽屉
	private Dictionary<string, Stack<GameObject>> poolDic = 
		new Dictionary<string, Stack<GameObject>>();

	private PoolMgr() {}
	
	//取
	public GameObject GetObj(string name)
	{
		GameObject obj;
		if(poolDic.ContainsKey(name) && poolDic[name].Count > 0)
		{
			obj = poolDic[name].Pop();
			//激活对象
			obj.SetActive(true);
		}
		else
		{
			obj = GameObject.Instantiate(Resources.Load<GameObject>(name));
			//避免实例化出来的对象默认在名字后加一个(clone)
			obj.name = name;
		}
		return obj;
	}
	
	//还
	publlic void PushObj(GameObject obj)
	{
		//将对象失活，而不是移除对象
		//除了失活，还可以将对象放在屏幕外
		obj.SetActive(false);
		
		if(!poolDic.ContainsKey(obj.name))
		{
			poolDic[obj.name] = new Stack<GameObject>();
		}
		poolDic[name].Push(obj);
	}
	
	//清空柜子
	public void ClearPool()
	{
		poolDic.Clear();
	}
}
```

### 【优化】窗口布局优化
#### 制作思路
1. 柜子管理自己的柜子根物体
2. 抽屉管理自己的抽屉根物体
3. 失活时建立父子关系，激活时断开父子关系

#### 具体实现
1. 先实现将所有对象放入柜子根物体中
2. 再实现将对象放入对应的抽屉根物体中
	 用面向对象的思想将抽屉相关数据行为封装起来
``` c#
//"抽屉"
class PoolData
{
	private Stack<GameObject> dataStack = new Stack<GameObject>();
	//抽屉根对象
	private GameObject rootObj;
	
	public PoolData(GameObject poolObj, string name)
	{
		if(PoolMgr.isLayout)
		{
			rootObj = new GameObject(name);
			rootObj.SetParent(poolObj);
		}
	}
	
	public int Count => dataStack.Count;
	
	public GameObject Pop()
	{
		GameObject obj = dataStack.Pop();
		obj.SetActive(true);
		if(PoolMgr.isLayout)
		{
			obj.transform.SetParent(null);
		}
		
		return obj;
	}
	
	public void Push(GameObject obj)
	{
		obj.SetActive(false);
		if(PoolMgr.isLayout)
		{
			obj.transform.SetParent(rootObj.transform);
		}
		dataStack.Push(obj);
	}
}

class PoolMgr : BaseSingleton<PoolMgr>
{
	//柜子-抽屉
	private Dictionary<string, PoolData> poolDic = 
		new Dictionary<string, PoolData>();

	//池子根对象
	private GameObject poolObj;
	
	//是否开启布局功能
	public static bool isLayout = true;
	
	private PoolMgr() {}
	
	//取
	public GameObject GetObj(string name)
	{
		
		GameObject obj;
		if(poolDic.ContainsKey(name) && poolDic[name].Count > 0)
		{
			obj = poolDic[name].Pop();
		}
		else
		{
			obj = GameObject.Instantiate(Resources.Load<GameObject>(name));
			//避免实例化出来的对象默认在名字后加一个(clone)
			obj.name = name;
		}
		return obj;
	}
	
	//还
	publlic void PushObj(GameObject obj)
	{
		//池子根对象
		if(poolObj == null && isLayout)
		{
			poolObj = new GameObject("Pool");
		}
		
		if(!poolDic.ContainsKey(obj.name))
		{
			poolDic[obj.name] = new PoolData(poolObj, obj.name);
		}
		poolDic[name].Push(obj);
	}
	
	//清空柜子
	public void ClearPool()
	{
		poolDic.Clear();
		poolObj = null;
	}
}
```

#### 将其变为可控制开启的功能
这样可以避免在真机运行时，由于父子关系的频繁变化，带来一些额外的性能开销

### 【优化】对象上限优化
目前制作的缓存池，理论上来说，当动态创建的对象长时间不放回抽屉，每次从缓存池中动态获取对象时，会不停地创建新对象，即对象的数量是没有上限的，场景上的某种对象可以存在n个。

对象上限优化：
希望控制对象的数量有上限，对于不重要的资源没必要让其无限加量，而是将“使用最久”的资源直接抢来用。

主要目的：
更加彻底地优化资源，对对象的数量上限加以限制，可以优化内存空间，甚至优化性能（减少数量上限，可以减小渲染压力）

#### 制作思路
1. 在抽屉里声明一个容器用来记录正在使用的资源
2. 每次获取对象时，传入一个抽屉最大容量值（可以给一个默认值）
3. 从缓存池中获取对象时就需要创建抽屉，用于记录当前使用的对象
4. 每次取对象应分情况考虑
	4-1. 没有抽屉时
	4-2. 有抽屉，并且抽屉里有没使用中的对象或使用中的对象超过上限时
	4-3. 有抽屉，但是抽屉里没有对象，使用中对象也没有超过上限时
5. 每次放回对象时
由于记录了正在使用的对象，因此每次放入抽屉时还需要从记录容器中移除对象

#### 具体实现
``` c#
//"抽屉"
class PoolData
{
	//没有使用的对象
	private Stack<GameObject> dataStack = new Stack<GameObject>();
	//正在使用的对象
	private List<GameObject> usedList = new List<GameObject>();
	//抽屉根对象
	private GameObject rootObj;
	
	public PoolData(GameObject poolObj, string name, GameObject usedObj)
	{
		if(PoolMgr.isLayout)
		{
			rootObj = new GameObject(name);
			rootObj.SetParent(poolObj);
		}
		
		PushUsedList(usedObj);
	}
	
	public int Count => dataStack.Count;
	public int UsedCount => usedList.Count;
	
	public GameObject Pop()
	{
		GameObject obj;
		if(Count > 0)
		{
			obj = dataStack.Pop();
			usedList.Add(obj);
		}
		else
		{
			//0索引的对象表示使用时间最长的对象
			obj = usedList[0];
			//放到尾部表示最新的
			usedList.RemoveAt(0);
			usedList.Add(obj);
		}
		obj.SetActive(true);
		if(PoolMgr.isLayout)
		{
			obj.transform.SetParent(null);
		}
		
		return obj;
	}
	
	public void Push(GameObject obj)
	{
		obj.SetActive(false);
		if(PoolMgr.isLayout)
		{
			obj.transform.SetParent(rootObj.transform);
		}
		dataStack.Push(obj);
		usedList.Remove(obj);
	}
	
	public void PushUsedList(GameObject obj)
	{
		usedList.Add(obj);
	}
}

class PoolMgr : BaseSingleton<PoolMgr>
{
	//柜子-抽屉
	private Dictionary<string, PoolData> poolDic = 
		new Dictionary<string, PoolData>();

	//池子根对象
	private GameObject poolObj;
	
	//是否开启布局功能
	public static bool isLayout = true;
	
	private PoolMgr() {}
	
	//取
	public GameObject GetObj(string name, int maxnum = 50)
	{
		//池子根对象
		if(poolObj == null && isLayout)
		{
			poolObj = new GameObject("Pool");
		}
	
		GameObject obj;
		//有数量上限时的逻辑
		if(!poolDic.ContainsKey(name))
		{
			obj = GameObject.Instantiate(Resources.Load<GameObject>(name));
			//避免实例化出来的对象默认在名字后加一个(clone)
			obj.name = name;
			
			poolDic[name] = new PoolData(poolObj, name, obj);
		}
		else if(poolDic[name].Count > 0 || poolDic[name].usedCount >= maxNum)
		{
			obj = poolDic[name].Pop();
		}
		else
		{
			obj = GameObject.Instantiate(Resources.Load<GameObject>(name));
			//避免实例化出来的对象默认在名字后加一个(clone)
			obj.name = name;
			
			poolDic[name].PushUsedList(obj);
		}
		
		return obj;
	}
	
	//还
	publlic void PushObj(GameObject obj)
	{
		poolDic[name].Push(obj);
	}
	
	//清空柜子
	public void ClearPool()
	{
		poolDic.Clear();
		poolObj = null;
	}
}
```

### 【优化】思考
现在确定最大容量是通过在获取时传入最大参数，若传入参数出错可能会导致“超上限”
能否优化，以其他思路去制作，可以更加方便地处理上限逻辑。

#### 制作思路
1. 让使用者不用每次设定上限
2. 初始化抽屉时，第一次就直接定好上限，之后在内部判断即可

让缓存池中的对象挂载一个用于配置上限值的脚本，只需要在制作预设体时，将脚本挂好，设置好上限即可

## 事件中心模块
必备知识点
1. C#的Dictionary
2. C#的委托
3. C#的泛型
4. C#的里氏替换原则
5. 观察者设计模式

### 主要作用
举例：
怪物死亡 -> 玩家获取奖励、任务系统更新任务、成就系统、等等

**系统、模块、对象之间耦合度较高**

==解耦程序模块==
==降低程序耦合度==
==不需要直接引用或依赖与彼此的具体实现==

### 基本原理
==通过一个中心化的机制，使得多个系统、模块、对象之间可以进行松耦合的通信==

需要理由字典和委托的相关知识，再结合观察者设计模式
观察者设计模式：定义了一种一对多的依赖关系，使得当一个对象状态发生变化时，所有依赖于它的对象都能够得到通知并自动更新

### 具体实现
#### 知识点一 实现事件中心模块
1. 创建EventCenter，继承不继承MonoBehavior的单例模式基类
2. 声明管理事件用容器
3. 实现关键方法
	3-1. 触发(分发)事件 方法
	3-2. 添加事件监听者 方法
	3-3. 移除事件监听者 方法
	3-4. 清除所有事件监听者 方法

``` c#
class EventCenter : BaseSingleton<EventCenter>
{
	private Dictionary<string, UnityAction> eventDic = 
		new Dictionary<string, UnityAction>();

	private EventCenter() {}
	
	//触发(分发)事件 方法
	public void EventTrigger(string name)
	{
		if(eventDic.ContainsKey(name))
		{
			eventDic[name]?.Invoke();
		}
	}
	
	//添加事件监听者 方法
	public void AddEventListener(string name, UnityAction callback)
	{
		if(eventDic.ContainsKey(name))
		{
			eventDic[name] += callback;
		}
		else
		{
			//这样写会导致移除时不干净
			//eventDic[name] = new UnityAction(callback);
			eventDic.Add(name, null);
			eventDic[name] += callback;
			
		}
	}

	//移除事件监听者 方法
	public void RemoveEventListener(string name, UnityAction callback)
	{
		if(eventDic.ContainsKey(name))
		{
			eventDic[name] -= callback;
		}
	}
	
	//清除所有事件监听者 方法
	public void Clear()
	{
		eventDic.Clear();
	}
	
	public void Clear(string name)
	{
		if(eventDic.ContainsKey(name))
		{
			eventDic.Remove(name);
		}
	}
}
```

#### 知识点二 测试

``` c#
class Monster : MonoBehavior
{
	void Die()
	{
		EventCenter.Instance.EventTrigger("MonsterDie");
	}
}

class Player : MonoBehavior
{
	void Awake()
	{
		EventCenter.Instance.AddEventListener("MonsterDie", WaitForMonsterDie);
	}
	
	void OnDestroy()
	{
		EventCenter.Instance.RemoveEventListener("MonsterDie", WaitForMonsterDie);
	}
	
	void WaitForMonsterDie() {}
}
```

### 传递参数
利用万物之父object来传递参数

``` c#
class EventCenter : BaseSingleton<EventCenter>
{
	private Dictionary<string, UnityAction<object>> eventDic = 
		new Dictionary<string, UnityAction<object>>();

	private EventCenter() {}
	
	//触发(分发)事件 方法
	public void EventTrigger(string name, object info=null)
	{
		if(eventDic.ContainsKey(name))
		{
			eventDic[name]?.Invoke(info);
		}
	}
	
	//添加事件监听者 方法
	public void AddEventListener(string name, UnityAction<object> callback) { ... }

	//移除事件监听者 方法
	public void RemoveEventListener(string name, UnityAction<object> callback) { ... }
	
	//清除所有事件监听者 方法
	public void Clear() { ... }
	public void Clear(string name) { ... }
}
```

#### 存在的问题
传递值类型时，存在装箱拆箱，增加性能开销

### 参数类型自定义
里氏替换原则

``` c#
abstract class EventInfoBase { }

class EventInfo<T> : EventInfoBase
{
	public UnityActin<T> callbacks;
	
	public EvnetInfo(UnityAction<T> callback)
	{
		callbacks += callback;
	}
}

//无参
class EventInfo : EventInfoBase
{
	public UnityActin callbacks;
	
	public EvnetInfo(UnityAction callback)
	{
		callbacks += callback;
	}
}

class EventCenter : BaseSingleton<EventCenter>
{
	private Dictionary<string, EventInfoBase> eventDic = 
		new Dictionary<string, EventInfoBase>();

	private EventCenter() {}
	
	//触发(分发)事件 方法
	public void EventTrigger<T>(string name, T info)
	{
		if(eventDic.ContainsKey(name))
		{
			(eventDic[name]	as EventInfo<T>).callbacks?.Invoke(info);
		}
	}
	//触发(分发)事件 方法(无参)
	public void EventTrigger(string name)
	{
		if(eventDic.ContainsKey(name))
		{
			(eventDic[name]	as EventInfo).callbacks?.Invoke();
		}
	}
	
	//添加事件监听者 方法
	public void AddEventListener<T>(string name, UnityAction<T> callback)
	{
		if(eventDic.ContainsKey(name))
		{
			(eventDic[name]	as EventInfo<T>).callbacks += callback;
		}
		else
		{
			eventDic.Add(name, new EventInfo<T>(callback));
		}
	}
	//添加事件监听者 方法(无参)
	public void AddEventListener(string name, UnityAction callback)
	{
		if(eventDic.ContainsKey(name))
		{
			(eventDic[name]	as EventInfo).callbacks += callback;
		}
		else
		{
			eventDic.Add(name, new EventInfo(callback));
		}
	}

	//移除事件监听者 方法
	public void RemoveEventListener<T>(string name, UnityAction<T> callback)
	{
		if(eventDic.ContainsKey(name))
		{
			(eventDic[name]	as EventInfo<T>).callbacks -= callback;
		}
	}
	//移除事件监听者 方法(无参)
	public void RemoveEventListener(string name, UnityAction callback)
	{
		if(eventDic.ContainsKey(name))
		{
			(eventDic[name]	as EventInfo).callbacks -= callback;
		}
	}
	
	//清除所有事件监听者 方法
	public void Clear()
	{
		eventDic.Clear();
	}
	
	public void Clear(string name)
	{
		if(eventDic.ContainsKey(name))
		{
			eventDic.Remove(name);
		}
	}
}
```

### 事件名优化
使用枚举代替字符串

``` c#
enum EnventType
{
	E_Monster_Dead,
	...
}

/*
struct EventArgs{}

struct MonsterDead : EventArgs {}
*/

class EventCenter : BaseSingleton<EventCenter>
{
	private Dictionary<EnventType, EventInfoBase> eventDic = 
		new Dictionary<EnventType, EventInfoBase>();
	
	...
}
```

## 资源加载模块
必备知识
1. C#的Dictionary
2. C#的委托
3. C#的泛型
4. Unity的特殊文件夹
5. Unity的协程
6. Unity的UnityWebRequest（Unity网络开发基础）
7. Unity的AB包（Unity热更新解决方案之Lua）

### 主要作用
* 举例：

Unity中的资源加载：
Resources			（Resources文件夹）
UnityWebRequest	（Application.streamingAssetsPath文件夹）（Application.persistentDataPath文件夹）
AB包						（Application.streamingAssetsPath文件夹）（Application.persistentDataPath文件夹）
Editor(AssetDatabase) （Editor文件夹）
System.IO（Editor文件夹）（Application.streamingAssetsPath文件夹）（Application.persistentDataPath文件夹）
Addressables（Application.streamingAssetsPath文件夹）（Application.persistentDataPath文件夹）

问题：
1. **实际开发中，可能会采用多种资源加载模式组合使用，逻辑分散时，不方便管理**
2. **异步加载时，往往会使用协程，异步加载资源会产生很多协程相关的冗余代码**

主要作用：
1. ==方便统一管理各种资源加载卸载方式==（Resources、AB包、UnityWebRequest、Editor）
2. ==方便异步加载资源==（Resources、AB包、UnityWebRequest）

### 基本原理
==将各资源加载模式模块化==
让他们分别管理不同加载方式加载的资源
最终整合在一起
==方便在资源加载时根据需求进行选择性使用==

### 【Resources资源加载模块】具体实现
1. 创建ResourcesMgr，继承不继承MonoBehavior的单例模式基类
2. 封装Resources异步加载资源的方法（主要目的是避免异步加载的代码冗余）
3. 封装Resources同步加载资源的方法
4. 封装Resources卸载资源的方法

``` c#
class ResMgr : BaseSingleton<ResMgr>
{
	private ResMgr() {}
	
	//异步加载
	public void LoadAsync<T>(string path, UnityAction<T> callback) where T: Object
	{
		MonoMgr.Instance.StartCotoutine(ReallyLoadAsync<T>(path, callback));
	}
	
	IEumerator ReallyLoadAsync<T>(string path, UnityAction<T> callback) where T: Object
	{
		ResourceRequest rq = Resources.LoadAsync<T>(path);
		yield return rq;
		
		callback(rq.asset as T);
	}
	
	//同步加载
	
	//卸载资源
}
```

### 【Resources资源加载模块】进阶优化——异步加载问题
问题：
在同一帧开启多个协程加载同一个资源

**不依赖Resources内部的缓存机制，而是自己管理已经加载过的资源**
通过字典记录已经加载过的资源，每次在加载资源时，若发现是已经加载过的资源，直接使用即可

1. 字典
	key：资源名（路径 + 类型 拼接）
	value：自定义数据结构类（资源、委托、协程对象等）

2. 修改异步加载资源相关逻辑
	字典中不存在资源时：
		开启协程进行加载，并且此时就要记录在字典中（可以避免重复异步加载）
	字典中存在资源时：
		资源还没加载完 - 记录委托
		资源已经加载完 - 直接使用

3. 修改同步加载资源相关逻辑
	字典中不存在资源时：
		同步加载资源
	字典中存在资源时：
		资源还没加载完 - 停止协程
		资源已经加载完 - 直接使用

4. 修改卸载资源相关逻辑
	字典中存在资源时：
		资源还没加载完 - 记录删除标识，待加载完后真正移除 或 停止协程后移除
		资源已经加载完 - 直接卸载，移除字典中的资源

``` c#
abstract class ResInfoBase {}

class ResInfo<T> : ResInfoBase
{
	//资源
	public T asset;
	//异步加载结束后的委托
	public UnityAction<T> callback;
	//异步加载的协程
	public Coroutine coroutine;
	//是否要移除
	public bool isDel;
}

class ResMgr : BaseSingleton<ResMgr>
{
	private Dictionary<string, ResInfoBase> resDic = 
		new Dictionary<string, ResInfoBase>();

	private ResMgr() {}
	
	//异步加载
	public void LoadAsync<T>(string path, UnityAction<T> callback) where T: Object
	{
		ResInfo<T> info;
		string resKey = path + "_" + typeof(T).Name;
		if(!resDic.ContainsKey(resKey))
		{
			info = new ResInfo<T>();
			resDic[resKey] = info;

			info.callback += callback;
			info.coroutine = MonoMgr.Instance.StartCotoutine(ReallyLoadAsync<T>(path));
		}
		else
		{
			info = resDic[resKey] as ResInfo<T>;
			//异步加载还没结束
			if(info.asset == null)
			{
				info.callback += callback;
			}
			//异步加载结束
			else
			{
				callback?.Invoke(info.asset);
			}
		}
	}
	
	IEumerator ReallyLoadAsync<T>(string path) where T: Object
	{
		ResourceRequest rq = Resources.LoadAsync<T>(path);
		yield return rq;
		
		string resKey = path + "_" + typeof(T).Name;
		if(resDic.ContainsKey(resKey))
		{
			ResInfo<T> info = resDic[resKey] as ResInfo<T>;
			info.asset = rq.asset as T;
			
			//发现需要删除时
			if(info.isDel)
			{
				UnLoad<T>(path);
			}
			else
			{
				info.callback?.Invoke(info.asset);
			
				info.callback = null;
				info.cotoutine = null;
			}
		}
		
	}
	
	//同步加载
	public T Load<T>(string path) where T: Object
	{
		ResInfo<T> info;
		string resKey = path + "_" + typeof(T).Name;
		if(!resDic.ContainsKey(resKey))
		{
			T res = Resources.Load<T>(path);
			
			info = new ResInfo<T>();
			info.asset = res;
			resDic[resKey] = info;
			return res;
		}
		else
		{
			info = resDic[resKey] as ResInfo<T>;
			if(info.asset == null)
			{
				MonoMgr.Instance.StopCoroutine(info.coroutine);
				
				T res = Resources.Load<T>(path);
				info.asset = res;
				info.callback?.Invoke(res);
				
				info.callback = null;
				info.coroutine = null;
				return res;
			}
			else
			{
				return info.asset;
			}
		}
	}
	
	//卸载资源
	public void UnLoad<T>(string path) where T: Object
	{
		string resKey = path + "_" + typeof(T).Name;
		if(resDic.ContainsKey(resKey))
		{
			ResInfo<T> info = resDic[resKey] as ResInfo<T>;
			if(info.asset == null)
			{
				//为了确保移除资源，不使用这样的方式
				//MonoMgr.Instance.StopCoroutine(info.coroutine);
				//resDic.Remove(resKey);
				info.isDel = true;
			}
			else
			{
				resDic.Remove(resKey);
				Resources.UnloadAsset(info.asset as Object);
			}
		}
	}
}
```

#### 存在的问题
1. 卸载资源时，不知是否还有地方**正在使用该资源**
2. `UnloadUnusedAssets`是卸载没有使用的资源，无法判断是否使用

### 【Resources资源加载模块】进阶优化——引用计数
通过引用计数判断资源是否使用的问题

1. 为ResInfo类加入引用计数成员变量和方法
2. 使用资源时加
3. 不使用资源时减
4. 处理异步回调问题，某一个异步不使用资源了应该移除回调函数的记录
5. 修改移除资源函数逻辑，引用计数为0时才真正移除资源
6. 考虑资源频繁移除问题，加入马上移除bool标签
7. 修改移除不使用资源函数逻辑，释放时清除引用计数为0的记录

``` c#
abstract class ResInfoBase 
{
	//引用计数
	public int refCount;
	public void AddRefCount()
	{
		++refCount;
	}
	public void SubRefCount()
	{
		--refCount;
		if(refCount < 0)
			Debug.LogError("引用计数小于0了，请检查使用和卸载是否配对执行");
	}
}

class ResInfo<T> : ResInfoBase
{
	//资源
	public T asset;
	//异步加载结束后的委托
	public UnityAction<T> callback;
	//异步加载的协程
	public Coroutine coroutine;
	//决定引用计数为0时，是否要移除
	public bool isDel;
}

class ResMgr : BaseSingleton<ResMgr>
{
	private Dictionary<string, ResInfoBase> resDic = 
		new Dictionary<string, ResInfoBase>();

	private ResMgr() {}
	
	//异步加载
	public void LoadAsync<T>(string path, UnityAction<T> callback) where T: Object
	{
		ResInfo<T> info;
		string resKey = path + "_" + typeof(T).Name;
		if(!resDic.ContainsKey(resKey))
		{
			info = new ResInfo<T>();
			info.AddRefCount();
			resDic[resKey] = info;

			info.callback += callback;
			info.coroutine = MonoMgr.Instance.StartCotoutine(ReallyLoadAsync<T>(path));
		}
		else
		{
			info = resDic[resKey] as ResInfo<T>;
			info.AddRefCount();
			//异步加载还没结束
			if(info.asset == null)
			{
				info.callback += callback;
			}
			//异步加载结束
			else
			{
				callback?.Invoke(info.asset);
			}
		}
	}
	
	IEumerator ReallyLoadAsync<T>(string path) where T: Object
	{
		ResourceRequest rq = Resources.LoadAsync<T>(path);
		yield return rq;
		
		string resKey = path + "_" + typeof(T).Name;
		if(resDic.ContainsKey(resKey))
		{
			ResInfo<T> info = resDic[resKey] as ResInfo<T>;
			info.asset = rq.asset as T;
			
			//当引用计数小于0时,移除
			if(info.refCount <= 0)
			{
				UnLoad<T>(path, info.isDel, null, false);
			}
			else
			{
				info.callback?.Invoke(info.asset);
			
				info.callback = null;
				info.cotoutine = null;
			}
		}
		
	}
	
	//同步加载
	public T Load<T>(string path) where T: Object
	{
		ResInfo<T> info;
		string resKey = path + "_" + typeof(T).Name;
		if(!resDic.ContainsKey(resKey))
		{
			T res = Resources.Load<T>(path);
			
			info = new ResInfo<T>();
			info.asset = res;
			info.AddRefCount();
			resDic[resKey] = info;
			return res;
		}
		else
		{
			info = resDic[resKey] as ResInfo<T>;
			info.AddRefCount();
			if(info.asset == null)
			{
				MonoMgr.Instance.StopCoroutine(info.coroutine);
				
				T res = Resources.Load<T>(path);
				info.asset = res;
				info.callback?.Invoke(res);
				
				info.callback = null;
				info.coroutine = null;
				return res;
			}
			else
			{
				return info.asset;
			}
		}
	}
	
	//卸载资源
	public void UnLoad<T>(string path, bool isDel=false, UnityAction<T> callback=null,
		bool isSubRefCount = true)
		where T: Object
	{
		string resKey = path + "_" + typeof(T).Name;
		if(resDic.ContainsKey(resKey))
		{
			ResInfo<T> info = resDic[resKey] as ResInfo<T>;
			if(isSubRefCount)
				info.SubRefCount();
			info.isDel = isDel;
			if(info.asset != null && info.refCount <= 0 && info.isDel)
			{
				resDic.Remove(resKey);
				Resources.UnloadAsset(info.asset as Object);
			}
			//资源正在异步加载中
			else if(info.asset == null)
			{
				//为了确保移除资源，不使用这样的方式
				//MonoMgr.Instance.StopCoroutine(info.coroutine);
				//resDic.Remove(resKey);
				//info.isDel = true;
				//当异步加载不使用时，应移除回调记录，而不是直接卸载资源
				if(info.callback!=null)
					info.callback -= callback;
			}
		}
	}
	
	//异步卸载没有使用的资源
	public void UnLoadUnused(UnityAction callback)
	{
		List<string> list = new List<string();
		foreach(string path in resDic,Keys)
		{
			if(resDic[path].refCount <= 0)
				list.Add(path);
		}
		foreach(string path in list)
		{
			resDic.Remove(path);
		}
		
		MonoMgr.Instance.StartCoroutine(ReallyUnLoadUnused(callback))
	}
	
	IEnumerator ReallyUnLoadUnused(UnityAction callback)
	{
		AsyncOperation ao = Resources.UnloadUnusedAssets();
		yield return ao;
		callback();
	}
}
```

#### 注意事项
1. 加入引用计数的ResMgr
	资源有使用就有删除；
	==当使用某个资源的对象移除时，一定要记得移除callback==。
2. 如果觉得卸载资源的功能麻烦，也可以不使用卸载的相关方法
	加载相关逻辑不会有任何影响，和以前使用Resources的用法一样，
	只需要再添加一个主动清空字典的方法即可。
``` c#
	public void Clear(UnityAction callback)
	{
		MonoMgr.Instance.StartCoroutine(ReallyClear(callback));
	}

	IEnumerator ReallyClear(UnityAction callback)
	{
		resDic.Clear();
		AsyncOperation ao = Resources.UnloadUnusedAssets();
		yield return ao;
		callback();
	}
```

### 【Editor资源加载模块】主要作用和基本原理

### 【Editor资源加载模块】具体实现

## 音效管理模块

## UI管理模块
必备知识点
1. UGUI
2. Dictoinary
3. 委托
4. MonoBehavior

### 回顾制作UI面版
1. 拼面板（必须做）
2. 声明组件（重复工作）
3. 查找组件（重复工作）
4. 监听事件（重复工作）
5. 处理逻辑（必须做）

### 基本原理
1. **制作UI面板基类**
自动化地声明组件、查找组件、监听事件，无需每次写大量冗余代码
两种写法：
	1-1. 自动化工具生成代码（Unity编辑器开发）
	1-2. 基类规范冗余代码

2. **制作UI管理器**
管理所有UI面板，UI面板的显示隐藏都通过UI管理器来进行管理
公共方法：
	2-1. 显示面板
	2-2. 隐藏面板
	2-3. 获取面板
	2-4. 添加自定义事件

### UI面板基类
知识点：
1. 字典
2. 里氏替换原则
3. `GetComponentsInChildren`方法
4. 委托
5. 闭包

实现内容：
通用的查找组件方法
通用的添加事件方法
显示隐藏面板时的逻辑执行虚函数
获取指定组件的方法

关键点：
**制定控件命名规则**：
要使用的组件需要改名；
不使用的只用于显示的组件可以使用默认名；

``` c#
using UnityEngine.UI;//UIBehavior

abstract class BasePanel : MonoBehavior
{
	public Dictionary<string, UIBehavior> controlDic =
		new Dictionary<string, UIBehavior>();
	
	//控件的默认名字
	//如果得到的控件名字存在于这个容器，意味着不会通过代码去使用它，它只会是起到显示作用
	private static List<string> defaultNameList = 
		new List<string>(){
			"Imgae",
			"Text (TMP)",
			"RawImage",
			"Background",
			"Checkmark",
			"Label",
			"Text (Legacy)",
			"Arrow",
			"Placeholder",
			"Fill",
			"Handle",
			"Viewport",
			"Scrollbar Horizontal",
			"Scrollbar Vertical"
		};
	
	protected virtual void Awake()
	{
		//优先查找比较重要的组合性控件
		FindChildrenControl<Button>();
		FindChildrenControl<Toggle>();
		FindChildrenControl<Slider>();
		FindChildrenControl<InputField>();
		FindChildrenControl<ScrollRect>();
		FindChildrenControl<Dropdown>();
		
		//以下控件常常在以上控件中
		FindChildrenControl<Text>();
		FindChildrenControl<TextMeshProUGUI>();
		FindChildrenControl<Image>();
	}
	
	public abstract void ShowMe();
	public abstract void HideMe();
	
	protected virtual void BtnClick(string name) { }
	protected virtual void SliderValueChanged(string name, float value) { }
	protected virtual void ToggleValueChanged(string name, bool value) { }
	
	public T GetChildrenControl<T>(string name) where T:UIBehavior
	{
		if(controlDic.ContainsKey(name))
		{
			T ctl = controlDic[name] as T
			if(ctl == null)
				Debug.LogError($"不存在名字{name} 类型为{typeof(T)}的组件");
			return ctl;
		}
		else
		{
			Debug.LogError($"不存在名字{name}的组件");
			return null;
		}
	}
	
	private void FindChildrenControl<T>() where T:UIBehavior
	{
		string name = ctls[i].gameObject.name;
		
		T[] ctls = this.GetComponentsInChildren<T>(true);
		for(int i=0;i<ctls.Length;++i)
		{
			if(!defaultNameList.Contains(name))
			{	
				controlDic[name] = ctls[i];
				
				//判断类型，是否加事件监听
				if(ctls[i] is Button)
				{
					(ctls[i] as Button).onClick.AddListener(()=>{
						//闭包
						BtnClick(name);
					});
				}
				else if(ctls[i] is Slider)
				{
					(ctls[i] as Slider).onValueChanged.AddListener((value)=>{
						//闭包
						SliderValueChanged(name, value);
					});
				}
				else if(ctls[i] is Toggle)
				{
					(ctls[i] as Toggle).onValueChanged.AddListener((value)=>{
						//闭包
						ToggleValueChanged(name, value);
					});
				}
			}
		}
	}
}
```

### UI管理器层级规划
主要思路：
1. UI面板在任何场景都会显示，因此==Canvas和EventSystem应过场景不移除==，并且保证唯一性和动态创建
	1-1. UI管理器为不继承MonoBehavior的单例模式
	1-2. 在构造函数中动态创建设置好的Canvas和EventSystem预设体（如果使用了UI摄像机，也需要单独处
		理摄像机预设体）
2. UI面板的显示可以存在层级（前后）关系，可以预先创建好层级对象，提供获取层级对象的方法
	2-1. 在Canvas下创建好管理层级的子对象，之后面板作为对应层级对象的子对象达到分层作用
	2-2. 提供获取层级对象的方法

在Canvas下创建好管理层级的子对象：
* Canvas
	* Bottom
	* Middle
	* Top
	* System

UI管理器具体实现：
1. 存储面板的容器
2. 显示面板
3. 隐藏面板
4. 获取面板

``` c#
enum E_UILayer
{
	Bottom,
	Middle,
	Top,
	System
}

//!!!面板预设体名要与面板类名一致!!!
class UIMgr : BaseSingleton<UIMgr>
{
	Camera uiCamera;
	Canvas uiCanvas;
	EventSystem uiEventSys;

	//层级父对象
	Transform bottomLayer;
	Transform middleLayer;
	Transform topLayer;
	Transform systemLayer;

	Dictionary<string, BasePanel> panelDic = 
		new Dictionary<string, BasePanel>();

	private UIMgr()
	{
		//动态创建唯一的Canvas和EventSystem和UICamera
		uiCamera = GameObject.Instantiate(
			ResMgr.Instance.Load<GameObject>("资源名")).GetComponent<Camera>();
		GameObject.DontDestroyOnLoad(uiCamera.gameObject);
		
		uiCanvas = GameObject.Instantiate(
			ResMgr.Instance.Load<GameObject>("资源名")).GetComponent<Canvas>();
		uiCanvas.worldCamera = uiCamera;
		//层级父对象
		bottomLayer = uiCanvas.transform.Find("Bottom");
		middleLaye r= uiCanvas.transform.Find("Middle");
		topLayer = uiCanvas.transform.Find("Top");
		systemLayer = uiCanvas.transform.Find("System");
		GameObject.DontDestroyOnLoad(uiCanvas.gameObject);
		
		uiEventSys = GameObject.Instantiate(
			ResMgr.Instance.Load<GameObject>("资源名")).GetComponent<EventSystem>();
		GameObject.DontDestroyOnLoad(uiEventSys.gameObject);
	}
	
	public Transform GetLayerFather(E_UILayer type)
	{
		switch(type)
		{
			case E_UILayer.bottom:
				return bottomLayer;
			break;
			case E_UILayer.middle:
				return middleLayer;
			break;
			case E_UILayer.top:
				return topLayer;
			break;
			case E_UILayer.system:
				return systemLayer;
			break;
		}
	}
	
  	//显示面板
	//callback:可能是异步加载
	//isSync:是否是同步加载
	public void ShowPanel<T>(E_UILayer layer = E_UILayer.Middle,
		UnityAction<T> callback=null,bool isSync=false)
	{
		string panelName = typeof(T).Name;
		if(panelDic.ContainsKey(panelName))
		{
			panelDic[panelName].ShowMe();
			callback?.Invoke(panelDic[panelName] as T);
			return;
		}
		//不存在则加载
		ABResMgr.Instance.LoadResAsync<GameObject>("UI",panelName, (res)=>
		{
			//处理层级
			Transform father = GetLayerFather(layer);
			if(father == null)
				father = middleLayer;
				
			GameObject panelObj = GameObject.Instantiate(res, father, false);
			T panel = panelObj.GetComponent<T>();
			panel.ShowMe();
			callback?.Invoke(panel);
			
			panelDic[panelName] = panel;
		}, isSync);
	}
	
  	//隐藏面板
	public void HidePanel<T>()
	{
		string panelName = typeof(T).Name;
		if(panelDic.ContainsKey(panelName))
		{
			panelDic[panelName].HideMe();
			GameObject.Destroy(panelDic[panelName].gameObject);
			panelDic.Remove(panelName);
		}
	}
	
   //获取面板
   public void GetPanel<T>()
   {
		string panelName = typeof(T).Name;
		if(panelDic.ContainsKey(panelName))
		{
			return panelDic[panelName] as T;
		}
		return null;
   }
}
```

提出问题：
1. 如果同一帧连续显示同一个面板，应如何处理
2. 如果不想要直接销毁面板，应如何处理
3. 如果想要为控件添加一些非默认事件，应如何处理

### 进阶优化——异步加载问题

### 进阶优化——隐藏面板可选销毁

### 进阶优化——自定义事件添加函数

## 场景切换模块
必备知识点
1. 场景切换
2. 协程
3. 委托

为什么制作场景切换模块：
避免冗余

举例：
1. 切换场景中：
	更新Loading界面进度条
2. 切换场景后：
	隐藏Loading界面
	动态创建场景中关卡信息：角色、怪物、场景物件等

### 主要思路
1. 制作SceneMgr单例模式管理器
2. 实现同步加载场景的方法
3. 实现异步加载场景的方法
4. 实现外部获取异步加载场景进度

### 具体实现
``` c#
class SceneMgr : BaseSingleton<SceneMgr>
{
	private SceneMgr() {}
	
	//同步切换场景
	public void LoadScene(string name, UnityAction callback=null)
	{
		SceneManager.LoadScene(name);
		callback?.Invoke();
	}
	
	//异步切换场景
	public void LoadSceneAsync(string name, UnityAction callback=null)
	{
		MonoMgr.Instance.StartCoroutine(ReallyLoadSceneAsync(name, callback));
	}
	
	IEnumerator ReallyLoadSceneAsync(string name, UnityAction callback=null)
	{
		AsyncOperation ao = SceneManager.LoadSceneAsync(name);
		while(!ao.isDone)
		{
			//利用事件中心分发 每一帧的进度
			EventCenter.Instance.EventTrigger<float>(EventType.E_SceneLoadChange,
				ao.progress);
			yield return null;			
		}
		//避免最后一帧直接结束，没有同步出去
		EventCenter.Instance.EventTrigger<float>(EventType.E_SceneLoadChange, 1);
		
		callback?.Invoke();
	}
}
```

## 输入控制模块

## 其他模块

## 其他工具模块

---