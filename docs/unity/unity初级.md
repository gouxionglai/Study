# Unity

# 基础

## 安装

​	目前是2019版本，unity默认是不带脚本编辑器的，比如C#,  需要自己安装visual studio

## 界面

在default layout下大致分为5个区域：

- toolbar：工具栏	
- hierarchy：树形项目层级
- scene view：场景预览
- inspector：参数设置，（组件component的参数设置）
- project：资源管理器

## 操作

### 移动视角

#### 移动自己

鼠标右键即可旋转画面，按住的情况下，WSAD分别是前进后退左右移动

锁定视角：ALT+鼠标左键进行旋转，立体视角

复制物件：ctrl+d

全屏预览：shift+ 空格

#### 移动别人

QWERT：分别对应几个图标的功能，可以快速使用。

Q检视

W位移

E旋转

R缩放

### 原型

1. Cube	立方体
2. Sphere 球
3. Quad 四边形(点=4)
   1. Plane 平面(点>=4)
4. Cylinder 圆柱体

### 材质

Project中新增Mateira  可以直接拖拽到图形上

1. Albedo：Texture
2. Tiling：X轴或者Y轴的重复次数
3. offset：偏移量
4. Metallic：金属度
5. Smoothness：光滑度，越高则越光滑
6. Normal：法向量贴图。pbr material 可以看到凹凸有致的效果。但是这只是视觉差异，实际还是原本的平的

所以通常想要一个形象的物体，一般是提供了albedo+normal

如果要更形象，则下载素材网站：textures.com对应的全部的图即可。比如阴影效果Occlusion

skybox  天空

hdri	全景图



mesh collider：下面的Convex不勾选的话碰撞或许有问题，并且勾选了is trigger会穿透而不是碰撞

### 预制物件

一系列的组件，可以快速搭建想要的东西。当然也可以自定义一些东西设置成assets。

提示：比如一些重复的东西，比如墙，通常是3面墙，一般来说可以直接CTRL+D ,两次，但是会造成渲染负担，所以把一面墙加入assets，可以减少渲染。只渲染一次即可。



Z-fighting：旋转视角时，由于物体的边界重叠导致渲染的时候不知道应该渲染哪一个，造成会抖动。解决办法，边界不要完全一样，比如x,y,z。x多0.01, y多0.01。。



### 脚本

```c#
//设置参数x, 并且可变范围是(-1,1)
[Range(-1,1)]  //必须是大写开头的Range,不然不能识别
public double x;
```



模板Prefab

脚本间的调用，声明的时候，需要声明script的名字，而不是控件的名字。

比如A调用B，声明了之后，还需要在Unity中A控件下加入刚才声明script的实体。



## 相机

clear flags：一般选择skybox

projection：成像模式，3D选择Perspective, 2D选择Orthographic

field of view：视野远近。越大看的越远。(2倍镜，4倍镜，6倍镜...)

clipping planes：渲染范围。(near, far) .超出far的就不用渲染了。比如特别远的一个小草，肯定没必要渲染，走近了才看得清。

depth：深度。深度越大的在前面，越先渲染。（多个摄像机就靠这个值看渲染哪个。）

audio listener：音频监听。如果有多个摄像机同时使用，应该避免冲突。比如同时只能有一个camera启用，音频采用对应的camera的声音。



常用案例：

- ​	正常玩家第一，三人称视角：
  - 3D，
  - 并且layer看不到图标mask
- 小地图视角：
  - 摄像机在地图最上方，上帝视角。2D，
  - layer看不到玩家mask，
  - 看到的地图也不是真正的地图，只是一张不会动的地图而已。因为两个camera每秒都会渲染。太消耗资源了。小地图看的是全局，所以地图不会变，则用一张固定的图片显示就行了。只是玩家会移动需要渲染而已。
  - clear flags选择depth only，则边框变透明。透明的地方看到的还是主摄像机的内容。而不会被遮挡住
  - 同时看到主相机+小地图：设置小地图相机的viewport rect。 w,h 高度和宽度。通常是0.3，0.3。x，y就是小地图在主相机视野中的位置。





