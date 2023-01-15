---
title: 戴森球计划BepInEx模组开发日志
categories:
  - 编程
tags:
  - C#
  - BepInEx
  - Unity
date: 2023-01-08 00:04:35
---

​	最近又把戴森球计划(Dyson Sphere Program)下回来了，并且发现有个叫创世之书模组挺有意思的。了解到现在模组作者都苦于游戏的数字ID，决定尝试一下能不能解决这个问题。解决问题第二步，选择框架并尝试进行模组开发。

<!--more-->

**以下部分摘抄xiaoye97版本的BIE教程**

# 开始编写插件 

我们给默认的Class1修改一个我们想要的名字，我这里改为PluginTutorial，然后将BepInEx的命名空间using一下。

在BepInEx中，给我们准备了一个类，BaseUnityPlugin，这是继承于MonoBehaviour的，也就是说，我们的插件，最终会以组件的形式挂载，实际上也是这样，每个插件最终都会挂载到游戏中BepInEx的物体身上。所以我们可以使用MonoBehaviour的各种生命周期，比如Awake，Start，Update等等，这些我们以后再说，先来看一下最基础的插件的样子。



```csharp
using System;
using BepInEx;

namespace PluginTutorial
{
    //插件描述特性 分别为 插件ID 插件名字 插件版本(必须为数字)
    [BepInPlugin("me.xiaoye97.plugin.Tutorial", "Tutorial Plugin", "1.0")]
    public class PluginTutorial : BaseUnityPlugin //继承BaseUnityPlugin
    {
        //Unity的Start生命周期
        void Start()
        {
            //输出日志
            Logger.LogInfo("HelloWorld!");
        }
    }
}
```

我们将PluginTutorial继承BaseUnityPlugin，并在类上方添加了一个叫做BepInPlugin的特性，这是必须要完成的，只有这样才能正确加载插件。

如果你不知道特性是什么，可以去补一下C#关于特性方面的知识。

在BepInPlugin特性中，我们填入了3个参数，分别是插件的ID，插件的名字，插件的版本号，对于ID，我个人习惯使用域名反写法，一般是me.xiaoye97.plugin.游戏名.插件名，插件的名字没有什么特别的要求，直观即可。在插件版本这里，必须是数字形式的版本号，不能夹杂字母等。

# 其他事项 

插件的特性除了BepInExPlugin之外，还有两个可能会用到的特性。

第一个是BepInProcess特性，大部分情况下，我们不需要写这个特性，但是偶尔会遇到特殊情况。比如，在I社游戏(例如AI少女、恋爱活动等)中，不仅有游戏本体，还有一个工作室程序，将游戏本体与内容创作进行了分割，这样，就会有两个exe，但是，他们是两个不同的exe，有很多地方是不能公用的。那么，只需要用这个特性，就可以限制插件在指定的exe上可以运行。
例如

```csharp
[BepInPlugin("me.xiaoye97.plugin.Tutorial", "Tutorial Plugin", "1.0")]
[BepInProcess("Maid In Makai.exe")]
public class PluginTutorial : BaseUnityPlugin
{

}
```

这样，就是限定只在这个exe中运行，如果想限制在几个exe中可以运行，就继续添加这样特性即可。

第二个是BepInDependency特性，如果我们的插件，需要以其他的什么插件为前置插件，那么就需要使用这个特性添加依赖，以保证只有在有前置插件的情况下加载我们的插件。

BepInDependency特性有3种写法，分别是

```csharp
[BepInPlugin("me.xiaoye97.plugin.Tutorial", "Tutorial Plugin", "1.0")]
// 软依赖，如果没有前置插件，依旧继续加载
[BepInDependency("com.bepinex.plugin.somedependency", BepInDependency.DependencyFlags.SoftDependency)]
// 硬依赖，如果没有前置插件，则停止加载
[BepInDependency("com.bepinex.plugin.importantdependency", BepInDependency.DependencyFlags.HardDependency)]
// 省略参数，则默认为硬依赖
[BepInDependency("com.bepinex.plugin.anotherimportantone")]
public class PluginTutorial : BaseUnityPlugin
{

}
```


