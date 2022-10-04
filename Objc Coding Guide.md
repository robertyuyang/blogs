## 1. 基本原则

#### 1.编写可读的代码

工程师们往往容易忽略一段代码的后续维护和修改的成本，但无数事实告诉我们，后面长期的阅读和修改代码的时间总是比第一次开发这段代码的时间要长的。任何代码编写者都要时时考虑自己正在写的这段代码，未来可能会被其他人修改，或者在几个月之后被自己修改，而好的可读性在这段代码长久的生命周期中，能为开发者省下来巨大的时间成本，并很大程度上降低代码出错可能。

#### 2.尽量保持Apple风格

不同的语言和框架会有自己的编写风格和惯用法，比如C++中匈牙利命名法在很多语言中都不会使用，而在Objective-C的语言环境中，在不同惯用法的选择中，我们应该更多和Apple风格保持一致。

## 2.命名

#### 2.1 通用命名原则

##### 2.1.1 命名要尽量携带信息

在命名中尽量携带代码读者所需要知道的信息，即使这会增加命名的长度，


```
insertNovel:atIndex: // Good
insert:at: // Avoid
BOOL shouldUpdate; // Good
BOOL upd; //Avoid
NSString* errorMessage; // Good
NSString* err; //Avoid;
```


##### 2.1.2 谨慎使用缩写

很多你认为人尽皆知的缩写，可能在很多人的眼里是困惑或者有歧义的。
> 比如mod，可能有些人理解是module/model,有些人理解是mode，有些人完全不知道是这个缩写代表什么。