unity物理状态分为三种：

- 静态static
- 动态dynamic ：完全遵循牛顿定理，F=ma,     v = v。+  at,   ....
- 自动能kinematic：（比如幽灵 不受牛顿控制的。。）



物理引擎：  某些静止的没有必要每秒都渲染。比如墙壁。只有人物这些动的东西才需要一直渲染。  

unity是通过判断有无RigidBody来实现的。 包含则一直渲染。反之不渲染。

RigidBody是一个组件，添加之后就有了物理引擎。



碰撞就会有接触点，称之为contact point

粒子系统

play on awake： 准备好之后就播放。 ready to go 。 通过这个选项，设置某些效果。比如按下后触发。走过之后消失等等。

ps.gameObject();  找到事件的所有者

ps.transform.parent.gameObject()； 找到事件的transform的父级。

Destroy(Object，Time);  销毁

- [ ] 搭建平台
- [ ] 放置人物  设置镜头 第三人称
- [ ] 搭建围墙  设置成assets
- [ ] 搭建楼梯  可以攀爬
- [ ] 放置闸门  设置升降效果（暂时设置按键行为）
- [ ] 放置开关，踩上去开启，离开关闭





## InstantOC

插件，主要有三部分核心。

### 渲染管线

图形数据在GPU上经过运算处理，最后输出到屏幕的过程。

1. cpu判断哪些需要渲染，交给GPU（draw call）
2. gpu进行顶点处理。即描点。（转换为unity知道的坐标）
3. gpu进行图元装配（点--线--面）
4. gpu进行光栅化（转换为像素块）
5. gpu进行像素处理（上色，写入缓存）
6. gpu刷新缓存（因为是不停的渲染，有些更新了有些没更新，不用全部重新执行上面的步骤）
   1. 帧缓存（顾名思义，存放在显存中的每一帧的缓存一起交给显卡，而不是渲染一个给一个。）
   2. 深度缓存：定义离相机近的缓存优先，离相机远的缓存被覆盖（两辆车重叠一前一后，后面的车的缓存被前面的车替换掉，因为视角看不见后面的车，不需要渲染。）

### Occlusion Culling

遮挡剔除：摄像机看不见的物体剔除掉，不进行渲染。从而减少cpu负担

### LOD

层次，一个物体可以有很多个层次。层次约深，点面越多。即高级模型，中级模型，低级模型。

优点：减少渲染。距离摄像头越近，看起来越精致；距离摄像头越远，看起来约粗糙。

缺点：模型多了，增加了内存负担。

## 光照系统

### 简介

GI：global Illumination   即全局光照

光照：不仅仅是直接光，直接照射到的部分，还包括反射漫射等间接光

#### 直接光

- directional light：平型光，一般用于模拟太阳
- point light：点光，一般用于模拟灯泡，有亮度和范围，无角度
- spot lignt：聚光灯，一般用于模拟手电筒,圆锥形范围内进行光照。
- area light：区域光，由一个面，向一个方向发射光线。只照射该区域的物体。（bake only 只能是烘培光，有点像路灯）

#### 阴影shadow

​	阴影的计算也相当消耗性能。所以尽量优化，比如远处不明显的物体根本不需要阴影。

设置：Edit-->Project Settings-->Quality-->面板设置

输出的产品等级。手动设置shadow distance 越小，则越近的地方就没有阴影。性能更好

#### 环境光

Ambient light

#### 反射光

Reflection light

#### 间接光照

比较耗时。只有标记为静态的物体才会去计算间接光照。即inspector面板的static勾选项。





remark: 只看间接光的选项是：Scene面板选择irradiance  

### 实时GI

运行期间，实时去计算光线效果

### 烘培GI

将光线制作成图片，只是看起来像光而已，并不是真正的光。