除了这些特性之外，还有一点我们需要注意的是，一个dll中可以包括多个插件，只要我们写多个继承BaseUnityPlugin的类，并为他们赋予BepInPlugin特性即可。

# ConfigEntry<T>

在插件功能的设计中，经常会有需要玩家自己配置的东西，比如插件的各种设置，快捷键的分配等。在BepInEx中，提供了一个ConfigEntry类简化了配置操作。

我们来看一段示例：

(注:由于阿B在代码页中会删除尖括号，所以我使用空格隔开)

```csharp
using BepInEx;
using BepInEx.Configuration; //ConfigEntry的命名空间

namespace PluginTutorial
{
    [BepInPlugin("me.xiaoye97.plugin.Tutorial", "Tutorial Plugin", "1.0")]
    public class PluginTutorial : BaseUnityPlugin
    {
        ConfigEntry<int> intConfig;
        ConfigEntry<string> stringConfig;

        void Start()
        {
            //绑定配置文件
            intConfig = Config.Bind<int>("config", "TestInt", 10, "测试用Int");
            stringConfig = Config.Bind<string>("config", "TestString", "Hello", "测试用String");

            //使用配置文件中的值
            Logger.LogInfo(intConfig.Value);
            Logger.LogInfo(stringConfig.Value);
        }
    }
}
```

Config是BaseUnityPlugin的成员，是每个插件都自带的，通过这个Config进行绑定时，会自动以插件ID为文件名生成配置文件，如果你需要多个配置文件，可以手动创建ConfigFile对象。

绑定时有4个参数，分别是 分类 Key 默认值 描述。分类就是这个配置在哪个标签下，比如我们之前打开控制台窗口的时候，是在[Logging.Console]下，Key则是这个配置的名字，比如打开控制台时的Enabled，默认值则是在没有配置文件的情况下，创建配置文件时使用的值，描述可填可不填，主要是提醒玩家这个配置的用处是什么。

# 使用ConfigurationManager在游戏运行时修改配置文件

BepInEx有多个非常实用的通用插件，放在任何游戏都可以使用，本章介绍ConfigurationManager插件，它可以在游戏内可视化的修改配置文件。

下载地址：https://github.com/BepInEx/BepInEx.ConfigurationManager/releases

安装插件后，在游戏中按F1打开配置管理界面，修改即可。

# 前言 

通过之前的教程，我们已经知道如何编写基本的插件，如果你有C#和Unity的基础，这个时候已经可以做出一些功能了，比如通过按键修改游戏数据之类的。但是，这有很大的局限性，因为通常情况下，我们并不想通过按键来调用我们的功能，我们想让大多数的功能都是加载之后就不需要管了，或者想做一些普通情况下比较难以操作的事情。这个时候，通过Harmony进行补丁可以解决我们绝大多数的需求。

Harmony的github链接 https://github.com/pardeike/Harmony 详细信息可以在github查看。

Harmony中使用最频繁的两个地方就是前置补丁和后置补丁，也是最简单的，本篇文章主要就讲这两种。一些特殊的需求需要修改函数本身也是可以的，Harmony支持修改函数的IL码，不过这个就不在基础的范畴了，以后有机会的话会放在进阶篇来讲。

# HarmonyPatch特性

要对游戏中的方法进行补丁，首先我们需要确定一个目标，这里我准备了一个类，我们就以这个类为例子，对它进行补丁。



```csharp
public class People
{
    public string Name{ get; set; };
    public int Age{ get; set; };

    public People(string name, int age)
    {
        Name = name;
        Age = age;
    }

    public void Sleep()
    {
        Console.WriteLine("睡觉");
    }

    public void Say()
    {
        Console.WriteLine("Hello");
    }

    public void Say(string content)
    {
        Console.WriteLine(content);
    }

    public void Say(int content)
    {
        Console.WriteLine(content);
    }
}
```

这是一个简单的People类，有两个属性，分别是姓名和年龄，一个构造函数，一个Sleep方法，还有3个说话的方法，使用了3种重载。

以Sleep方法为例，我们写一个最简单的补丁。

