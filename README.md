# iOS-Xpath-
iOS Xpath生成原理介绍以及应用

**官方概念：XPath，全称 XML Path Language，即 XML 路径语言，它是一门在XML文档中查找信息的语言。XPath 最初设计是用来搜寻XML文档的，但是它同样适用于 HTML 文档的搜索。**
** BG xpath概念：元素标识，页面元素id。作用就是配合全埋点进行可视化埋点 **
所谓元素标识，也就是通过某些信息，可以唯一标识某个元素。根据概念，需要实现两点：

1. 根据信息，能够筛选出所需的元素，不会遗漏；
2. 根据信息，只能筛选出所需的元素，不会多查。
例如：根据某些关键信息，可以唯一定位 App 中商品详情页面的 “加入购物车” 这个按钮元素。需要保证既不会匹配成 “立即购买” 按钮，也不会匹配成商品列表或其他页面的 “加入购物车” 按钮。
![企业微信20220701-111001.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656645016_95.png?raw=true)
![企业微信20220630-182724.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656584875_78.png?raw=true)
### 1.介绍xpath生成原理
**响应者和响应链**
我们知道，UIResponder 是 UIKit 框架中响应用户操作的基类，我们熟知的 UIView、UIViewController、UIApplication、UIWindow 都是直接或间接继承自 UIResponder 的子类，因此它们的实例都可以响应用户交互行为，从而构成了响应者。在用户操作 App 过程中，由离用户最近的 view 向系统层层传递，从而构成了响应者链，例如：interactive view –> superview(nextResponder) –> ..... –> viewController –> window –> Application –> AppDelegate，如图
![123e69b0d654adfb1e5c780682436a8a.jpg](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656660595_86.jpeg?raw=true)

**xpath = VCClassName + ViewPath + Position**
> VCClassName：即页面名称，也就是元素所在当前页面 viewController 的类名。
> ViewPath： 元素路径，根据元素响应者链拼接而成的字符串，以当前元素所在 viewController 的 view 作为起点。
> Position：表示元素位置，只有列表类型元素才支持，用于支持是否限定元素位置。UITableViewCell 的 position 结构即 "indexPath.section : indexPath.row"。

以加入购物车为例：
![企业微信20220701-111840.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656645560_45.png?raw=true)
对应的一个viewTree
![企业微信20220701-113752.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656646693_48.png?raw=true)

``` 
ProdDetailNewViewController/UIView/BGProdDetailBottomBarView/UIView
// 能确认是Add To Cart按钮吗,有什么问题？
```

``` 
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[2]/UIView[0]
// 加上同层级index,可以确认Add To Cart按钮，还存在什么问题？
```
![企业微信20220701-134947.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656654629_100.png?raw=true)
当上一层级新增了一个UIView。!!#ff0000 Add To Cart按钮Xpath就变了!!。

``` 
// BGProdDetailBottomBarView 下标index变成了3
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[3]/UIView[0]
```
``` 
// 再优化，index取同层级相同类的index
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[0]/UIView[0]
```
![企业微信20220701-141131.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656655922_71.png?raw=true)

``` 
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[0]/UIView[0]
// 这个xpath变成了pre order按钮
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[0]/UIView[1]
// add to cart xpath也变了
```
``` 
// 再优化, 加上元素内容
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[0]/UIView[1]/Add To Cart

// 虽然可以解决，但是存在几个问题：1.多语言问题。 2.元素本身没有内容，需要找到Label元素。 
```

``` 
// 最终优化：特殊处理，index不稳定，手动设置index
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[0]/UIView[AddToCart]
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[0]/UIView[PreOrder]
ProdDetailNewViewController/UIView[0]/BGProdDetailBottomBarView[0]/UIView[BuyNow]
```

***cell的xpath比较特殊，不能通过index，可以通过indexPath去替代index***
![企业微信20220701-143209.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656657151_35.png?raw=true)

``` 
// 通过indexPath去替代index
CategoryProdsViewController/UIView[0]/UICollectionView[0]/CateProdCardCell[0][-]/[0][22]
```

到这里基本可以通过xpath标识元素。

###2.介绍spmXpath
**实际场景中，大多数UI展示是通过接口下发的json动态展示，比如列表，列表可以无限加载，如果要对列表坑位进行可视化配置，现在的xpath就无法满足。于是必须有一种专门用来配置的xpath，这里统称为spmXpath。**
![企业微信20220701-144944.png](https://github.com/nibaJin/iOS-Xpath-/blob/main/img/tapd_30391015_1656658284_81.png?raw=true)

``` 
xpath:
.../HomeActivityView[0]/TPKeyboardAvoidingScrollView[0]/UIView[0] // game
.../HomeActivityView[0]/TPKeyboardAvoidingScrollView[0]/UIView[1] // coupons
.../HomeActivityView[0]/TPKeyboardAvoidingScrollView[0]/UIView[2] // group buy
```
``` 
spmXpath:
.../HomeActivityView[0]/TPKeyboardAvoidingScrollView[0]/UIView[瀑布流标签] // game
.../HomeActivityView[0]/TPKeyboardAvoidingScrollView[0]/UIView[瀑布流标签] // coupons
.../HomeActivityView[0]/TPKeyboardAvoidingScrollView[0]/UIView[瀑布流标签] // group buy
```
``` 
xpath:
CategoryProdsViewController/UIView[0]/UICollectionView[0]/CateProdCardCell[0][-]/[0][0]
CategoryProdsViewController/UIView[0]/UICollectionView[0]/CateProdCardCell[0][-]/[0][1]
CategoryProdsViewController/UIView[0]/UICollectionView[0]/CateProdCardCell[0][-]/[0][2]
。。。
```
``` 
spmXpath:
CategoryProdsViewController/UIView[0]/UICollectionView[0]/CateProdCardCell[0][-]/[0][瀑布流标签]
```
###3.app当中实际应用
**xpath：主要用来标识当前页面元素。 **
> 1.曝光用来标识元素是否曝光
> ...其他用途
**spmXpath: 用来可视化配置映射spm, 需要稳定的路径。**
> 1.设置瀑布流.
> 2.稳固xpath.
> 3.手动设置xpath.
> ...其他用途


###4.存在的问题讨论点：
1.如果层级发生变化，xpath也会相应的变化，配置便随之实效？ 