减少了性能消耗。因为烘培GI是在开发期间就定好了，运行期间不会实时计算的。

### 光源侦测

给烘培GI提供依据。使看起来的光照效果更加真实。

## 声音

### Audio Source

### 3D Sound Settings

#### volume Rolloff

声音衰减的方式。Linear Rolloff。线性衰减。注意这个线性衰减是可以手动调整的。一般是平滑的。

简单理解就是距离越近听得越大声，距离越远听得越小声。超过最大距离则听不见。



# C#初级

## 数据类型

- 值类型（Value types）
- 引用类型（Reference types）
- 指针类型（Pointer types）

------

### 值类型

#### 整形

| 字节 | 有符号 | 无符号 |
| ---- | ------ | ------ |
| 1    | sbyte  | byte   |
| 2    | short  | ushort |
| 4    | int    | uint   |
| 8    | long   | ulong  |

#### 浮点型

| 字节 | 类型    | 精度                  |
| ---- | ------- | --------------------- |
| 4    | float   | 7位（加减法会有误差） |
| 8    | double  | 15-16位               |
| 16   | decimal | 28-29位（财务使用）   |

#### 布尔型

bool  值为true / false

#### 字符型

char	单引号

### 占位符

```c#
Console.WriteLine("{0:c}",10);	//￥10.00 金额
Console.WriteLine("{0:d2}",5);	//05  不足用0填充
Console.WriteLine("{0:f1}",1.26);	//1.3  根据精度要求显示
Console.WriteLine("金额：{0:p0}",0.1);	//10%	以百分比形式显示， p1表示显示1位小数。p表示默认10.00%
```

### 运算符

```c#
一元：+, -, *,  /, %
二元：==, !=, >=, <=, &&, ||, ++, --
三元： (a>b)?x:y  若a>b则x,反之则y。a>b可以替换成任意结果为true/false的表达式
```

### 类型转换

string 转int/double/xxxx

```c#
string number = "18";
int number01 = int.Parse(number);
```

## 语句

### 判断语句

#### if else

```c#
if(x>0){
    //x>0
}
else if(x<0){
	//x<0
}else{
    //x=0
}
```

#### switch

```C#
string x = "C";
switch(x){
    case "A":
        result = xx1;
        break;
    case "B":
        result = xx2;
        break;
    default:
        result = xx3;
        break;
}
```



### 循环语句

#### for

//知道循环次数。不知道结果。

```c#
//普通版
for(int i =0; i<5; i++){
    Console.WriteLine(i);
}
//变形版
int i = 0;
for( ;i<=5; ){
    Console.WriteLine(i);	//0,2,4
    i+=2;//自增步长改为+2
}
```

#### foreach

高级for循环

```c#
int[] array = new int[]{1,5,6,2,664,7345,3};
foreach(int x in array)
{
    Console.WriteLine(x);
}
// var/object为泛型， 类似于java的Object
for(object x in array)
{
    xxx
}
```



#### while

//使用条件：不知道循环次数。只知道结果。

```c#
int count=0;
while(condition){
    //do something;
    count++;
}
```

#### do...while

//相比while，差别是第一次的判断都会执行方法体。

//而while是判断通过才执行。所以差别只在第一次。

```c#
do{
    //循环体
}while(condition)
```





### 控制语句

//基本上都在循环体内使用。

#### continue

跳过本次循环。即循环还执行后续。

#### break

跳出该循环。即整个循环都不执行后续了



## 语法

### 代码构成

#### 命名空间

就好像是文件夹的感觉。一个文件夹下面有很多类文件

#### 类名

同java的类

#### 域

相当于java中的属性

#### 属性

对域的一种保护，提供对外的get/set.

而不是直接去操作域.

```c#
class Student
   {

    //域
      private string code = "N.A";
      private string name = "not known";
      private int age = 0;

      // 属性，对code进行封装
      public string Code
      {
         get
         {
            return code;
         }
         set
         {
            code = value;
         }
      }
```