```csharp
[HarmonyPatch(typeof(People), "Sleep")]
class PeopleSleepPatch
{
    public static void Postfix(People __instance)
    {
        Console.WriteLine(__instance.Name + "睡觉了");
    }
}
```

这里的HarmonyPatch特性就用于确定补丁目标，这个特性的参数可以写在一排也可以分成几排写。例子中的两个参数分别是要补丁的类型，还有要补丁的方法的名字。这是最简单的情况，实际上我们还经常会遇到其他几种情况。比如，Say方法有3个重载，如何确定要补丁哪一个？属性要怎么补丁？我们再来看几个例子。



```csharp
[HarmonyPatch(typeof(People), "Say", new Type[] { })]
[HarmonyPatch(typeof(People), "Say", new Type[] { typeof(string) })]
[HarmonyPatch(typeof(People), "Say", new Type[] { typeof(int) })]
```

如此，面对有重载的情况，我们只需要在添加一个参数，这个参数是一个Type数组，我们按顺序将参数类型填入即可。

```csharp
[HarmonyPatch(typeof(People), "Name", MethodType.Getter]
[HarmonyPatch(typeof(People), "Age", MethodType.Setter]
[HarmonyPatch(typeof(People), MethodType.Constructor]
```

面对属性和构造函数，我们可以使用MethodType枚举来当作参数。需要注意的是，补丁构造函数时，函数名不能写dnSpy中看到的.ctor，而是应该直接省略不写函数名。

# 补丁方法 

既然已经可以确定补丁目标了，接下来让我们了解一下最基础最常用的两种补丁方法，Prefix(前置补丁)、Postfix(后置补丁)。

先说后置补丁，这是最简单的，它在补丁目标运行结束之后运行，上面示例中的就是后置补丁，可以使用__result参数接收目标的返回值。

然后是前置补丁，顾名思义，它是在补丁目标运行之前运行的，这个相对复杂一点。因为我们可以选择是否执行原方法。我们来看两个例子。

```csharp
[HarmonyPatch(typeof(People), "Say",  new Type[] { typeof(string) })]
class PeopleSayPatch
{
    public static bool Prefix(ref string content)
    {
        content = "要说的内容已被修改";
        return true;
    }
}

[HarmonyPatch(typeof(People), "Name",  MethodType.Getter)]
class PeopleNamePatch
{
    public static bool Prefix(ref string __result)
    {
        __result = "张三";
        return false; //拦截原方法，直接使用我们给出的结果
    }
}
```

第一个例子，我们将content的值修改为了我们自己想要的值，然后返回true表示让原函数继续执行。第二个例子，我们直接将最终结果修改，然后返回false，表示阻止原函数执行。如果你搞不明白IL代码，不知道如何修改函数本体，也可以通过前置补丁的方式自己计算结果然后修改。

# 补丁参数 

在上面的例子中，我们有时候使用了__instance，有时候使用了__result，想必读者还留有疑问，为什么要这么写。其实，这是Harmony作者为我们定好的获取方法信息的方式。

大概情况如下:

- 补丁方法**必须**是**静态**方法
- Prefix需要返回**void**或者**bool**类型(void即不拦截)
- Postfix需要返回**void**类型，或者返回的类型要与**第一个**参数一致(直通模式)
- 如果原方法不是静态方法，则可以使用名为**__instance**(两个下划线)的参数来访问对象实例
- 可以使用名为**__result**(两个下划线)的参数来访问方法的返回值，如果是Prefix，则得到返回值的默认值
- 可以使用名为**__state**(两个下划线)的参数在Prefix补丁中存储任意类型的值，然后在Postfix中使用它，你有责任在Prefix中初始化它的值
- 可以使用与原方法中**同名的参数**来访问对应的参数，如果你要写入非引用类型，记得使用ref关键字
- 补丁使用的参数必须**严格对应**类型(或者使用object类型)和名字
- 我们的补丁只需要定义我们需要用到的参数，不用把所有参数都写上
- 要允许补丁重用，可以使用名为**__originalMethod**(两个下划线)的参数注入原始方法

