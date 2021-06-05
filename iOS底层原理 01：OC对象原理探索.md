
# alloc对象的指针地址和内存
在开始研究之前，我们先创建一个工程，然后新建一个类`Person`，然后我们看一下下边代码的结果
```OC
Person *p1 = [Person alloc];
Person *p2 = [p1 init];
Person *p3 = [p1 init];
    
NSLog(@"%@--%p--%p", p1, p1, &p1);
NSLog(@"%@--%p--%p", p2, p2, &p2);
NSLog(@"%@--%p--%p", p3, p3, &p3);
```
最终打印结果为:
```
<Person: 0x2809ec5b0>--0x2809ec5b0--0x16f9f1b58
<Person: 0x2809ec5b0>--0x2809ec5b0--0x16f9f1b50
<Person: 0x2809ec5b0>--0x2809ec5b0--0x16f9f1b48
```
我们发现结果一摸一样，由此我们可以得出以下结论：\
1. `alloc`会创建一块内存
2. `init`不会对当前的指针做任何操作
3. `p1` `p2` `p3`三个指针地址为连续的，并且三个地址不一样的指针，指向了同一块内存空间
**如下图所示：**
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45fcf384be0c4e96b145e3d0b6cf9a2d~tplv-k3u1fbpfcp-zoom-1.image)
> 那么，`alloc`是如何开辟内存的? `init`真的什么都不做么？那么它有何用处？
要想解答这些问题，我们需要研究`alloc`的底层实现，但是我们点击发现我们无法查看`alloc`的具体实现，那么我们就要想其他方法来探索`alloc`底层究竟是如何实现的。
# 底层探索的三种方法
- 1、`control`+step into
- 2、符号断点定位查看调用流程
- 3、汇编查看调用流程
## `control`+step into
### 1、我们在`Person *p1 = [Person alloc];`代码处下断点，然后运行项目
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efeaec99d8bd4d89be006fefd2716b0f~tplv-k3u1fbpfcp-zoom-1.image)
### 2、此时，我们按住`control`键，然后点击`step into`按钮
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa423ca72f3b47a390eb55f5d8404ee9~tplv-k3u1fbpfcp-zoom-1.image)

此时，我们来到了`objc_alloc`方法，**下图为真机演示，模拟器有所区别不做展示**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e85ae67f1b145f19a1d3309e10563c9~tplv-k3u1fbpfcp-zoom-1.image)

继续下一步我们也无法获取更多信息，此时，我们需要添加一个`objc_alloc`的符号断点
## 符号断点定位查看调用流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b97732d3ffbd4805ae4e95e361b93583~tplv-k3u1fbpfcp-zoom-1.image)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0bbdf6d6eeb549cabab178cdb9cfdec3~tplv-k3u1fbpfcp-watermark.image)

然后继续执行，我们发现进入了`objc_alloc`的实现，然后还知道了接下来将会执行`_objc_rootAllocWithZone`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9d0f0a78ee24c789e17f6aa8e4f9dc9~tplv-k3u1fbpfcp-watermark.image)

我们也可以继续添加`_objc_rootAllocWithZone`的符号断点，继续查看流程

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb889b7b66d84229a7409bba3a31cc1d~tplv-k3u1fbpfcp-watermark.image)

## 汇编查看调用流程
除了以上两种方法以外，我们还有第三中方法，利用汇编查看调用流程(**先删除之前添加的符号断点**)
### 1、设置断点，运行项目
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efeaec99d8bd4d89be006fefd2716b0f~tplv-k3u1fbpfcp-zoom-1.image)
### 2、打开汇编窗口`Debug`->`Debug Workflow`->`Always Show disassembly`
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26493ed78eee4f748238d49a1bcb2987~tplv-k3u1fbpfcp-watermark.image)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9f2e0cfb59b4e4da64ae0c9866ced93~tplv-k3u1fbpfcp-zoom-1.image)
### 3、继续`control`+`step into`，执行到`objc_alloc`里边去
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06e8788ae30049de9e30384023d6283c~tplv-k3u1fbpfcp-zoom-1.image)
> 然后，添加相应的符号断点继续查看调用流程，再执行符号断点的时候，我们可以知道当前符号断点在源码的那个地方
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/731b8f2139984ba1841fa8ef32ee8ce5~tplv-k3u1fbpfcp-watermark.image)

除此之外，我们也可以直接添加`alloc`符号断点调试

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efe72a76361f4ffc805f232b5e02e94b~tplv-k3u1fbpfcp-watermark.image)

