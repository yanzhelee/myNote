## 通过反射完成不同对象之间的属性拷贝

```java
import java.beans.BeanInfo;
import java.beans.Introspector;
import java.beans.PropertyDescriptor;
import java.lang.reflect.Method;
import java.util.List;

/**
 * 数据工具类
 */
public class DataUtil {

	/**
	 * 实现属性拷贝
	 */
	public static void copyProperties(Object src , Object dest){
		try {
			//源对象的bean信息
			BeanInfo bi_src = Introspector.getBeanInfo(src.getClass());
			Class destClass = dest.getClass();

			//取出源对象类的属性描述符
			PropertyDescriptor[] ps = bi_src.getPropertyDescriptors();
			//迭代所有属性
			for(PropertyDescriptor pp : ps){
				//找出get方法和set方法
				Method getter = pp.getReadMethod();
				Method setter = pp.getWriteMethod() ;
				//标准属性
				if(getter != null && setter != null){
					//提取getter返回值类型
					Class retType = getter.getReturnType();
					//获得目标对象对应的setter方法签名
					Method destSetter = null ;
					try {
						destSetter = destClass.getMethod(setter.getName(), retType);

					} catch (Exception e) {
						continue;
					}
					//取出源对象的属性值
					getter.setAccessible(true);
					Object srcValue = getter.invoke(src) ;
					//设置给目标对象
					destSetter.setAccessible(true);
					destSetter.invoke(dest,srcValue) ;
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 将属性复制给集合中的每个对象
	 */
	public static void copyProperites(Object src , List list){
		for(Object dest : list){
			copyProperties(src,dest);
		}
	}

}
```
