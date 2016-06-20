---
layout: post
category : 学习笔记 
keywords: "nutz,源码学习,json"
description : "json模块的包是：org.nutz.json，下文中涉及到的类名都是从该包路径说起"
tags : [nutz,源码学习]
---

json模块的包是：org.nutz.json，下文中涉及到的类名都是从该包路径说起。

json模块的大致功能包括以下几点：

- Json类将java对象和字符串进行互相转换。
- 定义了json的注解。包括：Jsonield，JsonIgnore，ToJson
- 针对json的输出进行了一些格式的控制，涉及类包括：JsonFormat
- 异常处理：JsonException
- 其他定义的一些接口和具体工具类，这里不一一列觉。
- 该模块的结构图如下所示：

![json模块类结构图]({{ site.img_url }}/2016/nutz/json_diagram.png)

<!--break-->

{% include JB/setup %}

### Json类将java对象和字符串进行互相转换

该类主要通过fromJson和toJson两类方法来完成java对象与字符串之间的相互转换。
主要方法入下图所示：

![json类结构图]({{ site.img_url }}/2016/nutz/json.png)

#### fromJson方法
- fromJson方法进行了多次重载，接收参数可以是字符串，各种输入流，文件.
- 另外也提供了fromJsonAsArray方法用于将json字符串转换为java列表。设计和用法与fromJson没有太大差别。
- 返回结果可以是指定类型的java对象，或者Map对象（Map对象中可能继续嵌套Map或者List对象）
- 具体转换步骤

    - 转换过程中用到一些工具类，比如文件打开和关闭，字符对象转换，这些类都在org.nutz.lang模块中，这些以后读到相应的模块再深入了解。
    - fromJson调用的是impl.JsonCompileImplV2类的parse方法。
    - 而这个parse方法调用了JsonTokenScan的read方法。JsonTokenScan 类是解析json字符串，并返回一个Map-List对象的核心实现;
    - 最后在Json类中调用Mapl  工具类将Map-List对象转换为指定的java对象 
    - JsonTokenScan 中通过顺序读取json字符，根据字符特征来标记一个Map或者List的开始或者结束，将一个个属性存储在Map或者List对象中，并嵌套起来。这里是字符串转换为对象的核心算法实现。点击[查看具体代码](https://github.com/nutzam/nutz/blob/master/src/org/nutz/json/impl/JsonCompileImplV2.java)
## change from 2015-01-02

#### toJson方法

- toJson方法同样进行了重载，提供了将java丢想转换为字符串，输出到指定的输出流，文件等，输出的时候可以指定字符串格式，格式有传入的Jsonromat规定。
- 所有重载的toson方法最终都是调用：toJson(Writer writer, Object obj, JsonFormat format),该方法调用Jsonender接口的render将java对象输出到制定的输出流中，该接口有一个默认实现：JsonRenderImpl，接下来看它的render方法。

    <code>
        public void render(Object obj) throws IOException {
            if (null == obj) {
                writer.write("null");
            } else if (obj instanceof JsonRender) {
                ((JsonRender) obj).render(null);
            } else if (obj instanceof Class) {
                string2Json(((Class<?>) obj).getName());
            } else if (obj instanceof Mirror) {
                string2Json(((Mirror<?>) obj).getType().getName());
            } else {
                Mirror mr = Mirror.me(obj.getClass());
                // 枚举
                if (mr.isEnum()) {
                    string2Json(((Enum) obj).name());
                }
                // 数字，布尔等
                else if (mr.isNumber()) {
                    String tmp = obj.toString();
                    if (tmp.equals("NaN")) {
                        // TODO 怎样才能应用上JsonFormat中是否忽略控制呢?
                        // 因为此时已经写入了key:
                        writer.write("null");
                    }
                    else
                        writer.write(tmp);
                }
                else if (mr.isBoolean()) {
                    writer.append(obj.toString());
                }
                // 字符串
                else if (mr.isStringLike() || mr.isChar()) {
                    string2Json(obj.toString());
                }
                // 日期时间
                else if (mr.isDateTimeLike()) {
                    boolean flag = true;
                    if (obj instanceof Date) {
                        DateFormat df = format.getDateFormat();
                        if (df != null) {
                            string2Json(df.format((Date)obj));
                            flag = false;
                        }
                    }
                    if (flag)
                        string2Json(format.getCastors().castToString(obj));
                }
                // 其他
                else {
                    // Map
                    if (obj instanceof Map) {
                        map2Json((Map) obj);
                    }
                    // 集合
                    else if (obj instanceof Collection) {
                        coll2Json((Collection) obj);
                    }
                    // 数组
                    else if (obj.getClass().isArray()) {
                        array2Json(obj);
                    }
                    // 普通 Java 对象
                    else {
                        memo.add(obj);
                        pojo2Json(obj);
                        memo.remove(obj);
                    }
                }
            }
        }
    </code>
        
- 从上面可以看到该方法，主要针对要转换为json字符串的对象的类型进行判断，然后根据不同类型采用不同的处理方法。
    - 针对各种基本类型的处理方法是将其转换为String类型，然后调用string2Json输出为json字符串。在string2Json方法中，一个一个字符迭代，根据字符标识，进行输出
    
             <code>       
            private void string2Json(String s) throws IOException {
                if (null == s)
                    writer.append("null");
                else {
                    char[] cs = s.toCharArray();
                    writer.append(format.getSeparator());
                    for (char c : cs) {
                        switch (c) {
                        case '"':
                            writer.append("\\\"");
                            break;
                        case '\n':
                            writer.append("\\n");
                            break;
                        case '\t':
                        case 0x0B: // \v
                            writer.append("\\t");
                            break;
                        case '\r':
                            writer.append("\\r");
                            break;
                        case '\f':
                            writer.append("\\f");
                            break;
                        case '\b':
                            writer.append("\\b");
                            break;
                        case '\\':
                            writer.append("\\\\");
                            break;
                        default:
                            if (c >= 256 && format.isAutoUnicode()) {
                                writer.append("\\u");
                                String u = Strings.fillHex(c, 4);
                                if (format.isUnicodeLower())
                                    writer.write(u.toLowerCase());
                                else
                                    writer.write(u.toUpperCase());
                            }
                            else {
                                if (c < ' ' || (c >= '\u0080' && c < '\u00a0')
                                        || (c >= '\u2000' && c < '\u2100')) {
                                    writer.write("\\u");
                                    String hhhh = Integer.toHexString(c);
                                    writer.write("0000", 0, 4 - hhhh.length());
                                    writer.write(hhhh);
                                } else {
                                    writer.append(c);
                                }
                            }
                        }
                    }
                    writer.append(format.getSeparator());
                }
            }
             
         </code>
            
    - 针对map,list,数组调用相应的map2Json,coll2Json,array2Json方法输出。
    - 然后是最重要的，针对普通的pojo对象进行输出,
        - 该方法为：pojo2Json，该方法主要使用反射一一获取pojo对象的属性，放在列表中，然后迭代列表，继续调用render进行处理，这样递归调用render方法，直到所有属性都被转换为基本数据类型，或者集合（集合中是基本数据类型）；最后调用render方法中针对基本数据类型和集合（map，list,array)的处理方法输出json字符串。
        - 这个过程使用了几个工具类:
            - entity.JsonEntity:用于描述java对象映射为json字符串的规则（参考api）比如当前java对象有哪几个属性，分别什么类型等等。
            - org.nutz.lang.Mirror:该类提供了一些反射相关操作方法的封装。