#### 方法

同java的方法。

使用是  class.method(xxx);



### 方法

#### 参数传递

##### 值传递

##### 引用传递 ref

```c#
//这种基本类型是值传递，调用后原本的值根本不会改变。
//相当于只改了堆中的值
private void swap1(int a,int b){
    int c = a;
    a = b;
    b =c;
}

//但是如果是传递的引用，那就可以改到原本的值。
//相当于把引用传递了过去，把引用改了，值当然就变了
private void swap2(ref int a, ref int b){
    int c = a;
    a = b;
    b =c;
}

//当然调用也需要加上ref标记
int x = 1;
int y = 2;
swap(x,y);	//不改变
swap(ref x,ref y);	//改变
```

##### 输出传递 out

一般来说，参数都必须被赋值之后才能调用。但是输出传递则可以不用先指定。而且可以作为返回值输出

```c#
//比如int x = a+b; 正常来说都比如给定a和b具体的值。

//但是标记了out之后就可以不用
private void getValue(out int a,out int b)
{
    Console.WriteLine("请输入第一个值： ");
    x = Convert.ToInt32(Console.ReadLine());
    Console.WriteLine("请输入第二个值： ");
    y = Convert.ToInt32(Console.ReadLine());
}
static void Main(string[] args)
{
    //声明，但是不赋值
    int a , b;     
    /* 调用函数来获取值 */
    n.getValues(out a, out b);
}
```



#### 参数数组 Params

相当于java里面的可变参。不知道参数的数量，但是类型相同。

```c#
//比如n个整数相加
private int add(params int[] arr)
{
    int result = 0;
    foreach(var tempA in arr){
        result += tempA;
    }
    return result;
}
```

#### 构造函数

基本同java。有个特别的，有参构造调用之前还想调用无参构造怎么办？如果只能new一次的话。

**在构造函数后面加     :this(xxx)**

```c#
public Person()
{
    Console.WriteLine("无参构造执行了，，");
}

public Person(string name):this()
{
    Console.WriteLine("有参构造执行了1，，");
    this.name = name;
}

public Person(string name, int age):this(name)
{
    Console.WriteLine("有参构造执行了2，，");
    //this.name = name;
        this.age = age;
}
private void show(){
    Console.WriteLine("name="+name+", age="+age);
}
```

假设有如下调用，结果为：

```c#
Person person = new Person("zhangsan",18);
person.show();
//result 
无参构造执行了，，
有参构造执行了1，，
有参构造执行了2，，
name=zhangsan, age=18
```



### 可空类型

#### C# 单问号 ? 与 双问号 ??

**?** : 单问号用于对 int,double,bool 等无法直接赋值为 null 的数据类型进行 null 的赋值，意思是这个数据类型是 NullAble 类型的。

```
int? i = 3 
等同于
Nullable<int> i = new Nullable<int>(3);

int i; //默认值0
int? ii; //默认值null
```

**??** : 双问号 可用于判断一个变量在为 null 时返回一个指定的值。

相当于一个三元运算符的简化版。

```c#
//如果x为空，则设置指定值，不为空则为本身
x==null ? 3.1415 : x
x??3.1415
```



### string

不可变字符串。一旦赋值就不可修改了。

Insert(int index, string str);

IndexOf(char ch);

Remove(int index);

Replace(char old, char new)  /  Replace(string old, string new)

StartsWith(string str)

Contains(string str)

### StringBuilder

同java。**可变字符串。**主要是用来拼接。

```c#
StringBuilder sb = new StringBuilder(10);
for(int i=0; i<10;i++)
{
    sb.Append(i);	//减少内存消耗，直接拼接，不需要新开辟堆空间。
}
```



### 数组***

声明： int[] a;

初始化： a = new int[6];  //指定大小

获取赋值： a[0] =1 ;   a[1] = a[0];

方法：

