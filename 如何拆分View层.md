## 到底View层指哪一层

如果谈狭义的View层，可能范围会小一些，往往只包含布局文件（layout xml,storyboard,xib），然而我们更多打交道的往往是接管了View的响应操作的一些容器。客户端开发中，官方提供的框架里都有一个用来管理View的容器，iOS的叫ViewController，Android的叫Activity，Windows也有一些CFrameWindowImpl、CWindow这样的类，用message map的方式封装了View的各种响应。

在MVC的架构里，ViewController和Activity的定位是Controller，但实际上却跟View的职责密不可分，使得他们的负责内容变得非常暧昧。而我们谈论的View层，是一个广义的View层，更接近MVP/MVVM里的View层定位，它包含了布局的文件、自定义控件、自定义的View、以及ViewController/Activity——特别是ViewController/Activity，以及跟他们地位类似的一些类，是我们需要重构和拆分的重点。

## View层的职责是什么？

我们先不说View层应该做什么，先说View层可以做什么，或者说，包含了传统/非传统的架构里，合理/不合理的架构里，View都在做什么。

- 基本布局 （狭义的View）：静态布局、代码布局，控件被动运动能力及运动时的动画；
- 交互逻辑 ：修改View状态，修改数据状态（向下传递）；
- 展现逻辑 ：数据持有，数据加工，数据展现（向上传递）；

简单解释几句：
- 修改View状态是说，用户在View上的点击有一部分会反映到数据层（Model）的修改，一部分则是View的内部状态，比如导航的选中状态、控件的disable/hidden，位置大小的变化等。
- 数据展现包含了初始数据状态的展现，以及View层对数据层（或是逻辑层）的后续变化进行观察，并在观察后对最新数据的展现。
 
View层在App中的地位很特殊，它是一个必须项，如果我们写的App足够简单，甚至可以只有View、没有Model没有Controller没有Presenter。正是因为View的这种地位，使得它特别容易膨胀，因为：一来反正我们总是要有个View的，二来简单的去看，似乎一个App的所有行为都是在View里进行的，或跟View有关，那我们干嘛不把代码都放在里面呢？


## 怎么去拆分和重构一个过大的View层


### 拆分数据加工部分 (拆下层)

我觉得首先应该也必须从View层摘出去的就是数据加工的工作。

这些工作最初落进View层往往是因为最初写这些代码的时候并不认为自己在“加工”，比如有一个数码产品需要展现商品名:

```
self.itemNameLabel.text = self.item.name;
```

然后发现需求里还写了如果是笔记本或者手机商品，需要在名字里带上容量信息。代码变成：
```
if (self.item.category.hasCapacityInfo && self.item.capacity != 0) {
    self.itemNameLabel.text = [NSString stringWithFormate: @"%@ (%d)", self.item.name, self.item.capacity];
}
else{
    self.itemNameLabel.text = self.item.name;
}
```

哦，这样不对，后端吐的capacity居然是个数字，代表GB数，这里还需要换算;另外产品经理又说，如果产品下架了，商品名里要标明已经下架,代码可能变成：
```
if (self.item.category.hasCapacityInfo && self.item.capacity != 0) {
    if (self.item.capacity < 1024) {
        self.itemNameLabel.text = [NSString stringWithFormate: @"%@ (%dGB)", self.item.name, self.item.capacity];
    }
    else {
        self.itemNameLabel.text = [NSString stringWithFormate: @"%@ (%dTB)", self.item.name, self.item.capacity / 1024];
    }
    
}
else{
    self.itemNameLabel.text = self.item.name;
}

if(self.item.unavavialbe) {
    self.itemNameLabel.text = [self.itemNameLabel.text stringByAppendingString:@"(已下架)"];
}
```

写着写着，本来做的简单的“逻辑判断”，其实已经变成了“数据加工"，而这部分工作是不应该存在于View层的，尤其这些工作很有可能需要在其他位置复用的情况下。

将这些工作拆到哪去，有很多选择：可以放到一个ItemPresenter或者ItemViewModle里，或者用一个helper类去帮助Model做这些加工处理，还可以把一些更通用的数据处理（比如容量的字符串加工）放到Model里，相对更上层业务联系更紧密的数据处理放到Presenter或ViewModel里。

总之，经过拆分之后，View层的代码应该变成：

```
self.itemNameLabel.text = self.itemViewModel.displayName;
```

拆分这里的要领就在于，这些数据处理是不是完全不需要UI参与，如果是，那就应该拆出去。