> 既然知道了方法在源码中的位置，那么我们就可以通过源码更直接的去探索底层调用流程
# 汇编结合源码调试分析
苹果源码下载地址:\
[Apple Open Source](https://opensource.apple.com)\
[Source Browser](https://opensource.apple.com/tarballs/)

我们这里以`objc4-818.2`版本的源码为例进行探索,我们在上文已经知道了，`[Person alloc]`将会先调用`[NSObject alloc]`方法，那么我们可以在源码中找到此方法的实现的地方:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c6722d666904f3eb22fcf8cb97b173e~tplv-k3u1fbpfcp-watermark.image)

我们一步一步深入之后发现方法调用流程为`alloc`->`_objc_rootAlloc`->`callAlloc`,然后发现在`callAlloc`中，代码出现了判断分支调用

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/663bb782e414407680d851bb22b3ca7b~tplv-k3u1fbpfcp-watermark.image)

此时，我们无法准确判断出代码的调用流程，那么我们可以在工程中，依次添加`alloc`,`_objc_rootAlloc`,`callAlloc`三个符号断点(先将符号断点置为不可用)：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cf72c3965484fc7b63b8ccc9783e78a~tplv-k3u1fbpfcp-watermark.image)

运行项目至`[Peson alloc]`断点处，此时打开三个符号断点：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13e55d85227b4b3f98698ee49c9fe52c~tplv-k3u1fbpfcp-watermark.image)

执行代码，我们发现调用顺序如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75955625f7574ac18cf8bec2373e7cc1~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f5be69450d240cbbec653347a6ffd83~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d338f11a4b234326ac671991ea8afbc4~tplv-k3u1fbpfcp-watermark.image)

> 更深层次的调用，可以自行`step into`

在上述符号断点执行过程中，我们发现`callAlloc`这个符号没有被断点到，这是为什么呢？
> 这里有一个编译器优化的概念
# 编译器优化
>什么叫做编译器优化呢,我们来看一段代码：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55db9607b00043ab97792922078ce939~tplv-k3u1fbpfcp-watermark.image)

此时，我们切换到汇编窗口:

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cd21308fff043e3a4cecbaed201e314~tplv-k3u1fbpfcp-watermark.image)

继续执行汇编代码：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/613d4efcafc14669b93d05e7aeb66e6c~tplv-k3u1fbpfcp-watermark.image)

接下来，我们执行到`sum`函数结束：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff97bd16d5554eda8afcccd7db408d52~tplv-k3u1fbpfcp-watermark.image)

>我们得出结论：经过一系列汇编运算，最终运算结果在`w0(x0)`寄存器中返回\
>接下来我们对编译器进行优化之后，我们再来看这个运算流程

## 设置编译器优化

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7a12464356f4565bb6bf628c762c6f5~tplv-k3u1fbpfcp-watermark.image)

> `Debug`模式下，编译器默认没有优化\
> `Release`模式下，编译器优化为`Fastest,Smalest[-Os]`

**我们将`Debug`模式下编译器优化也调整`Fastest,Smalest[-Os]`**

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fa8bb52e20e40918f076aa24f67d679~tplv-k3u1fbpfcp-watermark.image)

然后运行项目，切换到汇编窗口

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de2b9b755520469c8ab45ba5c9cecde0~tplv-k3u1fbpfcp-watermark.image)

> 我们发现，没有`sum`方法调用的流程了，编译器直接将`sum`方法的运算结果存放在了`w8`寄存器，这就是编译器优化的结果

# alloc的主线流程
### 1、准备工作
在源码工程中创建`Person`类，在`main`方法中调用`[Person alloc]`方法

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/927331ca4782460c89ef1cbf376a897e~tplv-k3u1fbpfcp-watermark.image)

### 2、断点执行
经过`alloc`，`objc_rootAlloc`,最终进入`callAlloc`方法

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2f5b9998fc149518487bbe91a342e62~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4439b8aaefa40c393be76ee31d06353~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c8807f120ed4a88863ab34a491db502~tplv-k3u1fbpfcp-watermark.image)

### 3、继续执行
调用`_objc_rootAllocWithZone`
> 这里有两个宏\
>**slowpath**:`#define slowpath(x) (__builtin_expect(bool(x), 0))`\
>**fastpath**:`#define fastpath(x) (__builtin_expect(bool(x), 1))`\
>作用为：**允许程序员将最有可能执行的分支告诉编译器**\
>`__builtin_expect((x),1)`表示 x 的值为真的可能性更大\
>`__builtin_expect((x),0)`表示 x 的值为假的可能性更大