- object x = Array.IndexOf(arr, index);
- arr.Length
- Array.Clear(arr);
- Array.Sort(arr);
- Array.reverse(arr);

产生随机数

```c#
random.Next(1,37);	//产生1-37的自然数
```

练习

```c#
//彩票机制: 6红1蓝， 红球范围[1,33],蓝球范围[1,16]
//1.产生随机的奖注：红球不能重复
//2.输入自己想要的奖注：红球不能重复
//3.开奖：查看中奖结果，中了几个：不看顺序，红球和蓝球分开对应。switch case
```

### 结构体

类似于类吧，但是和类不一样。

- 类是引用类型，结构是值类型。
- 结构不支持继承。
- 结构不能声明默认的构造函数。

```
struct Books
{
   public string title;
   public string author;
   public string subject;
   public int book_id;
};  
```

### 枚举

```
enum Days { Sun, Mon, tue, Wed, thu, Fri, Sat };
```

### 继承

```c#
//：代表继承
class son : father    {..}
```

### 接口

可以被接口实现，也可以被类实现。同样是用“:”进行表示

```c#
using System;

interface IParentInterface
{
    void ParentInterfaceMethod();
}

interface IMyInterface : IParentInterface
{
    void MethodToImplement();
}

class InterfaceImplementer : IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
        iImp.ParentInterfaceMethod();
    }

    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }

    public void ParentInterfaceMethod()
    {
        Console.WriteLine("ParentInterfaceMethod() called.");
    }
}
```

### 异常处理

- **try**：一个 try 块标识了一个将被激活的特定的异常的代码块。后跟一个或多个 catch 块。
- **catch**：程序通过异常处理程序捕获异常。catch 关键字表示异常的捕获。
- **finally**：finally 块用于执行给定的语句，不管异常是否被抛出都会执行。例如，如果您打开一个文件，不管是否出现异常文件都要被关闭。
- **throw**：当问题出现时，程序抛出一个异常。使用 throw 关键字来完成。

```c#
try
{
   // 引起异常的语句
}
catch( ExceptionName e1 )
{
   // 错误处理代码
}
catch( ExceptionName e2 )
{
   // 错误处理代码
}
catch( ExceptionName eN )
{
   // 错误处理代码
}
finally
{
   // 要执行的语句
}
```

### 文件的输入与输出

System.IO 命名空间有各种不同的类，用于执行各种文件操作，如创建和删除文件、读取或写入文件，关闭文件等。

| I/O 类         | 描述                               |
| :------------- | :--------------------------------- |
| BinaryReader   | 从二进制流读取原始数据。           |
| BinaryWriter   | 以二进制格式写入原始数据。         |
| BufferedStream | 字节流的临时存储。                 |
| Directory      | 有助于操作目录结构。               |
| DirectoryInfo  | 用于对目录执行操作。               |
| DriveInfo      | 提供驱动器的信息。                 |
| File           | 有助于处理文件。                   |
| FileInfo       | 用于对文件执行操作。               |
| FileStream     | 用于文件中任何位置的读写。         |
| MemoryStream   | 用于随机访问存储在内存中的数据流。 |
| Path           | 对路径信息执行操作。               |
| StreamReader   | 用于从字节流中读取字符。           |
| StreamWriter   | 用于向一个流中写入字符。           |
| StringReader   | 用于读取字符串缓冲区。             |
| StringWriter   | 用于写入字符串缓冲区。             |

FileMode的选项有：