Transpilers还有一些可选参数，我们这里不做探讨，想了解可以访问Harmony的wiki。

# 自动补丁 

补丁的情况我们大体介绍完了，但是我们现在只是写了补丁，还没有对游戏进行补丁，其实很简单，我们只要在插件加载的时候，加上一句代码就好。



```csharp
new Harmony("me.xiaoye97.plugin.Tutorial").PatchAll(); //以作者输入的字符串作为ID，对程序集中所有找到的补丁方法进行补丁。
```

或者

```csharp
Harmony.CreateAndPatchAll(typeof(PluginTutorial)); //Harmony以类名为ID进行补丁，并且只补丁此类下的方法。
```

除了自动补丁之外，还可以进行手动补丁，可以更加细微的控制，就不在基础教程中说了，读者可以通过GitHub继续了解，以后的教程如果遇到需要手动补丁的情况我再继续讲解。

# 后话 

对于新接触BepInEx或者UnityModManager Mod开发的人来说，Harmony可能比较陌生，如果看了本篇教程之后还是感觉不太好下手，可以使用dnSpy反编译其他人制作的插件，或者如果他们有开源的话，可以直接查看开源的代码进行学习模仿。我在我的Github上也放了几个小插件，大家可以查看学习。

我的Github地址：https://github.com/xiaoye97

------

23.1.8 02:41

突然发觉给DSP换个英文ID是非常大的工程

并且我可能需要一套完整的解决方案和处理流程

例如ABN_RecipeUnlockCondition.cs中

```
if (recipeProto.preTech == null || this.gameData.history.TechUnlocked(recipeProto.preTech.ID))
```

的使用，显然this.gameData.history.TechUnlocked传参为int

```
 public bool TechUnlocked(int techId) => this.techStates.ContainsKey(techId) && this.techStates[techId].unlocked;
```

如果我想更改为英文ID，这些判定有关的需要全部从做

===

[事件函数的执行顺序 - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2020.3/Manual/ExecutionOrder.html)

Unity脚本生命周期

===

**以下部分为3DM MOD版教程**

一个示例

```c#
using System;
using BepInEx;
using UnityEngine;

namespace MyFirstBepInExMod
{
    [BepInPlugin("aoe.top.plugins.MyFirstBepInExMod", "这是我的第一个BepIn插件", "1.0.0.0")]
    public class MyFirstBepInExMod : BaseUnityPlugin
    {
        // 在插件启动时会直接调用Awake()方法
        void Awake()
        {
            // 使用Debug.Log()方法来将文本输出到控制台
            Debug.Log("Hello, world!");

        }

        // 在所有插件全部启动完成后会调用Start()方法，执行顺序在Awake()后面；
        void Start()
        {
            Debug.Log("这里是Start()方法中的内容!");
        }

        // 插件启动后会一直循环执行Update()方法，可用于监听事件或判断键盘按键，执行顺序在Start()后面
        void Update()
        {
            var key = new BepInEx.Configuration.KeyboardShortcut(KeyCode.F9);

            if (key.IsDown())
            {
                Debug.Log("这里是Updatet()方法中的内容，你看到这条消息是因为你按下了F9");
            }
        }
        // 在插件关闭时会调用OnDestroy()方法
        void OnDestroy()
        {
            Debug.Log("当你看到这条消息时，就表示我已经被关闭一次了!");
        }
    }
}
```

### HarmonyPrefix

HarmonyPrefix是Harmony为我们提供的一个接口，它将在我们指定的函数前进行执行，并且我们可以返回一个bool值来控制是否继续继续执行游戏原函数；
需要配合HarmonyPatch一起使用

