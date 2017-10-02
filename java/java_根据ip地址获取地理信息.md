# 利用com.maxmind.db根据ip地址获取地理位置信息

## 1 添加Maven依赖

```
<dependency>
    <groupId>com.maxmind.db</groupId>
    <artifactId>maxmind-db</artifactId>
    <version>1.0.0</version>
</dependency>
```

## 2 用法

### 2.1 简单示例

```java
File database = new File("/path/to/database/GeoIP2-City.mmdb");
Reader reader = new Reader(database);

InetAddress address = InetAddress.getByName("24.24.24.24");

JsonNode response = reader.get(address);

System.out.println(response);

reader.close();
```


## 案例(获取国家和省份信息工具类)

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.maxmind.db.Reader;

import java.io.IOException;
import java.io.InputStream;
import java.net.InetAddress;
import java.util.HashMap;
import java.util.Map;

public class GeoLiteUtil {
    private static Reader reader = null;

    private static Map<String, String> cache = new HashMap<String, String>();

    static {
        try {
            ClassLoader loader = Thread.currentThread().getContextClassLoader();
            InputStream in = loader.getResource("GeoLite2-City.mmdb").openStream();
            reader = new Reader(in);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    /**
     * 获取条目
     *
     * @param ip
     * @return
     */
    public static String getEntry(String ip) {
        String entry = cache.get(ip);
        try {
            if (entry == null) {
                JsonNode node = reader.get(InetAddress.getByName(ip));
                if (node == null){
                    return "未知国家,未知省份";
                }
                String country = node.get("country").get("names").get("zh-CN").textValue();
                String province = node.get("subdivisions").get(0).get("names").get("zh-CN").textValue();
                entry = country + "," + province;
                cache.put(ip, entry);
            }

        } catch (IOException e) {
            System.out.println(ip + " = null");
        }
        return entry;
    }

    public static String getCountry(String ip) {
        String entry = getEntry(ip);
        if (entry != null) {
            return entry.split(",")[0];
        }
        return null;
    }

    public static String getProvince(String ip) {
        String entry = getEntry(ip);
        if (entry != null) {
            return entry.split(",")[1];
        }
        return null;
    }
}
```