| 参数       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| FileMode   | **FileMode** 枚举定义了各种打开文件的方法。FileMode 枚举的成员有：**Append**：打开一个已有的文件，并将光标放置在文件的末尾。如果文件不存在，则创建文件。**Create**：创建一个新的文件。如果文件已存在，则删除旧文件，然后创建新文件。**CreateNew**：指定操作系统应创建一个新的文件。如果文件已存在，则抛出异常。**Open**：打开一个已有的文件。如果文件不存在，则抛出异常。**OpenOrCreate**：指定操作系统应打开一个已有的文件。如果文件不存在，则用指定的名称创建一个新的文件打开。**Truncate**：打开一个已有的文件，文件一旦打开，就将被截断为零字节大小。然后我们可以向文件写入全新的数据，但是保留文件的初始创建日期。如果文件不存在，则抛出异常。 |
| FileAccess | **FileAccess** 枚举的成员有：**Read**、**ReadWrite** 和 **Write**。 |
| FileShare  | **FileShare** 枚举的成员有：**Inheritable**：允许文件句柄可由子进程继承。Win32 不直接支持此功能。**None**：谢绝共享当前文件。文件关闭前，打开该文件的任何请求（由此进程或另一进程发出的请求）都将失败。**Read**：允许随后打开文件读取。如果未指定此标志，则文件关闭前，任何打开该文件以进行读取的请求（由此进程或另一进程发出的请求）都将失败。但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。**ReadWrite**：允许随后打开文件读取或写入。如果未指定此标志，则文件关闭前，任何打开该文件以进行读取或写入的请求（由此进程或另一进程发出）都将失败。但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。**Write**：允许随后打开文件写入。如果未指定此标志，则文件关闭前，任何打开该文件以进行写入的请求（由此进程或另一进过程发出的请求）都将失败。但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。**Delete**：允许随后删除文件。 |

比如：

```c#
FileStream F = new FileStream("sample.txt", FileMode.Open, FileAccess.Read, FileShare.Read);
```



# c#高级

## 集合

下面是各种常用的 **System.Collection** 命名空间的类。点击下面的链接查看细节。

