Spring配置文件

数据源一：MySQL

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"  
  destroy-method="close">  
 <property name="driverClassName" value="${driver}" />  
 <property name="url" value="${url}" />  
 <property name="username" value="${username}" />  
 <property name="password" value="${password}" />  
  <!-- 初始化连接大小 -->  
  <property name="initialSize" value="${initialSize}"></property>  
  <!-- 连接池最大数量 -->  
  <property name="maxActive" value="${maxActive}"></property>  
  <!-- 连接池最大空闲 -->  
  <property name="maxIdle" value="${maxIdle}"></property>  
  <!-- 连接池最小空闲 -->  
  <property name="minIdle" value="${minIdle}"></property>  
  <!-- 获取连接最大等待时间 -->  
  <property name="maxWait" value="${maxWait}"></property>  
</bean>

数据源二：SQLServer

<bean id="dataSource2" class="org.apache.commons.dbcp.BasicDataSource"  
  destroy-method="close">  
 <property name="driverClassName" value="${driver_sqlserver}" />  
 <property name="url" value="${url_sqlserver}" />  
 <property name="username" value="${username_sqlserver}" />  
 <property name="password" value="${password_sqlserver}" />  
  <!-- 初始化连接大小 -->  
  <property name="initialSize" value="${initialSize}"></property>  
  <!-- 连接池最大数量 -->  
  <property name="maxActive" value="${maxActive}"></property>  
  <!-- 连接池最大空闲 -->  
  <property name="maxIdle" value="${maxIdle}"></property>  
  <!-- 连接池最小空闲 -->  
  <property name="minIdle" value="${minIdle}"></property>  
  <!-- 获取连接最大等待时间 -->  
  <property name="maxWait" value="${maxWait}"></property>  
</bean>

数据源三：Oracle

<bean id="dataSource3" class="org.apache.commons.dbcp.BasicDataSource"  
  destroy-method="close">  
 <property name="driverClassName" value="${driver2}" />  
 <property name="url" value="${url2}" />  
 <property name="username" value="${username2}" />  
 <property name="password" value="${password2}" />  
</bean>

配置动态数据源

<!-- 动态DataSource配置    -->

<bean id="dynamicDataSource" class="com.pskj.GSLZ.utils.dataSource.DynamicDataSource">  
  <!--默认数据源  -->  
  <property name="defaultTargetDataSource" ref="dataSource"/>  
  <property name="targetDataSources">  
     <map key-type="java.lang.String">  
         <entry key="dataSource" value-ref="dataSource"/>  
         <entry key="dataSource2" value-ref="dataSource2"/>  
         <entry key="dataSource3" value-ref="dataSource3"/>  
    </map>
  </property>
</bean>



Spring和MyBatis整合

<!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
  <!-- 数据库连接池 -->  
 <!--<property name="dataSource" ref="dataSource" />-->  <property name="dataSource" ref="dynamicDataSource" />  
  <!-- 自动扫描mapping.xml文件 -->  
  <property name="mapperLocations" value="classpath:mapper/*/*.xml"></property>  
  <!--加载mybatis的全局配置文件-->  
  <property name="configLocation" value="classpath:mybatis-config.xml"></property>  
</bean>



多数据源配置

DataSource：



package com.pskj.GSLZ.utils.dataSource;  

import java.lang.annotation.*;  

//设置注解  

@Retention(RetentionPolicy.RUNTIME)  

@Target({ElementType.METHOD,ElementType.TYPE})  

@Documented  

public @interface DataSource {  

     String value() default "";  

}



DataSwitchAOP：



package com.pskj.GSLZ.utils.dataSource;  

import java.lang.reflect.Method;  

import org.aspectj.lang.JoinPoint;  
import org.aspectj.lang.reflect.MethodSignature;  
public class DataSwitchAop {

    /**

          拦截目标方法，获取由@DataSource指定的数据源标识，设置到线程存储中以便切换数据源 

    */

    public void intercept(JoinPoint point) throws Exception {  

       Class<?> target = point.getTarget().getClass();  

        MethodSignature signature = (MethodSignature) point.getSignature();  

        // 默认使用目标类型的注解，如果没有则使用其实现接口的注解  

        for (Class<?> clazz : target.getInterfaces()) {  

          resolveDataSource(clazz, signature.getMethod());  

        }  

       resolveDataSource(target, signature.getMethod());  

}  

/**  

        提取目标对象方法注解和类型注解中的数据源标识 * * @param clazz  

*/  

private void resolveDataSource(Class<?> clazz, Method method) {  

   try {  

       Class<?>[] types = method.getParameterTypes();  

// 默认使用类型注解  

if (clazz.isAnnotationPresent(DataSource.class)) {  

           DataSource source = clazz.getAnnotation(DataSource.class);  

DynamicDataSourceHolder.setDataSource(source.value());  

}  

       // 方法注解可以覆盖类型注解  

Method m = clazz.getMethod(method.getName(), types);  

if (m != null && m.isAnnotationPresent(DataSource.class)) {  

           DataSource source = m.getAnnotation(DataSource.class);  

DynamicDataSourceHolder.setDataSource(source.value());  

}  

   } catch (Exception e) {  

       System.out.println(clazz + ":" + e.getMessage());  

}  

}  

}



DynamicDataSource：



package com.pskj.GSLZ.utils.dataSource;  

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;  

public class DynamicDataSource extends AbstractRoutingDataSource {  

    @Override  

    protected Object determineCurrentLookupKey() {  

     return DynamicDataSourceHolder.getDataSource();  

    }  

}



DynamicDataSourceHolder：



public class DynamicDataSourceHolder{  

        private static final ThreadLocal<String> dataSourceKey = new InheritableThreadLocal<String>();  

 public static String getDataSource() {  
            return dataSourceKey.get();  
  }  

        public static void setDataSource(String dataSource) {  
            dataSourceKey.set(dataSource);  

  }  

        public static void clearDataSource() {  
            dataSourceKey.remove();  

  }  

}
