---
layout: post
category : 学习笔记 
keywords: "nutz,源码学习"
description : "一种以 Map, List 接口对象所组织起来的结构体系. 类似于JSON结构便于JAVA在内存中处理的结构. 主要提供键值对, 与列表的有机组合, 因这种结构只由Map, List组成, 因些称其为Mapl结构."
tags : [nutz,源码学习]
---
 
如标题所示，这篇文章来说说两个工具类MaplRebuild和MapMerge。
- 前者用来对已有的mapl结构进行重构。包括添加，删除，读取指定路径的值 比如针对下面一个mapl结构的数据
    
        {'user':[{'name':'jk', 'age':12},{'name':'nutz', 'age':5}]}
    
- 我们要在这个路径：user[0].test 增加值"hello"使其变为

        {'user':[{'name':'jk', 'age':12,'test':'hello'},{'name':'nutz', 'age':5}]}    

- 而MaplMerge 主要用来将多个mapl结构数据合并起来，该类只有一个接口：mergeItems(Object... objs)


<!--break-->

{% include JB/setup %}


## MapRebuild

该类提供了对mapl结构的方便操作，遥想当年（呃，其实也就这两年），偶有处理json数据的时候，需要给json某个属性更新个值的时候，不得不，一层一层的深入进去操作（咳咳）。
现在好了，有了这个神器，一个方法搞定。

该类一开是定义了针对mapl的三种操作

        enum Model {
            add, del, cell
        }
添加（也可以用于更新），删除，读取，分别对应MapRebuild类中的put，remove，cell方法。

![maplrebuild_public_method]({{ site.img_url }}/2016/nutz/maplrebuild_public_method.png)

- 这三个方法内部实现都调用了MapRebuild的两个私有类：init,inject。   
- 其中init方法是初始化参数

        private void init(String keys, Object obj) {
            this.keys = Lang.arrayFirst("obj", Strings.split(keys, false, '.'));
            this.val = obj;
            this.arrayItem = 0;
        }
- inject则是具体根据路径去操作改路径的值。      
    - inject方法中根据给定的路径一层一层深入到目的地，然后更新值，我们知道mapl结构的数据结构是有map和list组成的，所以inject方法中深入到指定路径进行操作的时候就是针对map和list的操作。
    - 所以inject中根据数据特征会去调用injectList和injectMap来分别处理list和map的操作
    - 最终分别调用的是list的add，remove，get方法个map的put，remove，get方法


## MapMerge

看名字就知道，这个是用来合并多个mapl结构数据的。
改接口只有一个方法mergeItems，将多个mapl数据合并成一个。
该方法传入的参数，必须是要么全部是list结构的数据，要么全部是map结构的数据，然后根据类型，分别调用mergeList 和mergeMap方法
其中list比较好处理，只要将多个list中的元素一个一个读取出来放到一个list中即可，实现中对相同元素不重复保存，实现如下所示：

        private Object mergeList(Object... objs) {
            List list = new ArrayList();
            for (Object li : objs) {
                List src = (List) li;
                for (Object obj : src) {
                    if (!list.contains(obj)) {
                        list.add(obj);
                    }
                }
            }
            return list;
        }
 
而针对map结构的合并处理，需要判断是否key有一样的，如果没有，直接存入即可，如果有一样的，将相同key下的值合并到一个list中再存放在统一个key下。

该类并没有去合并不同结构的mapl数据，比如合并一个顶级结构是list或者顶级结构是map的数据。因为这样合并的话，不太好拿捏到底是合并为一个map还是合并为一个list。

这里仅仅是一个想法，比如是否可以通过指定第一个参数为顶级容器来合并后面多个数据结构：
   
    //第一个参数顶级结构如果是map，就将后面数据按照索引指定key值，放在第一个map中，返回map；
    //第一个参数顶级结构如果是list，就将后面数据放在第一个参数中，返回list
    mergeInFirst(Object... objs)

