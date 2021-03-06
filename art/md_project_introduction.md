
## 想法来源
因为之前一直用的是`XRecyclerView`，此库可以进行下拉刷新和加载更多，但是有很多致命的问题，例如：

 - 1.自定义下拉刷新和加载更多布局时不方便，只能设置简单的样式，用的时候还是拷贝的项目的源码，然后在其中进行更改。
 - 2.不能在此基础上使用`SwipeRefreshLayout`，会有滑动冲突问题。
 - 3.不能在`CoordinatorLayout` + `AppBarLayout`中使用(作者后期已经修复，但是由于项目是拷贝的所以不便更新维护)。
 - 4.不能removeHeaderView(好像已经完善)
 - 5.不能设置EmptyView，或者不是以一个item形式添加，导致不能同时出现头布局和空布局
 - 6.不能添加FooterView
 - 7.不能设置item点击事件
 - 8.需要自己实现BaseRecyclerViewAdapter

为了解决上面的问题，我在项目中到处打补丁，但是治标不治本，导致我不得不选择新的RecyclerView库来满足我的业务需求。
于是看中了万众瞩目的`BaseRecyclerViewAdapterHelper`，这个现有18.7k star的开源库。

BRVAH 几乎可以解决上面所有的问题，并且使用简单，我在公司新项目中使用了它，但是不久我就遇到了新的问题：

 - 1.addHeaderView()是在一个item中操作的，导致我不能顺利使用锚点(滑动时，通过定位第一个item的位置来更改指示器)。如果使用多类型item会复杂很多，我需要对数据实现对应的多类型接口来达到目的，并在滑动时有明显的卡顿效果。
 - 2.不满一屏自动加载。这个功能导致有很多不必要的接口请求，每次进入不满一屏的页面都会请求两次，实在看不过去。设计讲究`所见即所得`，不要乱替我执行动作行为。[查看这位无奈的程序员](https://github.com/CymChad/BaseRecyclerViewAdapterHelper/issues/2942)。

这应该是此库比较大的两个槽点，导致我使用起来还是不那么称心如意。当然此库绝大部分的功能都是好用的。

## ByRecyclerView

于是就有了`ByRecyclerView`，它基本解决了上面的所有问题：

 - 不满一屏，上拉才执行加载更多；满一屏后触底加载更多
 - 可设置自己的下拉刷新头，并可直接自定义下拉刷新和加载更多布局
 - 也可配合`SwipeRefreshLayout`使用
 - 可添加/删除 HeaderView(多类型) / FooterView / EmptyView
 - 可设置item的点击事件/长按事件
 - `ByRecyclerView`与`Adapter`分离，意味着开发者之前使用自定义的`BaseAdapter`，会无缝衔接`ByRecyclerView`，完全可替换`XRecyclerView`，只需更换少量方法。
 - 可设置任意自定义行间距(自带ItemDecoration)
 - 结合`databinding`的`BaseBindingAdapter`
 - 提供`AndroidX`和`Support`包引入

## 当解决BRVAH描述的两个问题时，明白了为什么作者那样处理的原因

### addHeaderView()
 - 如果添加/删除的每个item都是一个类型的item，那么频繁的添加和删除就会难以处理viewType，导致item错乱。但是实际项目中这样的情境很少见，所以这样的情况本库选择了忽视。就是说，用户正常的addHeaderView不removeHeaderView，或者addHeaderView后removeHeaderView不再次addHeaderView，那么都不会有问题。

### 上拉加载机制
- BRVAH使用这样的加载机制，就不用处理用户是否下拉的逻辑，将会无缝适配`SwipeRefreshLayout`等刷新控件。而本控件为了处理不足一屏的加载逻辑而耗费了大量时间，判断是否上拉的核心逻辑是：（手指按下屏幕的纵坐标 - 手指离开屏幕的纵坐标 >= -10）&& （手指离开屏幕的纵坐标 - 手指到达的最高点 < 150）。
- 手指按下屏幕的纵坐标 - 手指离开屏幕的纵坐标 >= -10
	- 因为如果取0，那么手指按下然后直接惯性滑动到底部时，那么不会判定为上拉加载。
- 手指离开屏幕的纵坐标 - 手指到达的最高点 < 150
	- 要执行此段逻辑，是因为当用户上滑后再下滑，这时触发了`SwipeRefreshLayout`的刷新，但是不做判断的话也会触发加载更多。取150是因为`SwipeRefreshLayout`下拉会超过这个值(差不多250左右)，低于这个值比较合理，150的值不大，一般的下拉控件都会超过这个值，如果还是不能适配部分控件，后期还可将这个值暴露出去给开发者更改。

## 最后
这个控件是在`XRecyclerView`基础上大量完善丰富，`BaseByRecyclerViewAdapter`可基本衔接BRVAH的`BaseQuickAdapter`。现基本可以满足开发者的业务功能，当然此库还有很多功能未完善，比如加载动画、长按排序等，后期会慢慢完善。如果有任何问题，可提[Issues](https://github.com/youlookwhat/ByRecyclerView/issues)，或通过下列方式联系我。

## About me
 - **QQ：** 770413277
 - **简书：**[Jinbeen](http://www.jianshu.com/users/e43c6e979831/latest_articles)
 - **Blog：**[http://jingbin.me](http://jingbin.me)
 - **Email：** jingbin127@163.com
 - **QQ交流群：**[![](https://img.shields.io/badge/%E7%BE%A4%E5%8F%B7-727379132-orange.svg?style=flat-square)](https://shang.qq.com/wpa/qunwpa?idkey=5685061359b0a767674cd831d8261d36b347bde04cc23746cb6570e09ee5c8aa)