很多情况下不需要为了避免一个变量或类名过长而使用可能增加阅读成本的缩写。这里有一份苹果官方可接受缩写的列表： [Acceptable Abbreviations and Acronyms](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/APIAbbreviations.html#//apple_ref/doc/uid/20001285-BCIHCGAE)


##### 2.1.3 不要自引用

比如```NSString```，不应该叫```NSStringObject```或者```NSStringClass```

#### 2.2 变量

变量通用命名规范是小写开头，后面每个单词首字母大写。

建议在变量命名中带入它的类型信息和量词信息。

增加类型信息会让人更快知道如何使用这些变量，比如：


```
urlString //让人更明确这是一个NSString,而不是一个NSURL
titleLabel, coverImageView //从名字就可以让人知道这些控件应该如何填充
```

当然，很多常用的词语很容易从词本身看出其类型，比如title, name, length，但也有很多时候，用到的变量会有类型上的歧义，尤其涉及到类型变换，
比如:

```
NSURL* url = [NSURl urlWithString:urlString]; 
NSDictionary* jsonDict = [NSJSONSerialization JSONObjectWithData:jsonData options:0 error:&jsonError];
```


在使用一些传入时间信息的方法，入参往往让使用者不知道应该传秒还是毫秒，如果加入量词信息，使用起来会更佳直观：

```
timeOutSeconds // 尤其作为方法的参数时，让使用者知道用什么单位去传入参数
```


##### 2.2.1 全局变量

全局变量可以增加"_"或"g"前缀，来表明自己的作用范围，形如：

```
const int gBookCount;
```

##### 2.2.2 局部变量

应当重视局部变量在代码段中的代码解释意义，比如下面这段代码：
```
if (self.versionCode < onlineVersionCode 
    && self.userId.length > 0 
    && self.buildNumber != 3000) {
    [self update];
}
```

这段判断是否要进行升级的代码，让人很难明白具体判断的逻辑到底是什么。如果增加几个局部变量：

```
BOOL firstLaunch = self.userId.length == 0;
BOOL isTestingVersion = self.buildNumber == 3000;

if(self.versionCode < onlineVersionCode && !firstLaunch && !isTestingVersion) {
    [self update];
}
```
就能让人阅读时更加清晰，起到了一个自注释的作用。

不要为了省代码行数而不去使用局部变量，尤其在代码段很长的情况下，局部变量能更好的承接整个代码段的逻辑链条；另外，在长代码段重构的时候，局部变量也是如何重构的一个重要指示物。

##### 2.2.3 成员变量

应当尽量使用属性的setter和getter去访问成员变量，为了更好标识这一点，成员变量的名字应当以"_"开头，如```_itemCount```,并且显式的用@private、@protected修饰。

Objective-C的属性实际上是一个自动生成的setter、getter方法, 因此属性相关的内容会在“方法”小节讨论。


#### 2.3 常量

##### 2.3.1 全局静态常量

全局静态常量应该用每个单词首字母大写的方式命名，前缀k表明这是一个常量，为保证全局符号不冲突，也可以酌情加上前缀。

```
static const NSTimeInterval kViewControllerFadeOutAnimationDuration = 0.4; //Good
static const NSTimeInterval fadeOutTime = 0.4; //Avoid
```
全局静态变量可以在头文件里暴露给其他文件：

```
extern NSString *const kCacheControllerDidClearCacheNotification;
```
从编译上，这个extern的表达式不一定非要写在同一个类文件的h文件里，这个表达式只是告诉引入了这句话的编译单元，这个符号是一个外部符号，编译时可以认为这是一个合法的符号，到链接时会有这个符号的具体实现。但从设计上一般都会和这个变量的具体实现写到同一个类的h和m文件里。

##### 2.3.2 枚举常量

枚举常量的命名应该参考类的命名，使用前缀避免全局命名冲突，

枚举类型内的标识符，应该以枚举类型名开头，后面加上每个标识符自己的含义，这样能方便枚举使用者查找。

下面是```UIControl.h```里的例子：

```
typedef NS_ENUM(NSInteger, UIControlContentVerticalAlignment) {
    UIControlContentVerticalAlignmentCenter  = 0,
    UIControlContentVerticalAlignmentTop     = 1,
    UIControlContentVerticalAlignmentBottom  = 2,
    UIControlContentVerticalAlignmentFill    = 3,
};
```

#### 2.4 类

类名应该用每个单词首字母大写的形式，

##### 2.4.1 前缀

因为Objective-C对命名空间的缺失，我们需要通过前缀的方式来保持类名的唯一性。前缀往往使用两到三个大写字母表示，例子：

Prefix  | Cocoa Framework
------------- | -------------
NS  | Foundation
NS  | Application Kit
AB  | 	Address Book
IB  | Interface Builder


##### 2.4.2 子类

子类的名字应该是在父类的类前缀和父类名之间，比如父类叫```WTAudioPlayer```，子类可以叫做```WTStreamAudioPlayer```

#### 2.5 方法


##### 2.5.1 通用命名规范

方法名应该以小写开头，后面每个单词首字母大写的形式。有部分特殊情况可以允许大写字母开头命名一个方法，比如方法开头是一个固定的专有名词缩写：

```
+ (NSURL *)URLWithString:(NSString *)URLString;

- (NSString *)PDFfromFile(NSString *)filePath;

```

不要用前缀，因为类已经为类方法提供了一个命名空间。


##### 2.5.2 动词选择

方法命名应该尽量接近于一个自然语言的句子，Objective-C风格的方法名可以比较长，只要能够详细和通顺的表达这个方法的作用。

如果这个方法是用来表示一个动作，方法名应该以一个动词开头，另外不要在动词前添加副词：

```
- (void)addObject:(id)object;
```
尽量不选择do或者does作为一个方法名中的动词，因为do和does携带的信息太少。

如果方法是直接返回实例内部的不需要特殊加工的数值，那么可以省略掉"get"：

```
- (NSSize)cellSize; // GOOD
- (NSSize)getCellSize; // AVOID

```

注意必要时运用can,should,will,did等词放在动词前面来更准确的表明方法，避免歧义：

```
- (BOOL)canHide;
```

方法名作为对外接口的一部分，应当尽量隐藏方法内部的实现细节，但也要尽量在用词上提醒方法使用者这个方法的成本和副作用，比如异步方法，有网络请求开销，有文件IO的方法。

另外，同一个项目里，相同的动作应该用相同的动词，比如从硬盘读取数据都叫load或都叫read，统一动词能减少代码阅读的成本。

##### 2.5.3 参数

每个参数前都要有关键字。

```
- (void)sendAction:(SEL)aSelector toObject:(id)anObject forAllCells:(BOOL)flag; // GOOD
- (void)sendAction:(SEL)aSelector :(id)anObject :(BOOL)flag; // AVOID

```

参数前的关键字要去准确的描述后面的参数，

```
- (id)viewWithTag:(NSInteger)aTag; //GOOD
- (id)taggedView:(int)aTag; // AVOID 
```
不要用一两个字母表明参数，不要为了省几个字符而使用含义不明的缩写，参数名字里不要使用"pointer"或"ptr"，让参数前的关键字去决定它是否是一个指针。
谨慎选择参数前面的介词或连词，尽量不要用"and"去连接两个属性，

```
- (void)addTarget:(id)target action:(SEL)action; // GOOD
```

如果这个方法会进行两个独立的动作，则可以考虑用"and"连接两个动作：

```
- (BOOL)openFile:(NSString *)fullPath withApplication:(NSString *)appName andDeactivate:(BOOL)flag;
```
只有必须用介词和连词（from, to, with, for)来表明参数与动作的关系，并且避免歧义时，才去使用：