```
// 19 个重载
public HarmonyPatch();
public HarmonyPatch(Type declaringType);
public HarmonyPatch(MethodType methodType);
public HarmonyPatch(string methodName);
public HarmonyPatch(Type[] argumentTypes);
public HarmonyPatch(MethodType methodType, params Type[] argumentTypes);
public HarmonyPatch(string methodName, MethodType methodType);
public HarmonyPatch(string methodName, params Type[] argumentTypes);
public HarmonyPatch(Type[] argumentTypes, ArgumentType[] argumentVariations);
public HarmonyPatch(Type declaringType, MethodType methodType);
public HarmonyPatch(Type declaringType, string methodName);
public HarmonyPatch(Type declaringType, Type[] argumentTypes);
public HarmonyPatch(Type declaringType, string methodName, MethodType methodType);
public HarmonyPatch(string methodName, Type[] argumentTypes, ArgumentType[] argumentVariations);
public HarmonyPatch(Type declaringType, string methodName, params Type[] argumentTypes);
public HarmonyPatch(MethodType methodType, Type[] argumentTypes, ArgumentType[] argumentVariations);
public HarmonyPatch(Type declaringType, MethodType methodType, params Type[] argumentTypes);
public HarmonyPatch(Type declaringType, string methodName, Type[] argumentTypes, ArgumentType[] argumentVariations);
public HarmonyPatch(Type declaringType, MethodType methodType, Type[] argumentTypes, ArgumentType[] argumentVariations);
public HarmonyPatch(string assemblyQualifiedDeclaringType, string methodName, MethodType? methodType = null, Type[] argumentTypes = null, ArgumentType[] argumentVariations = null);
```

如：
我们想要对Mecha类下的SetForNewGame函数进行拦截，那么就是：

```
[HarmonyPrefix]
[HarmonyPatch(typeof(Mecha), "SetForNewGame")]
public static bool Mecha_SetForNewGame_Prefix()
{
    // 这里写入我们自己的内容            
    Debug.Log("这里的内容将会在游戏函数执行前进行执行");
    // 返回 true为继续执执行游戏原函数，返回 false为不执行游戏原函数,
    return true;
}
```

### HarmonyPostfix

HarmonyPostfix一样也是Harmony为我们提供的一个接口，它将在我们指定的函数执行完毕后，再执行。
一样需要配合HarmonyPatch一起使用

如：

```
[HarmonyPostfix]
[HarmonyPatch(typeof(Mecha), "SetForNewGame")]
public static void Mecha_SetForNewGame_Postfix()
{
    // 这里写入我们自己的内容            
    Debug.Log("这里的内容需要等待游戏原函数执行完后才会执行");
}
```

> 注意：
> 1.我们的函数需要使用static静态函数，不然会报错；
> 2.函数名可以自定义，但尽量不要和游戏原有函数冲突；
> 3.两种拦截方式大同小异，希望大家举一反三。

### this

在游戏原函数中难免会出现this参数，万能的Harmony当然也考虑到了这一点，针对于this，我们可以向函数中传递一个__instance。

游戏原函数内容：

```
public void SetForNewGame()
{
    ModeConfig freeMode = Configs.freeMode;
    this.coreEnergyCap = freeMode.mechaCoreEnergyCap;
    this.coreEnergy = this.coreEnergyCap;
    this.corePowerGen = freeMode.mechaCorePowerGen;
    this.reactorPowerGen = freeMode.mechaReactorPowerGen;
    this.reactorEnergy = 0.0;
    this.reactorItemId = 0;
}
```

我们可以这样写：

```
[HarmonyPostfix]
[HarmonyPatch(typeof(Mecha), "SetForNewGame")]
public static void Mecha_SetForNewGame_Postfix(Mecha __instance)
{
    ModeConfig freeMode = Configs.freeMode;
    __instance.coreEnergyCap = freeMode.mechaCoreEnergyCap;
    __instance.coreEnergy = __instance.coreEnergyCap;
    __instance.corePowerGen = freeMode.mechaCorePowerGen;
    __instance.reactorPowerGen = freeMode.mechaReactorPowerGen;
    __instance.reactorEnergy = 0.0;
    __instance.reactorItemId = 0;
}
```

> 注释：
>
> - Mecha` __instance`中，`Mecha` 为当前类的名称，`__instance`为Harmony的固有写法（有两个“_”）；
> - 这种方法只限于操作公共public变量和函数；

### 游戏私有变量

刚刚提到，“__instance”只能获取游戏的公共变量和方法，如果我们要获取游戏中私有的变量和方法的话，就需要用到Traverse工具；
我们可以通过Traverse工具,方便访问游戏里所有公有,私有,受保护的变量,方法,以及属性,

