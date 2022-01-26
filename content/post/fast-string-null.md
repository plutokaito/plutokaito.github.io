---
title: "FastJson 调用 toString 时 null 字段不显示"
date: 2021-01-26
tags: ["源码分析", "fastjson"]
description : "FastJson 调用 toString 时 null 字段不显示"
---
## 前提
1. 创建 maven 项目
2. 引入 fastjson , [github 地址](https://github.com/alibaba/fastjson)
```xml
<dependency>
 <groupId>com.alibaba</groupId>
 <artifactId>fastjson</artifactId>
 <version>1.2.79</version>
</dependency>
```

## 现象
```java
public class FastJsonDemo {
  public static void main(String[] args) {
   BizParam param = BizParam.builder().name("xxx").build();
   String json = JSON.toJSONString(param, true);
   System.out.println("json:" + json);
  }
}

====
json:{
	"name":"xxx"
}

```

上述显示会把 null 的字段干掉

## 源码跟踪

com.alibaba.fastjson.JSON 中最终调用了 `toJSONString(java.lang.Object, int, com.alibaba.fastjson.serializer.SerializerFeature...)` ,  查看实现
```java
    public static String toJSONString(Object object, int defaultFeatures, SerializerFeature... features) {
        SerializeWriter out = new SerializeWriter((Writer) null, defaultFeatures, features);

        try {
            JSONSerializer serializer = new JSONSerializer(out);
            serializer.write(object);
            String outString = out.toString();
            int len = outString.length();
            if (len > 0
                    && outString.charAt(len -1) == '.'
                    && object instanceof Number
                    && !out.isEnabled(SerializerFeature.WriteClassName)) {
                return outString.substring(0, len - 1);
            }
            return outString;
        } finally {
            out.close();
        }
    }

```

其中调用了 `com.alibaba.fastjson.serializer.JSONSerializer#write(java.lang.Object)` 方法，该方法中需要的注意是 `getObjectWriter` 方法，这个方法会调用
`com.alibaba.fastjson.serializer.SerializeConfig#getObjectWriter(java.lang.Class<?>, boolean)` ，该方法会判断一系列该类的解析类型, 如果第二个参数为 true 则创建 `JavaBeanSerializer` 的解析器
```java
...
if (create) {
     writer = createJavaBeanSerializer(clazz);
     put(clazz, writer);
}
...
```
最终调用了 `com.alibaba.fastjson.serializer.SerializeConfig#createJavaBeanSerializer(com.alibaba.fastjson.serializer.SerializeBeanInfo)`, 通过一系列的 asm 复制判断，最终如果 `asm == true` 的话，那么调用 createASMSerializer
```java
if (asm) {
    try {
        ObjectSerializer asmSerializer = createASMSerializer(beanInfo);
        if (asmSerializer != null) {
            return asmSerializer;
        }
    } ...
  ...
}
```
而实际上调用 `com.alibaba.fastjson.serializer.ASMSerializerFactory#createJavaBeanSerializer` ，创建了一个 `JavaBeanSerializer` .

因此在方法 `com.alibaba.fastjson.serializer.JSONSerializer#write(java.lang.Object)` 调用 `com.alibaba.fastjson.serializer.ObjectSerializer#write` 方法的时候，实际调用了 `com.alibaba.fastjson.serializer.JavaBeanSerializer#write(com.alibaba.fastjson.serializer.JSONSerializer, java.lang.Object, java.lang.Object, java.lang.reflect.Type, int, boolean)` ，查看此方法源码发现
```java
...
if (propertyValue == null) {
    int serialzeFeatures = fieldInfo.serialzeFeatures;
    JSONField jsonField = fieldInfo.getAnnotation();
    if (beanInfo.jsonType != null) {
        serialzeFeatures |= SerializerFeature.of(beanInfo.jsonType.serialzeFeatures());
    }
    // beanInfo.jsonType
    if (jsonField != null && !"".equals(jsonField.defaultValue())) {
       propertyValue = jsonField.defaultValue();
    } else if (fieldClass == Boolean.class) {
       ...
    } else if (fieldClass == String.class) {

        int defaultMask = SerializerFeature.WriteNullStringAsEmpty.mask;
        final int mask = defaultMask | SerializerFeature.WriteMapNullValue.mask;
        if ((!writeAsArray) && (serialzeFeatures & mask) == 0 && (out.features & mask) == 0) {
           // 进入这里之后跳出循环，不做任何解析
           continue;
        } else if ((serialzeFeatures & defaultMask) != 0) {
            propertyValue = false;
        } else if ((out.features & defaultMask) != 0
                   && (serialzeFeatures & SerializerFeature.WriteMapNullValue.mask) == 0) {
            propertyValue = false;
        }
    }
}
...
```

这里`((!writeAsArray) && (serialzeFeatures & mask) == 0 && (out.features & mask) == 0)` 为什么判断为真呢？
首先 第一个判断 `!writeAsArray` 为真，它的来源为该方法 (`JavaBeanSerializer#write`) 中的`isWriteAsArray(com.alibaba.fastjson.serializer.JSONSerializer, int)` 方法，而 beanInfo 在创建时 features 为 0，那么返回就是 false， 那么 `!writeAsArray` 便为真.

第二个是 serialzeFeatures 是字段的 serialzeFeatures 也为 0 ，那么 == 0 成立.

第三个 out.features & mask， 由于外部传入为  0。
至此水落石出。


如果要显示 null,使用配置 `SerializerFeature.WriteMapNullValue` 即可, 如下：
```java
public class FastJsonDemo {
    public static void main(String[] args) {
        BizParam param = BizParam.builder().name("xxx").build();
        String json = JSON.toJSONString(param, true);
        System.out.println("json:" + json);
        String json1 = JSON.toJSONString(param, SerializerFeature.WriteMapNullValue);
        System.out.println("json1:" + json1);
    }
}

-------
json:{
	"name":"xxx"
}
json1:{"name":"xxx","userId":null}

```

经过调试后调用 `com.alibaba.fastjson.serializer.ObjectSerializer#write` 时并不会进入 JavaBeanSerializer ，猜想的原因是由于 ASM 字节码修改执行文件有关。这块待深入理解。



其他配置参见 `com.alibaba.fastjson.serializer.SerializerFeature`

### 推荐阅读：
- [在 Spring 中集成 Fastjson](https://github.com/alibaba/fastjson/wiki/%E5%9C%A8-Spring-%E4%B8%AD%E9%9B%86%E6%88%90-Fastjson)
- [Integrate Fastjson in JAXRS](https://github.com/alibaba/fastjson/wiki/Integrate-Fastjson-in-JAXRS)
- [json](https://www.json.org/json-zh.html)