### 拆分部分基本布局成为通用自定义View（拆上层）
如果有一些相对通用的UI部分，而且这部分不涉及任何业务逻辑，可以单独拆成自定义View。比如刷新控件（MJRefresher），loading控件等，有些特别通用的控件有第三方的控件，那么有一些没那么通用的就需要我们自己去拆分了，比如一个app多处会使用到的错误页、通用的弹窗、设计了特定动画的搜索框等等；另外即使是第三方控件，在我们工程中需要一些定制，或需要绑定我们自己的素材，这种也完全可以拆成单独的View，但是记得，要把定制的部分和原有的第三方代码明确区分开，方便后面的维护和问题定位。

注意一点是，这里的拆分是不带业务逻辑的，自定义View需要设计通用的响应接口，本身可以做一部分的交互逻辑和表现逻辑，但这个逻辑一定是抽象过的、与具体Model无关的，而更具体业务逻辑则丢给实现响应接口的模块来做。

这种拆分方式提供了更好的复用性和更清晰的控制流和依赖关系，同时也有他的适用范围。就是待封装的这些子View有一定的关联性，而且这个自定义组合View的输入输出的接口数，是要明显少于子View个数的。

比如下面的搜索建议界面，

![image](http://note.youdao.com/yws/res/2245/5EC4DFDBDDF04F67B7459D226CCCBEFA)



我们可以把热门搜索的部分单独封装成自定义View，而这个View只需要一个输入:
```
- (void)resetPopularSearchData:(List<String> data);
```
和一个响应回调:
```
public interface IPopularSearchClickListener {
       void itemClick(Stirng item);
}
```
然后我们发现下面的“历史搜索”跟“热门搜索”的逻辑是非常类似的,

![image](http://note.youdao.com/yws/res/2224/BA211DF477F345C7A09CA78E18383B14)

那我们可以做一步抽象:
```
- (void)resetData:(List<String> data, String title);
```
```
public interface ISearchSuggClickListener {
       void itemClick(Stirng item);
       void deleteItemClick();
}
```
这是一个特别简单的例子，但不难看出适用于这种拆分方式的特征是：表现逻辑不需要太多的输入；交互逻辑不需要太多输出。

结合这个case来说更具体的是：
- 多个子View之间有联动，不需要为每一个子View设置输入（多个Cell只需要一个数组输入）。
- 多数表现逻辑是可以封装在内部不需要跟外部联动的（每个热词使用相同的控件，内部计算cell尺寸，计算排版和分行逻辑）。
- 最终产生需要传导到外部的交互逻辑的类型不会很多（这个case虽然多个控件有响应动作，但响应类型并不多）。

或者说，如果待封装的一组子View，每个都需要不同的输入，需要不同的响应，那这个封装的意义就会变小。

但实际的开发中，对自定义View的封装往往变成:

有一个弹出/浮动界面，看上去很适合封装成自定义View

--> 封装自定义View组合多个控件 

--> 发现很多控件都需要各自的输入数据，也都需要各自的响应动作 

--> 不想写8个set方法和10个响应回调，干脆把一个复杂对象（Model）传到自定义View里 

--> 发现封装的自定义View不光修改数据状态，还要修改主UI的View状态，

--> 干脆把主UI的对象指针传进来，让自定义View可以直接修改主UI的View状态。

最后的结果是，主View的代码行数确实下降了，但整体代码逻辑更加复杂了，阅读成本和调试成本都增加了。

然而很多时候确实封装出纯粹的UI层，那么带着业务逻辑去拆分是否可取和可行呢，下面会讲到：

### 拆分部分基本布局和对应的交互、表现逻辑（内部分块）

上面提到了每个View层的逻辑涉及到交互逻辑、表现逻辑，而这些逻辑都会有他们所服务的UI元素。一个View如果特别庞大，很有可能因为这个View上涉及了太多的UI元素，而这大量的UI元素，有时候并不是铁板一块，也就是说这些UI元素以及其对应的逻辑往往是可以分块的。

iOS可以用ChildViewController或者自定义View的方式，Android可以用Fragment或者自定义控件的方式

这里我们可以分成两种情况：
- 一个大的View里的众多子View是可以根据子业务分组的，每个组之间联系更紧密，我们可以把分组后的View以及其相关的逻辑拆出去。
- 一个View里有一个特别复杂子View，比如一个WebView、一个书架、一个地图，这个子View的表现逻辑和交互逻辑都特别复杂，我们可以选择把这个复杂子View保留在主View中，然后把部分表现逻辑和交互逻辑接口的实现拆到其他类里。

#### 对于第一种情况：

比如我们做一个听书的App，在书城上往往会悬浮（或者冻结在底部）一个miniPlayer，这个miniPlayer负责展示当前正在播放的声音的相关信息（表现逻辑）以及相关控制（交互逻辑），这些内容和书城的UI元素和逻辑基本没太多关联。那我们完全可以把miniPlayer相关的基本布局和逻辑拆分到一个单独的类里。

主View要做的工作就是用合适的Model或者ViewModel去初始化这个类，然后把这个类负责的子View安排到主View的预定位置上。

```
UIMiniPlayerViewController* miniPlayerVC = [UIMiniPlayerViewController alloc] initWithCurrentTrackData:self.currentTrackData];
[self addChildViewController:miniPlayerVC];
[self.view addSubview:self.miniPlayer.view];
CGRect bounds = self.view.bounds;
self.miniPlayer.view.frame = CGRectMake(0, bounds.size.height - playerHeight, bounds.size.width, playerHeight);
    
```





再举个例子，一个视频或者音频播放的界面，除了播放本身相关的表现逻辑和交互逻辑外，往往需要一个播放列表的展示，这个播放列表跟当前播放的音视频不是完全独立，但他们之间的通信内容也比较有限。

![image](http://note.youdao.com/yws/res/2275/34E52AC22C364554923691C9F2033B05)

主播放器在播放完当前的音视频的时候，切换到播放列表的下一个项目，而播放列表的当前播放位置也要相应切换，而播放列表中的点击，也会影响主播放器的播放内容。

这里有两种拆分方式，一种是让主View持有当前播放内容的Model/ViewModel，让播放列表View去持有播放列表的Model/ViewModel,播放列表View实现一个接口:

```
public interface IPlayList {
       void playNext();
       void playPrev();
}
```
让主播放器去通知播放列表切换当前播放内容，然后主播放器再实现一个观察者接口注册到播放列表的观察序列里：
```
public interface IPlayListListener {
       void newVideoPlayed(VideoItem item);
}
```

还有一种是主View和子View在设置好位置后不再互相通信，把当前播放内容和播放列表组合成一个Model/ViewModel，主播放器View和播放列表View都去观察这个组合后的Model，两个View通过Model来进行间接的通信。


其实最常见的例子就是各个app的首页，不同tab下都是不同的业务，分成了不同的View去实现各自的布局和逻辑。这是系统框架已经帮我们约定好的分组方法，更细的分组方式就需要我们自己根据业务场景去挖掘。

#### 对于第二种情况：

这种情况往往是一个负责的View支持很多种操作，比如一个列表，只对应一个Model，这两个东西都是很难再拆分的，而且主View也要用到。这里我们可以选择不让当前主View（一般是一个ViewController或者Activity）去实现这个列表的DataSource/Adapter和Delegate/OnItemClickListener，而是让一个子View去实现，并且这个子View同样持有这个Model。

这样拆分出来的子View是一种View Helper，能很明显的为主View减负，尤其是有的View交互逻辑非常复杂，或者有多个View都需要复杂的表现逻辑和交互逻辑的时候。

虽然这种拆分是依照控件的接口来拆分的，但我们仍然要先注意他们的“可拆分性”，
这种拆分有点类似于C#的partal类或者OBjc中的category的作用，它把本来View的诸多动作按类型拆分成不同的组，每个组之间可以有共同持有的对象，但不能太多。

这里有一篇很好的文章[《Lighter View Controllers》](
https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/)讲了如何把TableView的DataSource剥离出去，从而让ViewController的结构更简洁。

但这种方式的拆分，使用起来要更加小心，因为主View和子View的关系相对之前更加平行，两者持有类似的东西，做更相近的业务场景。很容易会变成仅仅是减少了主View的代码行数，却大大增加了阅读成本和维护成本。

为避免这种情况发生，一定要切记：**两个类不能相互依赖**。更具体一点的是，**子View绝不可以依赖主View的具体对象**，最好是子View完全不依赖其他，或者只通过一个观察者接口去通知主View。否则你就会发现本属于一个类的控制流却在两个类之前流来流去，本来阅读一个类的成本却要一直在两个类之前切换。如果你发现主View和子View之间避免不了频繁的通信，那说明最初拆分的理由就不充分，子View的弱独立性并不支持它单独成类。



## 总结

1. 不要让View层做数据加工，让View直接拿到可以用来展现的数据
2. 去封装通用控件
3. 创建子View或者View Helper去帮助View分担一部分工作，但切记，不要互相依赖。

## 参考
- [Lighter View Controllers](
https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/)
- [iOS应用架构谈 view层的组织和调用方案](https://casatwy.com/iosying-yong-jia-gou-tan-viewceng-de-zu-zhi-he-diao-yong-fang-an.html)