```
- (CGPoint)convertPoint:(CGPoint)point fromView:(UIView *)view;           // GOOD; conjunction clarifies parameter
- (void)replaceCharactersInRange:(NSRange)aRange
            withAttributedString:(NSAttributedString *)attributedString;  // GOOD.
```

如果编写了比原有一个方法更具体的方法时，新加的参数应该放在原有参数的后面：

```
- (id)initWithFrame:(CGRect)frameRect;
- (id)initWithFrame:(NSRect)frameRect mode:(int)aMode cellClass:(Class)factoryId numberOfRows:(int)rowsHigh numberOfColumns:(int)colsWide;

```
参数的顺序应该遵循更重要的参数在前，次重要的在后，输入参数在前，输出参数在后的原则：

```
//更重要的data在前，次重要的opt在后，输出参数error在最后
+ (nullable id)JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError **)error; 
```

```
//completion作为输出参数在最后
- (void)dismissViewControllerAnimated: (BOOL)flag completion: (void (^ __nullable)(void))completion NS_AVAILABLE_IOS(5_0);
```

##### 2.5.4 Accessor

accessor是用来返回和设置对象内部数据的方法（一般不经过复杂加工）。
如果要返回的数据是个名词，则直接用数据的名词作为方法名，不要加"get"：

```
- (id)delegate;     // GOOD.
- (id)getDelegate;  // AVOID.
```
如果是形容词则可以是用"is":

```
@property(nonatomic, getter=isGlorious) BOOL glorious;

- (BOOL)isEditable;
- (void)setEditable:(BOOL)flag;

```

##### 2.5.5 Delegate

delegate往往用来实现一种观察者模式，实现delegate协议的类是一个观察者，等待被观察者在数据发生变化时发送消息。

delegate方法名一般以被观察者作为第一个参数，第一个关键字去掉类型前缀：

```
- (BOOL)tableView:(NSTableView *)tableView shouldSelectRow:(int)row;
- (BOOL)application:(NSApplication *)sender openFile:(NSString *)filename;
```
因为观察者往往需要或者数据变化后的观察者的一些信息，另外传入被观察者，也允许一个对象可以通过同一个delegate协议观察多个同类型对象。

如果该方法只有一个被观察对象一个参数的话，则应该把动作放在前面，最后是被观察对象：

```
- (BOOL)applicationOpenUntitledFile:(NSApplication *)sender;
```