| 类                                                           | 描述和用法                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [动态数组（ArrayList）](https://www.runoob.com/csharp/csharp-arraylist.html) | 它代表了可被单独**索引**的对象的有序集合。它基本上可以替代一个数组。但是，与数组不同的是，您可以使用**索引**在指定的位置添加和移除项目，动态数组会自动重新调整它的大小。它也允许在列表中进行动态内存分配、增加、搜索、排序各项。 |
| [哈希表（Hashtable）](https://www.runoob.com/csharp/csharp-hashtable.html) | 它使用**键**来访问集合中的元素。当您使用键访问元素时，则使用哈希表，而且您可以识别一个有用的键值。哈希表中的每一项都有一个**键/值**对。键用于访问集合中的项目。 |
| [排序列表（SortedList）](https://www.runoob.com/csharp/csharp-sortedlist.html) | 它可以使用**键**和**索引**来访问列表中的项。排序列表是数组和哈希表的组合。它包含一个可使用键或索引访问各项的列表。如果您使用索引访问各项，则它是一个动态数组（ArrayList），如果您使用键访问各项，则它是一个哈希表（Hashtable）。集合中的各项总是按键值排序。 |
| [堆栈（Stack）](https://www.runoob.com/csharp/csharp-stack.html) | 它代表了一个**后进先出**的对象集合。当您需要对各项进行后进先出的访问时，则使用堆栈。当您在列表中添加一项，称为**推入**元素，当您从列表中移除一项时，称为**弹出**元素。 |
| [队列（Queue）](https://www.runoob.com/csharp/csharp-queue.html) | 它代表了一个**先进先出**的对象集合。当您需要对各项进行先进先出的访问时，则使用队列。当您在列表中添加一项，称为**入队**，当您从列表中移除一项时，称为**出队**。 |
| [点阵列（BitArray）](https://www.runoob.com/csharp/csharp-bitarray.html) | 它代表了一个使用值 1 和 0 来表示的**二进制**数组。当您需要存储位，但是事先不知道位数时，则使用点阵列。您可以使用**整型索引**从点阵列集合中访问各项，索引从零开始。 |



## 泛型

同java. 不指定类型，

```c#
public class MyGenericArray<T>
    {
        private T[] array;
        public MyGenericArray(int size)
        {
            array = new T[size + 1];
        }
        public T getItem(int index)
        {
            return array[index];
        }
        public void setItem(int index, T value)
        {
            array[index] = value;
        }
    }
```

## 多线程





# unity脚本

## 生命周期

从上到下依次执行

|    名称     |              触发时机              |                             用途                             |
| :---------: | :--------------------------------: | :----------------------------------------------------------: |
|    Awake    |        脚本实例被创建时调用        | 用于游戏对象的初始化，注意Awake的执行早于所有脚本的Start函数 |
|  OnEnable   |  当对象变为可用或激活状态时被调用  |                             用途                             |
|    Start    |    Update函数第一次运行之前调用    |                     用于游戏对象的初始化                     |
|   Update    |            每帧调用一次            | 用于更新游戏场景和状态（受机器性能影响。所以不能用于更新恒定的事情，比如固定速度跑步）//渲染更新 |
| FixedUpdate |    每个固定物理时间间隔调用一次    |                      用于物理状态的更新                      |
| LateUpdate  |  每帧调用一次（在update之后调用）  |     用于更新游戏场景和状态，和相机有关的更新一般放在这里     |
|    OnGUI    |        渲染和处理OnGUI事件         |                             用途                             |
|  OnDisable  | 当前对象不可用或非激活状态时被调用 |                             用途                             |
|  OnDestroy  |        当前对象被销毁时调用        |                             用途                             |

生命周期图

![img](D:\gitubDATA\Study\docs\unity\images\生命周期.png)



## 常用API

F12快捷键查看父类

### Component

组件

```c#
//获取所有组件
var allComponent = this.GetComponents<Component>();
foreach(var item in allComponent){...}

//获取指定的组件
//修改材质的颜色为红色
this.GetComponent<MashRenderer>().material.color = Color.red;
//获取后代物体的指定组件
var mashRenders = this.GetComponentsInChildren<MashRenderer>();
//获取父级物体的指定组件
this.GetComponentsInParent<xxx>();
```



### Transform

提供了查找，变换组件，改变位置、角度、大小的功能

Translate： 根据向量移动

```c#
//根据自身的位置，移动(x,y,z)向量
//通常最后一个参数可以不写。默认是自己
this.transform.Translate(x,y,z,Space.Self);
//根据世界坐标轴，移动(x,y,z)
this.transform.Translate(x,y,z,Space.World);
```

Rotate：旋转 

```c#
this.transform.Rotate(0,10,0); //沿自身坐标系y轴旋转10°
```



RotateAround：围绕旋转（有一个中心点）

```c#
this.transform.RotateAround(Vector3.zero,Vector3.forward);
```

获取/设置/查找

```c#
//获取父级
Transform tf = this.transform.parent();
//获取根部
Transform tff = this.transform.root();
//设置父物体
this.transform.SetParent(tf);
//查找子物体 根据物体名称
this.transform.Find("name");
//查找子物体 根据索引index
int count = this.transform.childCount; //子物体个数
for (int i=0; i<count; i++){
    Transform temp = this.transform.GetChild(i);
}
```



### GameObject

游戏物体：Hierarchy面板中的所有项都是GameObject

一般用于修改面板中的选项，比如禁用/启用，是否静态，设置静态，添加Component等。(AddComponent(xx))

### Object

移除Component： Destroy(gameObject/component/xx, 5);  //5代表什么时间之后消失

### Time

deltaTime ：单位时间 ΔTime

还有一个unscaledDelataTime  //不受暂停timeScale的影响

```c#
private void Update(){
    //增加deltaTime可以保证不受机器性能影响。以及渲染影响。
    // 旋转速度*每帧消耗时间。 相当于z = x*y;  机器好x大,则z大。但是加了deltaTime后就平衡了，互补了。
    this.transform.Rotate(0,1*Time.deltaTime,0);
}
```

timeScale：暂停0/启动1游戏

```c#
Time.timeScale = 0;//暂停游戏
Time.timeScale = 1;//恢复游戏
```

