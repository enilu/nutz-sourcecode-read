---
layout: post
category : 学习笔记 
keywords: "nutz,源码学习,json"
description : "上篇主要阅读了Json类的方法，针对java对象和字符串之间的转换。 这篇主要针对json的一些工具类行的功能进行阅读说明"
tags : [nutz,源码学习]
---

上篇主要阅读了Json类的方法，针对java对象和字符串之间的转换。 这篇主要针对json的一些工具类行的功能进行阅读说明，
包括：

- json的注解
- json的格式化
- json异常处理

以上。

<!--break-->

{% include JB/setup %}

### json的注解

json模块共提供了三个json相关注解，饭别是JsonField,JsonIgnore,ToJson他们的作用分别为：

- JsonField,定义pojo对象中要被转换为json字符串的属性的相关定义，比如可以定义默认值，输出格式。
- JsonIgnore, 针对一些特殊的值，可以通过这个注解来忽略输出。
- ToJson,作用与类的注解，如果添加改注解,可以根据注解的定义调用用户自定义的json输出方法，如果只是使用改注解但是没有设置value值，则默认调用pojo对象的toJson方法，如果没有改方法，则相当与该注解无效，将调用系统默认方法来输出json。

## change from 2015-01-02


### JsonFormat json格式化处理

- JsonFormat提供了json输出的格式设置，默认提供了5中输出格式：compact，full,nice,forLook,tidy;这几中格式的意思官方文档说明的很明白了，自行查看吧。
默认不设置格式的时候用的是compact格式。
- JsonFormat基本上是一个POJO类，包含了一些属性，以及属性的set和get方法。所以这个类相对比较容易理解了。
格式化设置的范围主要包括：日期格式，数字格式，缩进列数，是不是换行，是不是忽略空值;也可以设置只输出那些属性，或者部署出哪些属性
- JsonRenderImpl 在输出json的时候会使用JsonFormat对象进行格式控制，比如：
    - 在输出日期类型数据的时候调用日期格式来格式化日期：
    
        if (obj instanceof Date) {
            DateFormat df = format.getDateFormat();
            if (df != null) {
                string2Json(df.format((Date)obj));
                flag = false;
            }
        }
    - 在输出每个属性的时候输出JsonFormat中定义好的分割符：
              
        writer.append(format.getSeparator());
        
    - 在对象为空的时候判断是否需要输出：
    
        private boolean isIgnore(String name, Object value) {
            if (null == value && format.isIgnoreNull())
                return true;
            return format.ignore(name);
        }
        
### JsonException

JsonException 有4个构造函数，其中后面两个，详细定义了在读取json转换为java对象的时候，调用改Exception输出json的第几行，第几列不规范不是标准的json
