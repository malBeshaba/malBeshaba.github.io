---
title: 注解简化代码？Lombok！
date: 2020-05-25 13:42:56
tags: Java
---

## Lombok简介

> Lombok是一个可以通过简单的注解形式来帮助我们简化消除一些必须有但显得很臃肿的Java代码的工具，通过使用对应的注解，可以在编译源码的时候生成对应的方法。
>
> [官方传送门](https://projectlombok.org/)
>
> [github传送门](https://github.com/rzwitserloot/lombok)

## 安装方法

引入lombok的jar包就好

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.14</version>
</dependency>
```

<!-- more -->

IntelliJ IDEA需要安装插件哦**

![image.png](https://ae05.alicdn.com/kf/H0dd4ec2af44442fb8033b096f9f956d0i.png)



##lombok使用

lombok使用过程中主要是靠注解起作用的，官网上的文档里面有所有的注解，这里不一一罗列，只说明其中几个比较常用的。

#### `@NonNull`: 可以帮助我们避免空指针。

使用lombok：

```java
import lombok.NonNull;
    public class NonNullExample extends Something {
        private String name;  
        public NonNullExample(@NonNull Person person) {
        super("Hello");
        this.name = person.getName();
    }
}
```

不使用lombok：

```java
public class NonNullExample extends Something {
    private String name;  
    public NonNullExample(@NonNull Person person) {
        super("Hello");
        if (person == null) {
            throw new NullPointerException("person");
        }
        this.name = person.getName();
    }
}
```

#### `@Cleanup`: 自动帮我们调用`close()`方法。

使用lombok：

```java
import lombok.Cleanup;
import java.io.*;
public class CleanupExample {
    public static void main(String[] args) throws IOException {
        @Cleanup InputStream in = new FileInputStream(args[0]);
        @Cleanup OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
            int r = in.read(b);
            if (r == -1) break;
            out.write(b, 0, r);
        }
    }
}
```

不使用lombok：

```java
import java.io.*;
    public class CleanupExample {
        public static void main(String[] args) throws IOException {
            InputStream in = new FileInputStream(args[0]);
            try {
                OutputStream out = new FileOutputStream(args[1]);
                try {
                    byte[] b = new byte[10000];
                    while (true) {
                    int r = in.read(b);
                    if (r == -1) break;
                    out.write(b, 0, r);
                    }
                } finally {
                    if (out != null) {
                        out.close();
                    }
                }
            } finally {
                if (in != null) {
                in.close();
            }
        }
    }
}
```

#### `@Getter / @Setter`: 自动生成Getter/Setter方法

使用lombok：

```java
    import lombok.AccessLevel;
    import lombok.Getter;
    import lombok.Setter;
    public class GetterSetterExample {
        @Getter @Setter private int age = 10;
        @Setter(AccessLevel.PROTECTED) private String name;
    }
```

不使用lombok：

```java
public class GetterSetterExample {
    private int age = 10;
    private String name;
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    protected void setName(String name) {
        this.name = name;
    }
}
```

####`@ToString`

> 生成toString()方法，默认情况下，它会按顺序（以逗号分隔）打印你的类名称以及每个字段。可以这样设置不包含哪些字段***@ToString(exclude = "id")\*** / ***@ToString(exclude = {"id","name"})\***
>  如果继承的有父类的话，可以设置**callSuper** 让其调用父类的toString()方法，例如：***@ToString(callSuper = true)\***

```kotlin
import lombok.ToString;
@ToString(exclude = {"id","name"})
public class User {
  private Integer id;
  private String name;
  private String phone;
}
```

生成toString方法如下：

```cpp
public String toString(){
  return "User(phone=" + phone + ")";
}
```



#### @NoArgsConstructor`: 自动生成无参数构造函数。

#### `@AllArgsConstructor`: 自动生成全参数构造函数。

#### `@Data`: 自动为所有字段添加@ToString, @EqualsAndHashCode, @Getter方法，为非final字段添加@Setter,和@RequiredArgsConstructor!



**[官方文档https://projectlombok.org/features/index.html](https://projectlombok.org/features/index.html)**