如我们想获取游戏中的变量，那么在我们的插件中就可以这么写：

```
[HarmonyPostfix]
[HarmonyPatch(typeof(Mecha), "SetForNewGame")]
public static void Mecha_SetForNewGame_Postfix(Mecha __instance)
{
    // 获取 private float _dronesSpeed; 的值
    var _droneCount= Traverse.Create(__instance).Field("_droneCount").GetValue();
}
```

### 拓展知识

> 来自 https://bbs.3dmgame.com/thread-5870433-1-1.html
> **关于Traverse的使用:**
> Traverse是harmony类库下的一个工具类,也就是在一开始引用的using harmony;这条语句后,方便我们使用的一个类,首先我们要明白private和public还有protected三个关键词的区别,具体可以百度,我这里仅从结论讲明,除了public,其他的private和protected从外界是无法访问到的,但是用Traverse类不管它是public,private,protected,均可以强行访问,为什么不任何地方都使用Traverse去访问呢,因为性能问题,用Traverse要走映射,简单来说运行速度会有些许影响

> Traverse的具体使用方法简单的来说明一下,Traverse.create(类的实例),表名我要将一个类的实例转为Traverse对象,简单来说就是附加功能,比如我们以前都是自己买菜,后来有了XX外卖,我们不需要亲力亲为了,XX外卖就等于Traverse对象了(这里就是将映射功能简单化了,不需要自己打代码了),这样我们就有一个可以访问类实例的Traverse对象了,在上面法宝的例子中,我是直接写为了
> var itemID = Traverse.Create(__instance).Field("itemID").GetValue();
> 这是一种简化的写法的,下面我分开并且逐步注释一下
> Traverse t = Traverse.Create(__instance);//根据__instance (ToilRefining类的实例) 创建 Traverse对象,并且用t表示
> Traverse f = t.Field("itemID");//在ToilRefining实例里面有个itemID的字段,找到他并且创建一个Traverse对象,用f表示,这样可以强行访问 itemID,因为itemID是私有的没法直接访问
> int itemID = f.GetValue();//将Traverse版本的itemID提取成可以直接访问的数值,因为Traverse并不知道原本itemID是什么类型的,所以我们要用标注这是个int类型了,从源代码中我们可以知道itemID的变量类型,对应修改即可
> 于是我们就访问到itemID了
> 既然有获取,自然就有修改,修改我们可以用f.SetValue(数值),这里就不需要指定了,因为你在输入数值的时候,他会自动把你输入的数据转成对应的类型

> 这里我说一下字段,属性,方法的意思,这是C#的基础,字段代表类变量,可以理解为类中的全局变量,可以再类中任意地方访问到

> 属性是字段的升级版,他在源代码中的样子是这样的

> 他跟字段的定义差不多,但是后面会有括号,里面还有set和get,这种样子的就是属性,我们不能通过Traverse.Field(字段名字),而是通过Traverse.Property(属性名字)来访问,如果定义中只有get,表名这个东西只能获取,不能更改(就是游戏开发者也不能),get和set都在就是可以获取也可以更改

> 最后就是方法,在C#中称为方法,C语言中称为函数,比如游戏源码中,MakeFaBao就是制作法宝的方法,他定义时后面跟随的是()这种括号,我们想要访问游戏private的方法可以用Traverse.Method(方法名字).GetValue()来运行,注意后面要加上.GetValue(),因为仅仅Traverse.Method(方法名字)是获取的方法的Traverse对象,而没有运行他

> C#中有一个关键词是var,这个关键词是这个变量是智能根据你后面赋值来判断他的变量类型的
> 比如 var a = 6;//a是int类型
> var b = "我是文字";//b就是string类型的
> 于是之前为了itemID那么多行的代码就可以省略为var itemID = Traverse.Create(__instance).Field("itemID").GetValue();一句话搞定
> 当然也可以var t = Traverse.Create(__instance);
> var itemID = t.Field("itemID").GetValue();var XXXX = t.Field("XXXX").GetValue();
> 来多次获取
