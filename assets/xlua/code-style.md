> 作者：wsh 
> 时间：2017-12-01
> [原文](https://github.com/smilehao/xlua-framework/blob/master/Assets/LuaScripts/%E4%BB%A3%E7%A0%81%E8%A7%84%E8%8C%83.txt)

# 一、命名规范
1. 类命名：驼峰式命名：单词首字母大写，如：GetInstance
2. 函数命名：同类名
3. 公有变量命名：同类名
4. 私有变量命名：小写，单词之间用“_”分隔，如：self.action_list
5. 局部变量命名：同私有变量
6. 参数名命名：同私有变量
7. 任何情况下不应该由外部访问的成员，使用双下划线打头，其它同私有变量命名，如：析构函数__init，内部成员self.__callback
8. 由于脚本语言没有跳转功能，最好在UI组件实例的名字末尾标识组件类型，提高可读性：
	- 基础组件（UIBaseComponent）：xxx_cmp
	- 按钮（UIButton）：xxx_btn
	- 文本（UIText）：xxx_txet
	- 图片（UIImage）：xxx_img
	- 输入框（UIInput）：xxx_input
	- 标签组（UITabGroup）：xxx_tabgroup
	- 按钮组（UIButtonGroup）：xxx_btngroup
	- 可选中按钮（UIToggelButton）：xxx_togglebtn
	- 可复用组件（UIWrapGroup）：xxx_wrapgroup
	- 滑动条组件（UISlider）：xxx_slider
	- 后续...
9. 所有UI脚本以UI打头，即UIxxxx
10. 系统功能扩展函数：全部使用小写，不用下划线，如对table的扩展：table.walksort
11. 所有协程函数体以"Co"打头，如：CoAsyncLoad，表示该函数必须运行在协程中，并且可以使用任意协程相关函数
12. 所有Unity Object均使用全局函数IsNull判空===>***很重要
13. 所有热修复脚本放XLua目录下，由于以前写热修复脚本命名习惯沿用了XLua作者的命名习惯，现在不再去动它

# 二、类定义和使用
1. 所有函数定义为local，在脚本最底部导出，导出的函数一定是公有的
2. 所有公有函数第一个参数是self，函数使用调用：instance:function(...)
3. 所有私有函数第一个参数是self，不导出，只能在脚本内访问，函数调用：function(self, ...)
4. 所有私有函数一定要先定义，后调用
5. override的使用有点特殊：先用base = baseClassType，然后override时使用：base.function(self)调用父类方法
6. 继承类时，如果不是等同于cs侧sealed的概念，那么必须把基类的参数列表填写完整，后面接上自己需要的参数
7. 函数需要重载时，一般通过判断参数个数和类型来实现，此时必须把最长参数列表填齐，除了回调绑定等不定参数的特俗情况，一般情况下不要使用可变参数（...）
9. 所有定义回调的地方，都需要预先声明回调和注释说明回调原型，让使用者一目了然
10. __init不需要调用base.__init，底层会自动调用基类构造函数，__delete也一样

# 三、单例类定义和使用
1. 单例类从Singleton继承，不要重写GetInstance.Delete方法
2. 单例类定义时内部函数书写规范同上：类定义和使用
3. 单例类调用一律使用singletonClass:GetInstance():function/.var访问
4. 除了局部变量，不要使用成员变量或者全局变量缓存单例类引用，如：inst = singletonClass:GetInstance(),inst:function/.var，因为单例类销毁后inst还会存在引用
5. 单例类的Instace只用来查询该单例类是否已经被创建，如：if singletonClass.Instance ~= nil then singletonClass:Delete() end

# 四、数据类定义和使用
1. 数据类：对普通类增加访问限制，具体为：不能对不存在的域进行读写。目的：减少因为笔误而造成的不可察Bug
2. 定义格式使用：DataClass("dataClassName", dataTable)
3. dataTable是一张普通表，定义了该数据成员的域，必须初始化，不能有nil值
4. 定义以后不能新增数据域，访问不存在的域会提示错误，New新的数据实例同New新的类实例

# 五、常量类定义和使用
1. 常量类：对普通类增加访问限制，具体为：不能对不存在的域进行读写，数据域只读，不可写。目的：减少因为笔误而造成的不可察Bug
2. 定义格式使用：ConstClass("constClassName", constTable)，一般用于配置表等数据，一旦生成只能查表，不能写表
3. 定义以后查询不存在的域. 写不存在的域. 写存在的域都会有错误提示

# 六、UI窗口代码规范
1. 严格遵守MVC架构：Model层数据. View层窗口组件操作. Ctrl层数据操作
2. View层直接依赖Ctrl层，间接依赖Model层（只读）；Ctrl层依赖Model层；Model层不依赖Ctrl和View层
3. Ctrl层没有状态，可以操作游戏逻辑和Model层数据；View层除了读取配置表，不能直接操作任何游戏逻辑
4. 逻辑的运行不能依赖窗口的Ctrl层，如果需要这样的控制代码，写到游戏逻辑模块中
5. 窗口Model层不存游戏数据，它的生命周期是和窗口绑定在一起的，只能缓存用户操作，比如：当前选择了那个服务器做登陆服务器
6. 窗口Model层是针对窗口的数据，是游戏数据中心的一个抽取，比如数据中心UserData可能包括用户名. 背包. Vip. 英雄等等数据，但是用于界面可能只是从用户名. Vip提取部分数据展示

# 七、UI组件代码规范
1. 所有需要调度和管理的UI组件最好使用Lua侧封装的各种UIComponent，不要直接使用Unity侧的UI原生组件，否则不受Lua侧组件系统调度管理，需自行管理
2. 原则上尽量对UI组件执行封装：一是可以简化逻辑层脚本使用方式，二是可以利用缓存尽量减少lua与cs交换，提升性能
3. 原则上游戏逻辑代码中（包括窗口View层代码）不对UI组件做任何假设，即不假设Unity侧使用的是NGUI还是UGUI
4. 虽然目前这套框架是针对UGUI编写，但是如果要扩展（或者要替换插件），只需要另外针对NGUI写一个Lua侧的各种UIComponent
5. 所有UI组件最好先封装，后使用，以尽量使用Lua侧组件管理系统来简化写View层脚本的工作量，各个使用到的组件现在还不是很完善，后续...
6. 一个窗口（window.view）下的所有组件持有对窗口view脚本的引用，方便访问window.view，或者window.model层数据
7. 当设计通用组件时，不能直接依赖window.view，需要数据刷新最好提供函数回调
8. UI组件代码所有函数的执行规律同Unity脚本，UI组件代码不要使用__init. __delete函数，由OnCreate. OnDestory代替
9. 最好不要自己去New组件，使用AddComponent替代，否则必须自己管理生命周期---在OnDestory中调用组件的Delete方法

# 八、工具类代码规范
1. 所有和UI界面相关的公共函数添加到UIUtil
2. 所有和Lua语言直接相关的公共函数添加到LuaUtil
3. 所有对table操作的扩展函数添加到TableUtil
4. 其它待续...

# 九、框架代码规范
1. 原则：保证框架代码的可迁移，如果需要迁移到新项目，可以不修改任何代码，或者修改很少的粘合代码即可使用
2. 如果需要完善框架，框架内的代码理论山不要牵涉任何游戏逻辑，一般只提供管理类和基类，和业务相关的子类不要放在框架中

# 十、性能
1. 性能瓶颈出现在两点：lua作为脚本语言本身的速度问题. lua与cs的频繁交互造成高频率堆栈操作和Marshall操作
2. 原则1：单次调用，内部执行性能要求高的函数，比如寻路计算，考虑放CS侧，或者用C/CPP写---要求：函数执行时间要高于lua与cs调用交互时间，否则得不偿失
3. 原则2：尽量避免与cs的交互，交互越少越好，如tolua作者对Vector3在lua侧的实现，就是为了避免Vector3操作调用cs接口，其它实现的数据结构类似
4. 不要使用cs侧协程，lua这边我已经实现了一套，Unity支持的所有协程功能这里都支持，而且进行了很大的性能优化
5. 更新频率低的函数（如UI界面倒计时）使用定时器，尽量不要用Updater
6. 虽然Lua采用分步GC，不需要太关注GC造成游戏Lag的问题，但是分配. 回收频率很高的table，还是要做缓存，参考定时器管理模块