根据宏定义的含义，大概率会执行`objc_rootAllocWithZone`方法，结果也确实如此：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7aef80cc472c41988d2af273eb6fff70~tplv-k3u1fbpfcp-watermark.image)
随后进入`_class_createInstanceFromZone`方法

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe31d0afb7f84df68f5982e22a22ca4e~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbbe298c6945468e8cb052bd61aded5a~tplv-k3u1fbpfcp-watermark.image)
### 4、`cls->instanceSize`先计算出需要的内存空间大小`(16字节)`

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/680fb873651a4ac0a04faf3d990bbf25~tplv-k3u1fbpfcp-watermark.image)
### 5、`calloc`申请内存空间

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b67a9bb0ec67473a8c2f712ce51df1ba~tplv-k3u1fbpfcp-watermark.image)
> 我们初始化的`obj`,系统默认为我们分配了一块脏内存空间，**内存的数据只会覆盖不会清除**
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c122aeaa668048dfb2a7c384a2254894~tplv-k3u1fbpfcp-watermark.image)
> 执行过`calloc`方法之后，系统为我们申请了新的内存空间，根据打印信息我们发现，当前的`obj`还没有和我们的`Person`关联上，我们继续执行
### 6、`initInstanceIsa`将`obj`与`Person`绑定

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9af2e4a2c8394a59b39055eed7f5cd44~tplv-k3u1fbpfcp-watermark.image)
> 执行过`obj->initInstanceIsa(cls, hasCxxDtor)`之后，`obj`与`cls`存在了绑定关系

**至此，`alloc`底层调用逻辑已经完成，绘制流程图如下：**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c54737766214ba6a0f996e1935bbded~tplv-k3u1fbpfcp-watermark.image)

# 字节对齐及其原理
上文说道`cls->instanceSize`会计算出需要的内存大小，那么如何计算所需内存大小呢？我们进入此方法：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/991745e5fcf94b3aad01e71e0d430b00~tplv-k3u1fbpfcp-watermark.image)
断点执行`[Person alloc]`时，我们可以看到`extraBytes`值为`0`，那么所需内存大小`size`
就由`alignedInstanceSize`决定：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff784400d3ba415a90182f6c1d7d455c~tplv-k3u1fbpfcp-watermark.image)

这里的`unalignedInstanceSize`是`8`字节，来源于`Person`的父类`NSObject`中的`Class isa`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81c37d3439ce4727907fb3da8682852a~tplv-k3u1fbpfcp-watermark.image)

那么`8`字节是如果得出最终的`16`字节大小呢？我们看一下`word_align`的实现：
```
static inline uint32_t word_align(uint32_t x) {
    return (x + WORD_MASK) & ~WORD_MASK;
}
```
> `#define WORD_MASK 7UL`可知`WORD_MASK`为`7`
那么计算变为
```
(8 + 7) & ~7 
即  
15 & ~7 
```
计算过程如下：
```
0000 1111 (15的二进制 )
0000 0111 (7的二进制)
1111 1000 (~7的二进制)非7

15 & ~7 与运算如下
0000 1111
1111 1000
结果
0000 1000  转换为二进制是8
```
> 类似`(8 + 7) >> 3 << 3`

> 8字节对齐，取8的整数，`alignedInstanceSize`结果为`8`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cd57c15034c43b7baf97d4009da0ee5~tplv-k3u1fbpfcp-watermark.image)

>最终计算结果 `if (size < 16) size = 16`可知，`size`小于`16`时，结果取`16`

# 对象的内存空间

那么对象占用内存空间的大小，有什么决定呢？
> 对象占用内存空间的大小，由其`成员变量`决定
**验证过程如下：**
### 1、删除项目中所有断点

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ec99f47de804297849aa5907d526fb1~tplv-k3u1fbpfcp-watermark.image)

> 打印可知对象占用了`8`个字节，(内存中数据以16字节对齐)
### 2、添加一个属性

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3a3ec7e8bb54eb58f96d7c7298c76e0~tplv-k3u1fbpfcp-watermark.image)
### 3、继续添加属性

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd858481ccca444b97e2089bea1274cd~tplv-k3u1fbpfcp-watermark.image)

> 结论：**成员变量越多，占用内存空间越大**

那么`job1`去哪了呢？

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee5472a78f1a474f9ab8250218f8fb62~tplv-k3u1fbpfcp-watermark.image)

> `x/4gx p`以`4`个格式化的排版打印对象`p`