善用"did","will",来更准确的告知观察者，被观察对象动作的不同阶段：

```
- (void)browserDidScroll:(NSBrowser *)sender;
```


#### 2.6 Category

category是一个非常好用的，可以顺畅融入原有类的一个语言特性，也因为它融入得过于顺畅，因此也是需要谨慎使用的一个特性。
强烈推荐在分类方法名前加入小写加下划线的前缀，如：

```
@interface NSSortDescriptor (XYZAdditions)
+ (id)xyz_sortDescriptorWithKey:(NSString *)key ascending:(BOOL)ascending;
@end
```

这个写法也是被苹果推荐的：[Avoid Category Method Name Clashes](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html#//apple_ref/doc/uid/TP40011210-CH6-SW4)

这个前缀的作用有两个：一个是能够避免全局性的命名冲突，如果对同一个类的两个category都编写了一个同名方法，那么先加载的category会被被加载的category的方法所覆盖，而这个时候出现的问题将是非常难以察觉和调试的。第二个好处是将原有方法和扩展方法分开，方便代码阅读者阅读，代码维护者可以轻易的分辩哪些方法是原有的sdk方法，是通用的且经过了验证和测试的，哪些方法是开发人员自己扩展的方法，是可能存在特殊逻辑或潜在问题的。


#### 2.7 NSNotification

Notification的名字应该命名成：

```
[Name of associated class] + [Did | Will] + [UniquePartOfName] + Notification
```

## 3.格式

#### 3.1 基本排版
##### 3.1.1 缩进
使用空格作为缩进，不要使用tab，可以在Xcode中设置成tab也由空格组成。

##### 3.1.2 行宽
单行代码不要超过100列。

##### 3.1.3 空格和空行
二元运算符之间应该有空格

```
// GOOD:
x = 0;
v = w * x + y / z;
v = -y * (x + z);
```

在长的代码段中，应该用空行来分隔相对逻辑更紧密的代码段。

##### 3.1.4 方法行数

这里不强行规定最长的方法行数，但始终应该尽量避免在一个方法中有过多的代码，应该尽量用重构的方式去缩短方法内的代码行数。

#### 3.2 条件语句

##### 3.2.1 括号



if,else后面应该永远接一个括号，即使只有一条语句。

在 2014年2月 苹果的 SSL/TLS 实现里面发现了知名的 goto fail 错误。

代码在这里：

```
static OSStatus
SSLVerifySignedServerKeyExchange(SSLContext *ctx, bool isRsa, SSLBuffer signedParams,
                                 uint8_t *signature, UInt16 signatureLen)
{
  OSStatus        err;
  ...

  if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
    goto fail;
  if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
    goto fail;
    goto fail;
  if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
    goto fail;
  ...

fail:
  SSLFreeBuffer(&signedHashes);
  SSLFreeBuffer(&hashCtx);
  return err;
}
```

这里有没有括号包围的2行连续的 goto fail，而第二个goto fail无论如何都会被执行。

条件判断的后续表达式应该尽量使用Egyptian Brackets，即代码段括号的开始位于一行的末尾，而不是另外起一行的风格:

```
//GOOD:
if (user.isHappy) {
    //Do something
}
else {
    //Do something else
}

//AVOID
if (user.isHappy)
{
  //Do something
} else {
  //Do something else
}
```

##### 3.2.2 减少逻辑判断的嵌套层数

嵌套次数过多会严重影响可读性，并增加出错可能。

一方面应该再业务层面检查是否能够减少逻辑判断次数，另一方面要多使用break/return来减少进入嵌套的可能。

```
// GOOD:
- (void)someMethod {
  if (![someOther boolValue]) {
      return;
  }

  //Do something important
} 

//AVOID:
- (void)someMethod {
  if ([someOther boolValue]) {
    //Do something important
  }
}
```

##### 3.2.3 三元运算符
首先三元运算符应该是一个谨慎使用的表达式，很多时候它并不比if的可读性高。应该只在特别间断的逻辑判断中使用。

```
result = a > b ? x = c > d ? c : d : y; //AVOID
```

当三元运算符的第二个参数（if 分支）返回和条件语句中已经检查的对象一样的对象的时候，下面的表达方式更灵巧：

```
result = object ? : [self createObject]; //GOOD
result = object ? object : [self createObject]; //AVOID
```

##### 3.2.4 BOOL判断

不要直接拿一个BOOL值跟YES进行比较。

不光是因为这样增加了阅读成本，还因为BOOL类型实际上是一个8bit的char，它的取值范围其实是多于YES(1)和NO(0)的，当你直接拿一个BOOL值去跟YES(1)相比，得到的结果可能并非你的预期。

```
BOOL great = [foo isGreat];
if (great == YES) {  // AVOID.
  // ...be great!
}

BOOL great = [foo isGreat];
if (great) {         // GOOD.
  // ...be great!
}
```

不要强行将一个其他类型转换成BOOL，比如将一个容器的size转成BOOL值，这时候这个BOOL值的取值完全取决于转换之前的数字的最后8bit。

也不要对BOOL值使用位运算。

##### 3.2.5 case语句

对数值或枚举类型的变量进行多行的判断时，应该用case语句去代替多行的else if。编译器在编译case语句时，把不同的case情况的代码段地址编译到一个数组中，然后直接用switch中输入的数字或枚举作为索引跳转代码段，效率远高于多行的else if。

在使用fall-through的时候，如果在当前case编写了语句再进行fall-through，应该增加注释，避免遗漏break:

```
// GOOD:

switch (i) {
  case 1:
    ...
    break;
  case 2:
    j++;
    // Falls through.
  case 3: {
    int k;
    ...
    break;
  }
  case 4:
  case 5:
  case 6: break;
}
```
 
有限的枚举值的case语句，应该去掉最后的default：
 
```
switch (menuType) {
    case ZOCEnumNone:
        // ...
        break;
    case ZOCEnumValue1:
        // ...
        break;
    case ZOCEnumValue2:
        // ...
        break;
}
```
这样，如果新增加了一个枚举值，而case语句中没有处理，编译器会报出一个warning：xx enumeration values not handled in switch:


#### 3.3 方法

##### 3.3.1 方法声明

方法名与方法类型 (-/+ 符号)之间应该以空格间隔。参数和参数前关键字之间没有空格，参数之间用一个空格。

```
- (void)doSomethingWithString:(NSString *)theString {
  ...
}
```
类型名和*之间的空格可按照自己的习惯，但最好跟整体项目保持统一。

如果一个方法的签名太长，可以拆分每个参数到单独一行，每行冒号对齐：

```
// GOOD:
- (void)doSomethingWithFoo:(GTMFoo *)theFoo
                      rect:(NSRect)theRect
                  interval:(float)theInterval {
  ...
}
```

方法段之间也应该以空格间隔（以符合 Apple 风格）。

##### 3.3.2 方法调用

方法调用时如果参数太多，可以拆分到单独一行，每行冒号对齐：

```
// GOOD:

[myObject doFooWith:arg1
               name:arg2
              error:arg3];
```

下面的调用方式都应该避免：

```
// AVOID:

[myObject doFooWith:arg1 name:arg2  // some lines with >1 arg
              error:arg3];

[myObject doFooWith:arg1
               name:arg2 error:arg3];

[myObject doFooWith:arg1
          name:arg2  // aligning keywords instead of colons
          error:arg3];
```


## 4.代码组织

#### 4.1 头文件
##### 4.1.1 包含顺序
头文件包含的顺序，应该遵循越通用的的头文件在上面，越跟当前业务相关的头文件在下面。

一般一个标准的包含顺序应该是：related header, 操作系统头文件，语言库头文件，第三方库头文件，本项目通用/底层模块头文件，当前模块的外部依赖头文件

```
// GOOD:

#import "ProjectX/BazViewController.h"

#import <Foundation/Foundation.h>

#include <unistd.h>
#include <vector>

#include "base/basictypes.h"
#include "base/integral_types.h"
#include "util/math/mathutil.h"

#import "ProjectX/BazModel.h"
#import "Shared/Util/Foo.h"
```

##### 4.1.2 尖括号

导入系统framework时要使用尖括号，方便编译器从系统配置头文件路径里查找。

#### 4.2 编译器/预编译器指令
##### 4.2.1 Pragma Mark
用Pragma Mark给代码文件分段是非常好的一种代码组织方式。一般可以分为以下几类：

- 不同功能组的方法
- protocols 的实现
- 对父类方法的重写

```
- (void)dealloc { /* ... */ }
- (instancetype)init { /* ... */ }

#pragma mark - View Lifecycle （View 的生命周期）

- (void)viewDidLoad { /* ... */ }
- (void)viewWillAppear:(BOOL)animated { /* ... */ }
- (void)didReceiveMemoryWarning { /* ... */ }

#pragma mark - Custom Accessors （自定义访问器）

- (void)setCustomProperty:(id)value { /* ... */ }
- (id)customProperty { /* ... */ }

#pragma mark - IBActions  

- (IBAction)submitData:(id)sender { /* ... */ }

#pragma mark - Public 

- (void)publicMethod { /* ... */ }

#pragma mark - Private

- (void)zoc_privateMethod { /* ... */ }

#pragma mark - UITableViewDataSource

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath { /* ... */ }

#pragma mark - ZOCSuperclass

// ... 重载来自 ZOCSuperclass 的方法

#pragma mark - NSObject

- (NSString *)description { /* ... */ }
```

##### 4.2.2 宏

谨慎的使用宏，尽量用const常量、枚举以及全局函数来替代宏。

因为宏会让你看到的代码和编译器看到的代码不一致。使得一旦出现问题，编译器很容易给出让你看不明白的提示，非常难以调试。宏应该只有在没有其他替换方案的时候才去使用。

宏应该使用所有字母大写，单词之间用下划线的SHOUTY_SNAKE_CASE形式，如果是函数形式的宏，可以用类似全局函数的命名风格，并且要起独特的名字来避免命名冲突，有必要的话可以用#undefining缩小这个宏的影响范围。

```
// GOOD:

#define GTM_EXPERIMENTAL_BUILD ...      // GOOD

// Assert unless X > Y
#define GTM_ASSERT_GT(X, Y) ...         // GOOD, macro style.

// Assert unless X > Y
#define GTMAssertGreaterThan(X, Y) ...  // GOOD, function style.
```

不要在宏里出现不完整的语法结构。比如：

```
#define unless(X) if(!(X))              // AVOID
```

不要用宏来生成类、属性、变量和方法。不要在宏里面声明变量给外面使用，也不要在宏里使用宏外面声明的变量。

```
// AVOID:

#define ARRAY_ADDER(CLASS) \
  -(void)add ## CLASS ## :(CLASS *)obj toArray:(NSMutableArray *)array

ARRAY_ADDER(NSString) {
  if (array.count > 5) {              // AVOID -- where is 'array' defined?
    ...
  }
}
```
##### 4.2.3 warning和error

应该善用warning和error在代码段里增加编译期的代码提示。

```
- (NSInteger)divide:(NSInteger)dividend by:(NSInteger)divisor
{
    #warning 需要增加安全性检查
    return (dividend / divisor);
}
```

#### 4.3 注释
好的代码应该是自注释的，但必要时也可以通过注释提高代码可读性，或记录容易遗忘的细节。

##### 4.3.1 格式

对方法声明的注释方式：

```
/**
 *  Designated initializer.
 *
 *  @param  store  The store for CRUD operations.
 *  @param  searchService The search service used to query the store.
 *
 *  @return A ZOCCRUDOperationsStore object.
 */
- (instancetype)initWithOperationsStore:(id<ZOCGenericStoreProtocol>)store
                          searchService:(id<ZOCGenericSearchServiceProtocol>)searchService;
                          
```

单行注释应该至少距离被注释代码后两个空格，如果有多行注释的话，可以考虑把他们对齐：

```
// GOOD:

[self doSomethingWithALongName];  // Two spaces before the comment.
[self doSomethingShort];          // More spacing to align the comment.
```

##### 4.3.2 注释内容

注意注释不要注释显而易见的内容，比如：

```
[self.itemList addObject: responseObject];  //insert responseObject into itemList
```
而应该更多把一些难以从代码层面读明白的逻辑，或者代码段的目的写到注释里：

```
//
//必须延迟5秒再隐藏，否则会有XXXX的问题
//
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t) (NSEC_PER_SEC * 5)), dispatch_get_main_queue(), ^{

    self.hidden = YES;
});
```

#### 4.4 代码块

下面是代码块的写法：

```
NSURL *url = ({
    NSString *urlString = [NSString stringWithFormat:@"%@/%@", baseURLString, endpoint];
    [NSURL URLWithString:urlString];
});
```

这个写法其实类似做了一个局部匿名函数，而函数是能明显减少变量生命周求和影响范围的，这种代码块不但增加的代码的可读性，让读者明确知道这一段代码的目的，另外也减小代码块里面的变量对其他代码的影响。

## 5.NSObject

#### 5.1 init和dealloc

##### 5.1.1 不要在init和dealloc里调用实例方法

不同于C++语言的虚函数，Objective-C里的任何实例方法都是可以被子类改写的，因此在一个NSObject类的init方法里调用实例方法，有可能去调用子类的实现，而这个时候子类的init是在父类的init调用之后的，因此这个时候调用子类的实例方法会出现意想不到的结果。同样的问题也存在于dealloc中。

一个容易忽略的方法调用是属性的setter和getter，所以在init和dealloc中要直接调用属性变量，而不要通过setter和getter去访问。

```
// GOOD:

- (instancetype)init {
  self = [super init];
  if (self) {
    _bar = 23;  // GOOD.
  }
  return self;
}
```

##### 5.1.2 不要使用+new

用alloc init来代替NSObject的new方法。

##### 5.1.3 designated initializer

用NS_DESIGNATED_INITIALIZER宏来明确明确的指出当前类的designated initializer。

designated initializer能让你定义的类安全的初始化所有的成员，也让继承这个类的开发者就能很容易的知道如何去重写父类的initializer，而且在调试的时候也能很容易的分辨这个类初始化时的数据流。

##### 5.1.4 初始化为0

不要在initializer里对成员变量赋值0或者nil。因为编译器会将这些变量[初始化为0](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/ObjectAllocation/ObjectAllocation.html)，手动初始化只会降低运行效率。

#### 5.2 重写方法

重写方法要放在 NSObject的@implementation的最顶端。

## 6.其他Objective-C Features

#### 6.1 Nullability
接口和属性可以用nullability注释来描述这些接口和属性的使用方式，如果觉得麻烦可以用 NS_ASSUME_NONNULL_BEGIN 和 NS_ASSUME_NONNULL_END 可以节约一部分的工作量。
这种nullability可以提高代码的健壮性，也让使用者更容易知道这些接口的预期行为。

```
// GOOD:

/** A class representing an owned book. */
@interface GTMBook : NSObject

/** The title of the book. */
@property(readonly, copy, nonnull) NSString *title;

/** The author of the book, if one exists. */
@property(readonly, copy, nullable) NSString *author;

/** The owner of the book. Setting nil resets to the default owner. */
@property(copy, null_resettable) NSString *owner;

/** Initializes a book with a title and an optional author. */
- (nonnull instancetype)initWithTitle:(nonnull NSString *)title
                               author:(nullable NSString *)author
    NS_DESIGNATED_INITIALIZER;

/** Returns nil because a book is expected to have a title. */
- (nullable instancetype)init;

@end

/** Loads books from the file specified by the given path. */
NSArray<GTMBook *> *_Nullable GTMLoadBooksFromFile(NSString *_Nonnull path);

```


#### 6.2 容器类型显式标注泛型类型
Xcode7以上就支持在容器类型后面增加<>来标识容器内部承载的对象的类型。

所有的NSArray, NSDictionary, 和 NSSet等类型，都应该加上这个标注，来增加代码可读性和类型安全性：

```
// GOOD:

@property(nonatomic, copy) NSArray<Location *> *locations;
@property(nonatomic, copy, readonly) NSSet<NSString *> *identifiers;

NSMutableArray<MyLocation *> *mutableLocations = [otherObject.locations mutableCopy];
```

#### 6.3 self的循环引用

应该用下面这种方式来避免block引起的循环引用：

```
__weak __typeof(self)weakSelf = self;
__strong __typeof(weakSelf)strongSelf = weakSelf;
```

第二行的__strong__可以避免在调用到block内部代码运行过程中self被意外释放，__strong__可以保证在当前这个生命周期内，self一直至少有一个引用计数。


#### 6.4 UIKit中的delegate

iOS的framework实际分两种，我们自己编写呃framework是静态库，而系统暴露出来给开发者使用的framework实际是动态库，开发者在Xcode里使用的UIkit实际只有声明（h文件）和导出符号表，所有UIKit的实现都是在操作系统中以动态库形式存在的。而UIKit中UITableView，UIScrollView，UICollectionView等view的delegate在iOS8前后的实现是有本质的差别的。

```
// UITableView iOS 8 and below
@property(nonatomic, assign) id<UITableViewDataSource> dataSource
@property(nonatomic, assign) id<UITableViewDelegate> delegate

// UICollectionView iOS 8 and below
@property(nonatomic, assign) id<UICollectionViewDelegate> delegate
@property(nonatomic, assign) id<UICollectionViewDataSource> dataSource
```
而这些delegate属性在iOS9中从assign变成了weak：

```
// UITableView iOS 9
@property(nonatomic, weak, nullable) id<UITableViewDataSource> dataSource
@property(nonatomic, weak, nullable) id<UITableViewDelegate> delegate

// UICollectionView iOS 9
@property(nonatomic, weak, nullable) id<UICollectionViewDelegate> delegate
@property(nonatomic, weak, nullable) id<UICollectionViewDataSource> dataSource

```
可能出现问题的场景就是，当一个View的delegate（通常是一个ViewController)先于View销毁了，而之后View又想给这个delegate发送消息，而在iOS8或之前的系统里，这个delegate属性不是一个weak指针，不会随着delegate的销毁而自动置为nil，这时候的delegate就是一个不可访问的内存，会造成程序的crash。

因此，任何支持iOS8及往前版本iOS的app程序，UITableView，UIScrollView或者UICollectionView的delegate在自己的dealloc时应当把自己赋值给其他view的delegate属性设置为nil，否则会出现内存错误。

#### 6.5 异常

不要跑出任何的Objective-C异常，但是要准备好catch第三方库的异常。


## 参考文献

- [Google Objective-C Style Guide](https://github.com/google/styleguide/blob/gh-pages/objcguide.md)
- [Zen and the Art of the Objective-C Craftsmanship](https://github.com/objc-zen/objc-zen-book)
- [ Coding Guidelines for Cocoa](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html#//apple_ref/doc/uid/10000146-SW1)
- [UIKit changes in iOS 9](https://www.jessesquires.com/blog/UIKit-changes-in-iOS-9/)
- [Object Allocation](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/ObjectAllocation/ObjectAllocation.html)








