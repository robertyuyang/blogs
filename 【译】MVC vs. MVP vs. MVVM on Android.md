# MVC vs. MVP vs. MVVM on Android

##### 原文:https://academy.realm.io/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/

关于如果**最优地**把Android应用组织成逻辑模块的实践方法，最近几年一直在不断的演进。开发社区已经大规模地脱离原来单一的Model View Controller（MVC）模式，而更偏爱更佳模块化的，更可测试的模式。

Model View Presenter (MVP) 和 Model View ViewModel (MVVM)是两个最被广泛采用的方案，但是开发者们经常分裂成哪一个更适合安卓开发的不同阵营中。过去这些年无数的博客不断的提出强烈的主张，认为其中一个比另外一个更优秀，然而又过于执着于主观观点的争论而忽略了客观标准的讨论。本文并不会过多的争论孰优孰劣，而是更客观地去探讨这三种方法的价值和潜在问题，这样读者更容易自己去做选择。

为了更好的在实际场景中讨论这三种模式，我们使用一个简单的Tic-Tac—Toe游戏作为例子。


![image](https://images.contentful.com/emmiduwd41v7/3PB84hQgVio4G0i6Eic6kQ/07ab24b16897766cf4e53992ed6cb472/tictactoe.png)

这篇文章的剩余部分会按顺序一步步介绍MVC，MVP，MVVM模式。在每一个章节的开头，我们会给每个一个组件和组件职责下一个通用的定义，然后再来看怎么应用到我们的游戏中去。

这个例子的源代码在这个[GitHub仓库](https://github.com/ericmaxwell2003/ticTacToe)上。你在阅读不同的章节时可以切换到不同对应分支上以方便跟上文章内容(e.g. git checkout mvc, git checkout mvp, git checkout mvvm)。

## MVC

MVC模式把你的程序从宏观上划分成三个主要的职责块。

### Model

对于我们的Tic-Tac-Toe游戏来说吗，model就是**数据+状态+业务逻辑**。可以说是我们程序的大脑。它是不能和view或者controller绑定的，因为这样的话，在绝大多数场景下是无法复用的。

### View

view就是model的**可视化展现**。view的职责是交付一个用户接口（UI）并且在用户与程序发生交互时负责与controller进行沟通。在MVC的架构中，view往往是很“蠢”的，它们不了解底层的model，也不理解“状态”，也不知道如果用户做一些诸如点个按钮，输入一个值这样的操作之后应该作何反应。这的理念就是他们对model知道的越少，他们的耦合就越小，这样他们对更改就越灵活。

### Controller

controller就是一个把程序整合起来的**粘合剂**。它是这个程序如何运行的主控制者。当view告诉controller用户点了一个按钮，controller来决定如何跟model进行相应的交互。当model发生数据变化时，controller可能需要决定适当的更新view的状态。在Android应用的场景下，controller往往是一个Activity或者Fragment。

下图是从上层看我们的Tic-Tac-Toe游戏程序里不同模块的职责：


![image](https://images.ctfassets.net/emmiduwd41v7/2XWsL8SmQEg24GIegMsOOs/a595a0352d2dd26ead78a338c16b8419/MVCsvg.svg)

让我们通过代码来看下这个controller的更多细节：

```

public class TicTacToeActivity extends AppCompatActivity {

    private Board model;

    /* View Components referenced by the controller */
    private ViewGroup buttonGrid;
    private View winnerPlayerViewGroup;
    private TextView winnerPlayerLabel;

    /**
     * In onCreate of the Activity we lookup & retain references to view components
     * and instantiate the model.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.tictactoe);
        winnerPlayerLabel = (TextView) findViewById(R.id.winnerPlayerLabel);
        winnerPlayerViewGroup = findViewById(R.id.winnerPlayerViewGroup);
        buttonGrid = (ViewGroup) findViewById(R.id.buttonGrid);

        model = new Board();
    }

    /**
     * Here we inflate and attach our reset button in the menu.
     */
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu_tictactoe, menu);
        return true;
    }
    /**
     *  We tie the reset() action to the reset tap event.
     */
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.action_reset:
                reset();
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }

    /**
     *  When the view tells us a cell is clicked in the tic tac toe board,
     *  this method will fire. We update the model and then interrogate it's state
     *  to decide how to proceed.  If X or O won with this move, update the view
     *  to display this and otherwise mark the cell that was clicked.
     */
    public void onCellClicked(View v) {

        Button button = (Button) v;

        int row = Integer.valueOf(tag.substring(0,1));
        int col = Integer.valueOf(tag.substring(1,2));

        Player playerThatMoved = model.mark(row, col);

        if(playerThatMoved != null) {
            button.setText(playerThatMoved.toString());
            if (model.getWinner() != null) {
                winnerPlayerLabel.setText(playerThatMoved.toString());
                winnerPlayerViewGroup.setVisibility(View.VISIBLE);
            }
        }

    }

    /**
     * On reset, we clear the winner label and hide it, then clear out each button.
     * We also tell the model to reset (restart) it's state.
     */
    private void reset() {
        winnerPlayerViewGroup.setVisibility(View.GONE);
        winnerPlayerLabel.setText("");

        model.restart();

        for( int i = 0; i < buttonGrid.getChildCount(); i++ ) {
            ((Button) buttonGrid.getChildAt(i)).setText("");
        }
    }
}

```

### 评估

MVC在分离model和view方面起到了巨大的作用。显然model因为不和任何东西绑定因而非常容易测试，而view在单元测试中也没有太多值得测试的东西。然而controller则存在一些问题。

### Contrller的隐患：

- *可测试性*：controller是和Android的API紧密联系的使得它很难单元测试。
- *模块化和灵活性*：controler于view紧耦合。它可能就是view的一个扩展。当我们改变view，就必须回过头去修改controller。
- *可维护性*：时间一长，尤其对那些使用[瘦model](https://martinfowler.com/bliki/AnemicDomainModel.html)的程序，越来越多的代码会转移到controler里，让程序膨胀而且脆弱。
- 
那我们怎么解决这些问题呢？MVP来帮忙了。

## MVP

MVP把controller拆开了，这样view/activity的耦合就是可以接受的了，而不是一定要把它们绑定到
controller的其余职责中。下面会有更多讨论，但我们还是先对比着MVC从最基本职责定义讲起。

### Model
和MVC相同

### View

这里唯一个改变是，Activity/Fragment现在成了view的一部分。我们不再去阻止它们一定要手拉着手的自然趋势。更好的实现是，让Activity去实现一个view interface，这样presenter就可以跟这个接口来交互了。这样presenter和具体的view之间的耦合就消除了，同时也更方便用一些仿制（mock）view来对presenter做单元测试。

### Presenter

presenter基本上就是MVC里的controller，只不过它不会完全和view绑定到一起，而是只依赖一个接口。这样就解决了我们在MVC中对可测试性和模块化/灵活性的担忧。事实上，更较真的人会认为presenter应该永远不依赖任何Android的api或者代码。

我们再试一下在我们的app里MVP应该长成什么样子：

![image](https://images.ctfassets.net/emmiduwd41v7/79cWBA2VhuiuUeK6uu4WAE/59f8d55cd0997100c718e36b84ad2b37/MVPsvg.svg)

下面有presenter的更多的细节，你首先会注意到的每一个操作的意图是如此的简单和明确。它只会告诉view要展示*什么*，而不用再告诉view*如何*展示。

```
public class TicTacToePresenter implements Presenter {

    private TicTacToeView view;
    private Board model;

    public TicTacToePresenter(TicTacToeView view) {
        this.view = view;
        this.model = new Board();
    }

    // Here we implement delegate methods for the standard Android Activity Lifecycle.
    // These methods are defined in the Presenter interface that we are implementing.
    public void onCreate() { model = new Board(); }
    public void onPause() { }
    public void onResume() { }
    public void onDestroy() { }

    /** 
     * When the user selects a cell, our presenter only hears about
     * what was (row, col) pressed, it's up to the view now to determine that from
     * the Button that was pressed.
     */
    public void onButtonSelected(int row, int col) {
        Player playerThatMoved = model.mark(row, col);

        if(playerThatMoved != null) {
            view.setButtonText(row, col, playerThatMoved.toString());

            if (model.getWinner() != null) {
                view.showWinner(playerThatMoved.toString());
            }
        }
    }

    /**
     *  When we need to reset, we just dictate what to do.
     */
    public void onResetSelected() {
        view.clearWinnerDisplay();
        view.clearButtons();
        model.restart();
    }
}
```

为了让activity和presenter不绑定到一起，我们创建了一个接口让Activity去实现。在测试中就可以用一个基于这个接口的仿制（mock）类来测试presenter和view之间的交互。

```
public interface TicTacToeView {
    void showWinner(String winningPlayerDisplayLabel);
    void clearWinnerDisplay();
    void clearButtons();
    void setButtonText(int row, int col, String text);
}
```

### 评估
现在的方式就清晰很多，我们可以简单的对presenter的逻辑进行单元测试因为它并不和任何的特定的Android view和APIs。而且这个presenter也可以跟任何其他的view一起工作，只要这个view实现了**TicTacToeView**接口

### Presenter的隐患
- 可维护性：Presenter，跟controllers一样，随着时间的推移，很容易就会把业务逻辑越加越多。到某个时刻，开发者就会发现，presenter也变得庞大笨重难以切割。
- 
当然，仔细的开发者在程序随时间的变迁中能优雅的低档这种诱惑，然而，MVVM能通过更精简的工作来帮忙解决这个问题。

## MVVM

Android上利用数据绑定的MVVM有着易测试性和模块化的优点，同时减少了很多的我们用来连接view和model的粘合作用的代码。

让我们看下MVVM的组成部分。

### Model

和MVC一致

### View

view会通过一种灵活的方式和viewModel暴露出来的可观察的变量和动作绑定在一起。后面会详细介绍。

### ViewModel

ViewModel的作用是用来封装model并且把view需要的可观察的数据准备好。它还会把view的一些事件截取下来发给model。尽管如此ViewModel和View并不强行绑定。

Tic-Tac-Toe程序的高层次拆解：
![image](https://images.ctfassets.net/emmiduwd41v7/3Zt3epjcm40kYyygqo8kWG/d9bf5bb78a800eb434a1354bc812c15e/MVVMsvg.svg)

让我们再仔细看一下新的组件，从ViewModel开始：

```
public class TicTacToeViewModel implements ViewModel {

    private Board model;

    /* 
     * These are observable variables that the viewModel will update as appropriate
     * The view components are bound directly to these objects and react to changes
     * immediately, without the ViewModel needing to tell it to do so. They don't
     * have to be public, they could be private with a public getter method too.
     */
    public final ObservableArrayMap<String, String> cells = new ObservableArrayMap<>();
    public final ObservableField<String> winner = new ObservableField<>();

    public TicTacToeViewModel() {
        model = new Board();
    }

    // As with presenter, we implement standard lifecycle methods from the view
    // in case we need to do anything with our model during those events.
    public void onCreate() { }
    public void onPause() { }
    public void onResume() { }
    public void onDestroy() { }

    /**
     * An Action, callable by the view.  This action will pass a message to the model
     * for the cell clicked and then update the observable fields with the current
     * model state.
     */
    public void onClickedCellAt(int row, int col) {
        Player playerThatMoved = model.mark(row, col);
        cells.put("" + row + col, playerThatMoved == null ? 
                                                     null : playerThatMoved.toString());
        winner.set(model.getWinner() == null ? null : model.getWinner().toString());
    }

    /**
     * An Action, callable by the view.  This action will pass a message to the model
     * to restart and then clear the observable data in this ViewModel.
     */
    public void onResetSelected() {
        model.restart();
        winner.set(null);
        cells.clear();
    }

}
```
view部分的一些摘录，能看到这些变量和动作是如何绑定的：


```
<!-- 
    With Data Binding, the root element is <layout>.  It contains 2 things.
    1. <data> - We define variables to which we wish to use in our binding expressions and 
                import any other classes we may need for reference, like android.view.View.
    2. <root layout> - This is the visual root layout of our view.  This is the root xml tag in the MVC and MVP view examples.
-->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <!-- We will reference the TicTacToeViewModel by the name viewModel as we have defined it here. -->
    <data>
        <import type="android.view.View" />
        <variable name="viewModel" type="com.acme.tictactoe.viewmodel.TicTacToeViewModel" />
    </data>
    <LinearLayout...>
        <GridLayout...>
            <!-- onClick of any cell in the board, the button clicked will invoke the onClickedCellAt method with its row,col -->
            <!-- The display value comes from the ObservableArrayMap defined in the ViewModel  -->
            <Button
                style="@style/tictactoebutton"
                android:onClick="@{() -> viewModel.onClickedCellAt(0,0)}"
                android:text='@{viewModel.cells["00"]}' />
            ...
            <Button
                style="@style/tictactoebutton"
                android:onClick="@{() -> viewModel.onClickedCellAt(2,2)}"
                android:text='@{viewModel.cells["22"]}' />
        </GridLayout>

        <!-- The visibility of the winner view group is based on whether or not the winner value is null.
             Caution should be used not to add presentation logic into the view.  However, for this case
             it makes sense to just set visibility accordingly.  It would be odd for the view to render
             this section if the value for winner were empty.  -->
        <LinearLayout...
            android:visibility="@{viewModel.winner != null ? View.VISIBLE : View.GONE}"
            tools:visibility="visible">

            <!-- The value of the winner label is bound to the viewModel.winner and reacts if that value changes -->
            <TextView
                ...
                android:text="@{viewModel.winner}"
                tools:text="X" />
            ...
        </LinearLayout>
    </LinearLayout>
</layout>

```

> 专家提示：要着重使用属性这个工具。注意在上面的例子里属性用来显示赢家的值和是否显示的设置。如果你不去设这些，那么在设计阶段就很难看清楚具体的逻辑。

>  关于Android上的MVVM和数据绑定的另一点资料。这个只是介绍了一点数据绑定能做的事情的皮毛。我强烈建议你去读一下[Android数据绑定](https://developer.android.com/topic/libraries/data-binding/index.html)文档来学习一下这个强有力的工具。在这一页底部还有一个Google Android Architecture Blueprints项目页，里面有一些更好的MVVM和数据绑定例子。

### 评估

现在单元测试甚至更简单了，因为你对view已经没有任何依赖。在测试的时候，你只需要有安正在数据变化时被观察的变量是否被正确地赋值了。没必要像在MVP中那样需要做一个仿制（mock）的view了。

### MVVM的隐患：
- 可维护性：因为我们既可以绑定变量也可以绑定表达式。大量外部的展现逻辑会随着时间的推演而泛滥，会在我们的XML的代码明显增多。为了避免这一点，永远要直接从viewmodel去获取值，而不是尝试从在view的绑定表达式里去计算或者扩展他们。这样计算部分就能狗被很好地单元测试。
- 
## 总结

MVP和MVVM都比MVC能够更好地拆解你app中的模块和职责，但是他们也给你的app增加了一定的复杂性。对于一个只有一两屏的简单app来说，MVC也是可以适用的。因为响应式编程的引入和更少的代码量，MVVM是很有吸引力。

那么哪种模式是最适合你的呢？如果你在MVP和MVVM中做选择，很多决定其实最后都会取决于个人喜好，不过，去实践会帮助你更好的理解他们的优缺点。

如果你对更多的MVP和MVVM的例子感兴趣。我鼓励你去看一下 [Google Architecture Blueprints](https://github.com/googlesamples/android-architecture)项目。还有很多blog文章会对MVP实现进行更深层次的探讨。
