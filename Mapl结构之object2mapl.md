---
layout: post
category : 学习笔记 
keywords: "nutz,源码学习"
description : "一种以 Map, List 接口对象所组织起来的结构体系. 类似于JSON结构便于JAVA在内存中处理的结构. 主要提供键值对, 与列表的有机组合, 因这种结构只由Map, List组成, 因些称其为Mapl结构."
tags : [nutz,源码学习]
---


Mapl的使用方式参考[官方文档](http://nutzam.com/core/maplist/overview.html),灰常详细了。这里不再赘述。
如图所示，Mapl有以下方法，可供使用

![Mapl方法示例]({{ site.img_url }}/2016/nutz/mapl.png)

Mapl类中并没有实现具体的操作方法，分别调用的具体的工具类：

- ObjConvertImpl，将maplist数据转换为指定的java对象
- ObjCompileImpl，将java对象转换为maplist结构。
- MaplRebuild，对maplist结构进行修改操作，可以修改，删除制定路径的值，可以合并多个maplist为一个。
- MaplMerge， 将多个maplist合并为一个。
- FilterConvertImpl，针对maplist进行过滤，可以根据路径去掉不想要的路径，或者仅保留制定路径的值。
- StructureConvert，可以根据模板将maplist结构进行转换，比如将{"name":"华为","date":"1992/01/02"}转换为:{"entName","华为","esDate":"1992/01/02"}


<!--break-->

{% include JB/setup %}



### ObjCompileImpl
ObjCompileImpl 是MaplCompile接口的默认实现，该接口只有一个方法parse，将用来将java对象转换为maplist，
parse方法中，将传入的参数（java对象）归类为三种类型进行处理：单个pojo对象，集合，数组，map，
这三个对象分别对应了 pojo2Json,coll2Json,array2Json,map2Json方法。

    if (obj instanceof Collection || obj.getClass().isArray()) {
        List<Object> list = new ArrayList<Object>();
        memo.put(obj, list);
        // 集合
        if (obj instanceof Collection) {
            return coll2Json((Collection) obj, list);
        }
        // 数组
        return array2Json(obj, list);
    } else {
        Map<String, Object> map = new LinkedHashMap<String, Object>();
        memo.put(obj, map);
        // Map
        if (obj instanceof Map) {
            return map2Json((Map) obj, map);
        }
        // 普通 Java 对象
        return pojo2Json(obj, map);
    }
    
- pojo2Json方法将java对象存储成一个个键值对，最后返回Map对象。当然如果对象的属性也是一个pojo对象，继续调用parse方法继续处理即可。
- map2Json方法和pojo2Json类似，只不过是map本身就是一个个键值对。
- coll2Json和array2Json 方法将传入的java对象直接放入List中，如果属性是pojo对象继续调用parse处理即可。
- 不过有个疑惑，我原以为maplist中只是map和list组成的结构。其中存储基本数据类型，但貌似并不是，不太明白为什么这么设计，为什么不全部转为map或者list结构的数据
    
    - map2Json方法中，并没有对值类型进行判断，然后进行递归处理。而是直接返回，这样返回的maplist结构中就有pojo对象了。
    
            private Map<String, Object> map2Json(Map map, Map<String, Object> valMap) {
                if (null == map)
                    return null;
                ArrayList<Pair> list = new ArrayList<Pair>(map.size());
                Set<Entry<?, ?>> entrySet = map.entrySet();
                for (Entry entry : entrySet) {
                    String name = null == entry.getKey() ? "null" : entry.getKey().toString();
                    Object value = entry.getValue();
                    list.add(new Pair(name, value));
                }
                return writeItem(list, valMap);
            }
    - 而在pojo2Json中，只对属性依然是pojo的情况进行parse处理。而如果属性类型是集合的情况并没有处理，这里就不粘帖代码了，自行查看pojo2Json方法。
    
    
## 题外话
 
 话说nutz的文档和注释，以及测试用例太完美了，这些配合起来读源码非常顺畅。

