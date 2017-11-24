# java操作xml——JDom使用详解

JDom是一个开源项目，它基于树形结构，利用纯JAVA的技术对XML文档实现解析、生成、序列化以及多种操作。

## JDom简介

JDom直接为JAVA变成服务。它利用更为有力的java语言的诸多特性(方法重载、集合概念以及映射)，把SAX和DOM的功能有效地结合起来。在使用设计上尽可能地隐藏原来使用xml过程中的复杂性。利用JDom处理xml文档是一件轻松简单的事。

JDOM 在2000年的春天被Brett McLaughlin和Jason Hunter开发出来，以弥补DOM及SAX在实际应用当中的不足之处。

这些不足之处主要在于SAX没有文档修改、随机访问以及输出的功能，而对于DOM来说，java程序员在使用时不是很方便。

DOM的缺点主要是来自于DOM是一个接口定义语言(IDL)，它的任务实在不同语言实现统一。并不是为java特别设计的。

## JDOM包概览

JDOM是由以下几个包组成

|        包名        |                      解释                       |
| ------------------ | ----------------------------------------------- |
| org.jdom           | 包含了所有的xml文档要素的java类                 |
| org.jdom.adapters  | 包含了与dom适配的java类                         |
| org.jdom.filter    | 包含了xml文档的过滤器类                         |
| org.jdom.input     | 包含了读取xml文档的类                           |
| org.jdom.output    | 包含了写入xml文档的类                           |
| org.jdom.transform | 包含了将jdom xml文档接口转换为其它xml文档的接口 |
| org.jdom.xpath     | 包含了对xml文档xpath操作的类                    |

## 案例之通过jdom生成xml

```java
package xmlTest;

import java.io.FileOutputStream;

import org.jdom.Document;
import org.jdom.Element;
import org.jdom.output.Format;
import org.jdom.output.XMLOutputter;

public class GenerateJdom{
    public static void main(String[] args) throws Exeception{
        Document doc = new Document() ;
        Element root = new Element("root") ;
        doc.addContent(root) ;

        Element name = new Element("name") ;
        root.addContent(name) ;
        root.setAttribute("author","yanzhelee").setAttribute("url","http://www.csdn.com") ;

        name.addContent("yanzhelee");

        XMLOutputter out = new XMLOutputter() ;
        Format format = Format.getPrettyFormat();
        format.setIndent("  ");
        out.setFormat(format);
        out.output(doc, new FileOutputStream("jdom.xml")) ;
    }
}

```

下面是生成的xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root author="yanzhelee" url="http://www.csdn.com">
    <name>yanzhelee</name>
</root>
```

## 通过jdom解析xml文档

```java
package xmlTest;
/**
 * @author CIACs
 */
import java.io.File;
import java.io.FileOutputStream;
import java.util.List;

import org.jdom.Attribute;
import org.jdom.Document;
import org.jdom.Element;
import org.jdom.input.SAXBuilder;
import org.jdom.output.XMLOutputter;

public class ParseJdom{

    public static void main(String[] args) throws Exeception{
        // 通过SAXBuilder解析xml
        SAXBuilder builder = new SAXBuilder();

        Document doc = builder.build(new File("jdom.xml"));
        Element root = doc.getRootElement();

        System.out.println(root.getName());
        String name = root.getChild("name").getText();

        System.out.println("name: "+name);

        List attrs = root.getAttributes();

        for(int i = 0; i < attrs.size();i++)
        {
            String attrName;
            String attrValue;
            Attribute attr = (Attribute)attrs.get(i);
            attrName = attr.getName();
            attrValue = attr.getValue();
            System.out.println(attrName+":"+attrValue);

        }
        //删除属性url，并保存到jdom2.xml
        root.removeAttribute("url");

        XMLOutputter out = new XMLOutputter();
        out.output(doc, new FileOutputStream("jdom2.xml"));
    }

}
```

## maven依赖

```xml
<dependency>
    <groupId>org.jdom</groupId>
    <artifactId>jdom</artifactId>
    <version>1.1</version>
</dependency>
```

## 参考博文

[https://www.cnblogs.com/zhi-hao/p/4016363.html](https://www.cnblogs.com/zhi-hao/p/4016363.html)
