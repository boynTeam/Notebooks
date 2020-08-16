> 本文来源于Google公司雇员的一篇文章:[Understanding Semantic Commit Messages Using Git and Angular](https://nitayneeman.com/posts/understanding-semantic-commit-messages-using-git-and-angular/)
>
> 从这篇文章中精炼出提交Git记录的规范,仅供参考,若与现规范不同还望求同存异
>
> 关于例子,我们可以去Angular的库中看看他们的提交记录:[Commits](https://github.com/angular/angular/commits/master)

# 提交消息的标准结构

我们的提交消息应为三段式的结构,即分为头部,主题与注脚

![](http://imageblog.boyn.top/202002150915_802.png)

接下来我们以一个提交Pull Request的例子,来看看怎么拼接出规范的提交消息

## 头部

头部的消息是必须的,它主要包含提交的类型(后面会说),简要的工作概括(最好在一行之内)

它本身包含三个部分:

- 类型 - 一个简单的单词或缩写用于表示本次git更改的类别
- 范围 - 通常是包名(java/go...)来表示本次修改的范围
- 题目 - 简要概括本次提交的内容

我们来看一条实际的操作

```shell
git commit -m "fix(core): remove deprecated and defunct wtf* apis"
```

其类型为fix,意味着这条提交类似于修复的功能,其范围为core,即修复core包中的bug,而后面是题目,概括了本次提交的内容为删除冗余代码和废止了一项功能

## 主题

主题是可选的,它的主要内容可以是描述修改的动机,想法等等,也可以是对头部中提交内容做进一步的更详细的解释.我们来看一条git消息

```shell
git commit -m "fix(core): remove deprecated and defunct wtf* apis" -m "These apis have been deprecated in v8, so they should stick around till v10, but since they are defunct we are removing them early so that they don't take up payload size."
```

在刚刚写好了头部的基础上,增加了对这条修改的动机

需要注意的是,我们使用了多个`-m`来进行消息的分割,并不仅仅是语义上的,在显示中,头部与主题部分会以一个空行作为分割

## 注脚

同样地,注脚也是可选的,它可以用于标识此次修改带来的变化后果,以及宣布对某项issue的解决等等

```shell
git commit -m "fix(core): remove deprecated and defunct wtf* apis" -m "These apis have been deprecated in v8, so they should stick around till v10, but since they are defunct we are removing them early so that they don't take up payload size." -m "PR Close #33949"
```

最后这条消息表明,本次的修改来源于#33949的Pull Request



我们可以看到,成果的git记录是这样的

![](http://imageblog.boyn.top/202002150931_157.png)

# 提交类型

我们刚刚看到,如果是修复bug的类型,可以使用fix来进行声明,为了规范,Angular团队预定义了几个声明的类型

![](http://imageblog.boyn.top/202002171058_926.png)

## build

build类型用于表示与构建项目,依赖变更等相关的更改,例如改变了maven中的哪些依赖,变更了makefile中的哪些语句等等

![](http://imageblog.boyn.top/202002150934_960.png)

## ci

ci是(  continuous integration )持续集成的缩写,ci用于表示与持续集成工具,部署系统等等的更改,比如修改了CI的脚本等内容

![](http://imageblog.boyn.top/202002150936_994.png)

## docs

doc类型通常表示对于项目中注释或者文档的完善

![](http://imageblog.boyn.top/202002150936_372.png)

## feat

feat表示与项目新功能的增加有关的操作

![](http://imageblog.boyn.top/202002150938_777.png)

## fix

修复bug

![](http://imageblog.boyn.top/202002150938_44.png)

## perf

perf用于表示与表现改善的提交,即优化程序的速度等方面

## refactor

refactor表示重构,用于表示对项目的功能没有太多的更改,但是对项目的代码进行了重构的提交

## style

style表示对代码观赏性的提升

## test

test表示新增或修改的测试

# 规范化的好处

规范了git提交消息之后,好处主要有如下两点:

1. 可以更加快捷地查找到git提交信息

比如说我们要查找类型不是feat,fix,perf的类型

```shell
git log --oneline --grep "^feat\|^fix\|^perf"
```

又比如说想要统计feat类型提交的个数

```shell
git log --oneline --grep "^feat" | wc -l
```

2.  规范化消息之后,在自动发布新版本的时候会更加有效

![](http://imageblog.boyn.top/202002150944_825.png)