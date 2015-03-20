Android 测试驱动开发-从工从具到实践
---

测试驱动开发（Test-driven development，简称TDD）是极限编程中倡导的程序开发方法，以其倡导先写测试程序，然后编码实现其功能得名。      

- 摘自维基百科     
![flow](http://img.blog.csdn.net/20150319132142325)        

## 上图描述了测试驱动开发的过程1. 编写一个测试 2. 运行所有测试, 检查新加入的测试会失败 3. 编写产品代码4. 运行测试5. 重复前面步骤直至测试通过6. 重构代码7. 重复整个过程

在互联网应用开发领域, TDD已经发展得较为成熟, 开发者可以方便地从网上找到教程和下载辅助工具. 然而, 在移动开发领域, TDD起步晚, 网上很难找到比较完整的材料.       笔者总结了最近一段时间在国内某大型快递公司实施TDD的经验, 希望能为挣扎于代码泥潭的朋友提供一些帮助, 同时为仍在纠结于是否实施TDD开发的同学带来一点信心.


首先, 介绍一下团队组成情况. 目前, 团队有11名成员, 包括两个业务人员, 一个项目经理, 一个测试, 以及7个开发人员. 其中开发人员包括两名顾问, 三名快递公司的内部开发人员以及两名第三方公司外派人员. 7个人中, 仅两位顾问和一名内部员工有较丰富的Android开发经验, 另外, 绝大多数人不了解测试驱动开发.       

![coverage](http://img.blog.csdn.net/20150319132525603)      

如上图所示, 左侧为项目开始阶段, 测试数量增长趋势变化大, 表明团队需要花时间去理解和掌握, 同时, 测试覆盖率在项目初期有比较大的起伏. 右侧为项目进行三个月时间以来的情况, 测试数量平稳增长, 覆盖率波动变小, 行级覆盖率和分支覆盖率都稳定在90%以上.

## 测试工具的选择
* **JUnit**     单元测试必备工具。
* **Robolectric**   Android本身自带一套单元测试框架, 但是基于该框架编写测试真正执行前需要经历编译, 打包以及部署到测试设备众多环节, 该过程耗时长, 难以用于测试驱动开发. Robolectric重新实现了应用运行时所依赖的库文件, 使得测试可以脱离android运行时执行(运行在JVM), 并在不改变默认行为的基础进行了扩展, 以便于测试环境准备以及测试结果验证. 具体原理这里不在赘述, 感兴趣的朋友可以参考源码或者到Google Group里提问. 截止笔者写此文时, Robolectric最新版本为2.1.beta，项目中使用版本为1.1.     
* **fest**     fest(http://fest.easytesting.org/)是一套测试验证工具集, 包括fest-core, fest-android, fest-reflect等组件. 借助这些组件, 不仅省去数组是否为空, 列表是否为空, 列表内容验证等繁琐的代码, 而且fest-android里包含了对View属性的验证, 比如验证View的visibility时可以使用isVisible方法.  同时, 扩展fest也非常方便, 这使定制应用专属测试框架变得异常简单.     
* **mockito**     著名的Mock框架, 使用它可以方便创建mock, stub以及dummy对象, 其语法如when(mock.someMethod()).thenReturn(value), 大大提高了测试代码可读性.


## 如何进行TDD
TDD并不意味着有另外一个人会告诉你怎么做, 而是需要开发自己结合要完成的功能, 自己计划并完成编码. 对于刚接触TDD的朋友, 往往不知从何下手。 对此, 我们一般会选择一名有经验的开发人员与其先进行一段时间的结对开发.在这段时间内, 有经验的开发人员会与新人一起讨论并制定一份任务列表, 并基于此开始实现第一个测试. 然后重复上述过程, 直至新人对代码架构以及工作流程熟悉. 

项目要提供一个类似Google Suggestions一样的功能, 用户输入任意汉字, 下拉列表中显示包含此汉字的所有产品信息. 以此为例.     
![google](http://img.blog.csdn.net/20150319132909236)   

首先, 该功能可以拆分为两个子任务。   
1. 当输入文字有对应产品时, 要显示所有匹配的产品.2. 当输入文字无对应产品时, 列表为空.    
这时, 虽然可以编写基本的测试, 但是实现起来仍比较困难, 进一步讨论, 我们可以发现:   

1. 当输入文字有对应产品时, 要显示所有匹配的产品.    	1.1 从数据库读取产品信息, 并转换成Product对象的数组  	1.2 Product对象信息在界面(ListView)中的显示     
	至此, 我们可以编写出至少三个测试:       
数据库层的dbHelper.loadProducts、UI层的adapter.getView以及一个Activity级别的集成测试should_display_matched_product_categories, 开发人员可以自底向上逐个击破, 最后完成该集成测试.        
任务分解通常有以下几个技巧或者注意事项:      1. 从常规路径出发, 先按照业务划分出任务.2. 注意边界条件, 如数组为空的情况.3. 按照功能划分出任务后, 可以结合代码架构进一步做技术上的划分. 如上面的例子可以看出, 数据库层和UI层被拆分成不同的任务.4. 任务拆分的粒度应该以开发人员可以写出单元测试, 并且实现测试(编写对应产品代码)的时间不会过长为准.5. 任务拆解过程中, 可以进行一些初步的代码设计, 划分出层次.## TDD实战
1, 创建测试类
```java    @RunWith(RobolectricTestRunner.class)    public class ProductListActivityTest {    }
```    在测试类开始要以注解的方式指定TestRunner为RobolectricTestRunner.2, 编写第一个测试
```java    @Test    public void should_display_matched_product_categories() {        prepareProductListData();        displayProductList();        enterTextOn(searchCriteriaView(), "手机");        assertThat(productList()).has(numberOfItems(2));        assertThat(productList()).has(text("手机壳"));        assertThat(productList()).has(text("手机贴膜"));    }
```    测试通常分为三部分第一部分, 准备测试数据和测试上下文, 如上面的测试中, 我们做了两件事, 插入测试数据到数据库, 创建ProductList的UI
```java    private void prepareProductListData() {        List<Product> data = asList(new Product("手机壳"), new Product("手机贴膜"));   
	DatabaseHelper.getInstance(Robolectric.application)					.insertData(data);
    }    private void displayProductList() {        productListActivity = new ProductListActivity();        productListActivity.onCreate(null);    }
```    第二部分, 触发要测试的功能. 
```java    public static void enterTextOn(View view, String text) {        TextView textView = (TextView) view;        textView.setText(text);    }
```
第三部分, 验证结果.
```java        assertThat(productList()).has(numberOfItems(2));        assertThat(productList()).has(text("手机壳"));
```       


如上面代码所示, fest增强了验证代码的表现力. 其中, numberOfItems()和text()两个方法皆为我们开发人员自己编写的matcher, 详细扩展方法, 请参看官方文档, 这里不再赘述.很多时候, 我们需要对遗留系统加测试, 这时往往发现加测试很难.根据我们的经验, 难以测试的代码通常涉及UI和通信, 另外是因为我们的代码过于集中(长), 逻辑太复杂.      
1. 对于UI, 借助robolectric该部分的复杂度已经大大降低. 除了使用robolectric, 还可以参考马丁福勒编写的文章 GUI Architectures (http://martinfowler.com/eaaDev/uiArchs.html ), 借助不同的架构模式对UI进行解耦和测试.    
2. 对于通信, 一般有两种方案, 一是借助Robolectric的http组件, 如使用Robolectric.addPendingHttpResponse(); 和Robolectric.addHttpResponseRule();方法, 虚拟Http回复.二是借助一些比较成熟稳定的Http服务框架, 如Moco, 以集成测试的方式验证通信层的逻辑. 无论哪种方案, 都应该专注于通信逻辑.      
3. 对于代码过长, 逻辑复杂的问题, 常见的解决方案就是组件化. 除了上面提到的通信, 序列化, 数据库的模块划分, 我们还可以将UI进一步细化. 如一个几百行的Activity, 我们曾经成功将activity里的10几个android自带控件抽离到3个具有一定业务含义的组件中, 最终每个组件只有约不到300行代码.## 与TDD相结合的开发实践1. 持续重构, 与定期进行大面积代码重构相比, 团队采取了更细粒度, 且成本更为可控的重构方式, 每完成一个或几个测试就进行一次重构, 从删除重复代码, 消除代码中的坏味道开始.      另外, 团队成员统一使用了基于intellij的android studio开发环境. 该集成开发环境对常用的重构手法提供了便利的自动化支持, 只需几个按键便可完成较为复杂的操作, 从而进一步降低了重构的风险和成本.     
2. 坚持团队内的代码评审. 每天下午下班的半个小时到一个小时, 团队成员会集中到投影仪前, 一起审查当日提交的代码.      值得一提的是, 代码评审不仅要查看产品代码, 而且要看测试代码. 两者可以有略不同的标准, 如测试代码追求更高的可读性, 像本文中的示例代码一样, 用下划线分隔不同单词. 一个检查测试可读性的好方法是让写此代码以外的人来解释这个测试的含义.     代码评审中发生的反馈一定落实到具体的修改负责人头上, 确保大家发现的问题及时被修正.     
3. 项目健康度可视化项目搭建了基于Jenkins的持续集成服务器, 每次提交都会更新测试数量, 测试覆盖率, lint代码质量评估等的报告。        
![ci](http://img.blog.csdn.net/20150319133501708)         

同时, 团队一致决定将代码分支覆盖率低于85%作为构建失败的因素, 结合CI构建报警灯和提示音, 确保问题早发现, 早处理.     


## 结尾
“路漫漫其修远兮, 吾将上下而求索”,笔者深知写好代码不是一朝一夕和一人可以完成的事情, 分享此文, 一来希望能对前一段时间的工作做一个总结, 同时也希望能交到正在或者打算在android平台尝试TDD以及喜欢移动开发的朋友.
