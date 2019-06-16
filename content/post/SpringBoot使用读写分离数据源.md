---
title: "SpringBoot使用读写分离数据源"
date: 2019-06-16T21:44:38+08:00
lastmod: 2019-06-16T21:44:38+08:00
draft: false
keywords: ["springboot","读写分离"]
description: "SpringBoot使用读写分离数据源"
tags: ["springboot","java"]
categories: ["java","springboot"]
# author: ""
comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---

## SpringBoot使用读写分离数据源

本文来源：https://springboot.io/t/topic/49

> 网站应用的瓶颈大多在DB，DB的性能直接的决定了网站的负载能力。一般的b/s应用。读请求数量大于写请求，于是可以通过读写分离来把读请求负载到多个数据源中（一主N从）。在并发过大的时候，读写分离就能为应用带来一定的性能提升。
> 读写分离不是特别复杂的技术，可以通过中间件或者自己编码实现。可幸`Spring`框架已经提供了动态数据源的这种机制
<!--more-->
### AbstractRoutingDataSource

- 可以路由的数据源，这是一个抽象类。本身已经实现了java的数据源接口`javax.sql.DataSource`，使用它来替换传统单一的数据源。从而达到动态数据源的效果
- 该类内部维护了一个Map，该map存储着系统的所有数据源

```java
@Nullable
private Map<Object, DataSource> resolvedDataSources;
```

- 唯一的抽象方法

```java
/**
 * Determine the current lookup key. This will typically be
 * implemented to check a thread-bound transaction context.
 * <p>Allows for arbitrary keys. The returned key needs
 * to match the stored lookup key type, as resolved by the
 * {@link #resolveSpecifiedLookupKey} method.
*/
@Nullable
protected abstract Object determineCurrentLookupKey();
```

#### 总结

- 其实最为关键的两个点已经说清楚了，所有的数据源存放在`resolvedDataSources`中，里面包含了一个主数据源，以及N多个从数据源
- 在系统需要操作数据源的时候，先调用`determineCurrentLookupKey()`来获取数据源的key，从而确定要使用`resolvedDataSources`中的哪个数据源。

#### 思路

- 既然是读写分离，我们该怎么确定当前请求是读还是写？可以考虑通过`@Transactional`注解来判断，如果`readOnly=true`，则是读请求，反之这是写请求。当然也可以通过自己定义注解来实现
- 确定了怎么区分读写请求后，剩下的事情应该确定怎么从N个从库中选择一个出来处理读请求。很简单 - 轮询。这是最直接简单的办法。
- 通过程序日志可以看到，在执行`@Transactional`方法之前，会先从数据源获取jdbc的连接，在系统获取jdbc连接之前，会调用`determineCurrentLookupKey()`来获取数据源key，根据该key获得数据源，再从该数据源获取到连接。
- 所以我们要做的就是，覆写`determineCurrentLookupKey()`方法，在系统获取jdbc连接之前（执行`@Transactional`），确定本次请求使用的Datasource

### DynamicDataSourceHolder

- 该类维护了一个线程安全的map，`ThreadLocal`，用来标识当前请求是读还是写。从而确定是使用主库还是从库

```java
public class DynamicDataSourceHolder {
	
	//定义主库的key
    public static final String MASTER = "master";
	
    private static final String SLAVE = "slave";

    private static final ThreadLocal<String> HOLDER = new ThreadLocal<>();

    private static void putDataSourceKey(String key) {
        HOLDER.set(key);
    }

    public static String getDataSourceKey() {
        return HOLDER.get();
    }
    
	//标记为主库
    public static void markMaster() {
        putDataSourceKey(MASTER);
    }
	//标签为从库
    public static void markSlave() {
        putDataSourceKey(SLAVE);
    }

	//判断是否是主库
    public static boolean isMaster() {
        return MASTER.equals(HOLDER.get());
    }
}
```

### DynamicDataSource

- `AbstractRoutingDataSource`的实现

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import org.springframework.util.ReflectionUtils;

import javax.sql.DataSource;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.atomic.AtomicInteger;


public class DynamicDataSource extends AbstractRoutingDataSource {

    private static final Logger LOGGER = LoggerFactory.getLogger(DynamicDataSource.class);

	//int的原子操纵类，在多线程环境下可以安全的自增，初始值设置为 -1
    private AtomicInteger counter = new AtomicInteger(-1);

	//存放从库的key
    private List<Object> slaveDataSources = new ArrayList<>(0);

    @Override
    protected Object determineCurrentLookupKey() {
		
        Object key = null;
        if (DynamicDataSourceHolder.isMaster() || this.slaveDataSources.isEmpty()) {
			//如果当前请求是写请求，并且没有任何的从库，返回主库的key
            key = DynamicDataSourceHolder.MASTER;
        } else {
			//否则轮询从库的key
            key = this.getSlaveKey();
        }
        LOGGER.debug("动态数据源 dataSourceKey = {}", key);
        return key;
    }
	//AbstractRoutingDataSource 实现了 InitializingBean 接口，所以在属性注入完毕后会执行该方法
	//通过覆写该方法获，从 resolvedDataSources 中获取到所有的从库
    @SuppressWarnings("unchecked")
    @Override
    public void afterPropertiesSet() {
        super.afterPropertiesSet();
        Field field = ReflectionUtils.findField(AbstractRoutingDataSource.class, "resolvedDataSources");
		//resolvedDataSources 是 private 属性，设置暴力访问
        field.setAccessible(true); 
        try {
            Map<Object, DataSource> resolvedDataSources = (Map<Object, DataSource>) field.get(this);
            for (Map.Entry<Object, DataSource> entry : resolvedDataSources.entrySet()) {
                if (DynamicDataSourceHolder.MASTER.equals(entry.getKey())) {
					//如果数据源的key与master库的key相同，跳过
                    continue;
                }
				//添加 从库的key 到集合
                slaveDataSources.add(entry.getKey());
            }
        } catch (Exception e) {
            LOGGER.error("afterPropertiesSet error! ", e);
        }
    }
	
	//轮询从库key
    public Object getSlaveKey() {
        Integer index = counter.incrementAndGet() % this.slaveDataSources.size();
        if (counter.get() > 9999) {
            counter.set(-1);
        }
        return slaveDataSources.get(index);
    }
}
```

### DynamicDataSourceConfig

- 配置类，为了方便通过配置文件来定义数据源。
- 这里使用的阿里巴巴的`Druid`数据源
- 不是很明白`@ConfigurationProperties`干啥的，先百度

```java
import java.util.HashMap;
import java.util.Map;

import org.springframework.boot.context.properties.ConfigurationProperties;

import com.alibaba.druid.pool.DruidDataSource;


@ConfigurationProperties(prefix = "daynamic")
public class DynamicDataSourceConfig {
	
	private Map<String,DruidDataSource> datasources = new HashMap<>();
	
	public Map<String,DruidDataSource> getDatasources() {
		return datasources;
	}

	public void setDatasources(Map<String,DruidDataSource> datasources) {
		this.datasources = datasources;
	}
}
```

- yml的配置

```
daynamic:
  datasources:
    # master库,唯一只能有一个且名称不可修改（DynamicDataSourceHolder中的MASTER的定义相同）
    master:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/springcloud?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&useSSL=false
      username: root
      password: root

    # slave库,名称唯一可以有多个
    slave_1:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:33006/springcloud?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&useSSL=false
      username: root
      password: root
    
    slave_2:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/springcloud?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&useSSL=false
      username: root
      password: root
```

### DynamicDataSourceConfiguration

- 通过`@Configuration`把数据源配置到ioc

```java
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.alibaba.druid.pool.DruidDataSource;

import io.springcloud.datasource.DynamicDataSource;
import io.springcloud.datasource.DynamicDataSourceConfig;
import io.springcloud.datasource.DynamicDataSourceHolder;

/**
 * 
 * 读写分离数据源
 * @author KevinBlandy
 *
 */

@Configuration
@EnableConfigurationProperties(DynamicDataSourceConfig.class)		//载入yml配置到配置类
public class DynamicDataSourceConfiguration {
	
	private static final Logger LOGGER = LoggerFactory.getLogger(DynamicDataSourceConfiguration.class);
	
	@Autowired
	private DynamicDataSourceConfig dynamicDataSourceConfig;
	
	@Bean
	public DynamicDataSource dynamicDataSource() throws SQLException {
		
		//定义数据源
		DynamicDataSource dynamicDataSource = new DynamicDataSource();
		
		//定义数据源map
		Map<Object, Object> dataSources = new HashMap<>();
		
		for(Map.Entry<String, DruidDataSource> entry : dynamicDataSourceConfig.getDatasources().entrySet()) {
			
			String key = entry.getKey();
			
			DruidDataSource dataSource = entry.getValue();
			
			if(!key.equals(DynamicDataSourceHolder.MASTER)) {
				//如果是从库，则添加到map
				dataSources.put(key, dataSource);
			}
			
			//初始化数据源
			dataSource.init();
			
			LOGGER.info("数据源初始化:key={}",key);
		}
		
		//尝试从配置中获取到主库
		DruidDataSource masterDataSource = dynamicDataSourceConfig.getDatasources().get(DynamicDataSourceHolder.MASTER);
		
		if(masterDataSource == null) {
			throw new IllegalArgumentException("未定义,master库(" + DynamicDataSourceHolder.MASTER + ")");
		}
		
		//添加主库到map
		dataSources.put(DynamicDataSourceHolder.MASTER, masterDataSource);
		
		//设置动态数据源默认使用的库为主库
		dynamicDataSource.setDefaultTargetDataSource(masterDataSource);
		//设置数据源map
		dynamicDataSource.setTargetDataSources(dataSources);
		
		return dynamicDataSource;
	}
}
```

### DaynamicDatasouceAop

- 最后我们通过 `AOP`在业务层方法执行之前，确定当前请求是读还是写

```java
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import io.springcloud.datasource.DynamicDataSourceHolder;

@Order(-9999)		//它必须优先于业务层的所有切面方法之前执行
@Aspect
@Component
public class DaynamicDatasouceAop {
	
	private static final Logger LOGGER = LoggerFactory.getLogger(DaynamicDatasouceAop.class);
	
	//切面定义，业务层的所有类的所有方法
	@Pointcut(value = "execution(* io.springcloud.service.*.*(..))")
	public void service() {
	}

	@Before("service()")
	public void beforService(JoinPoint joinPoint) throws NoSuchMethodException, SecurityException {
		
		Object target = joinPoint.getTarget();
		
		String methodName = joinPoint.getSignature().getName();
		
		Object[] args = joinPoint.getArgs();
		
		Class<?>[] parameterTypes = ((MethodSignature) joinPoint.getSignature()).getMethod().getParameterTypes();
		
		//目标方法
		Method method = null;
		
		method = target.getClass().getMethod(methodName, parameterTypes);
		
		if (method.isBridge()) {
			for (int i = 0; i < args.length; i++) {
				Class<?> genClazz = getSuperClassGenricType(target.getClass(), 0);
				if (args[i].getClass().isAssignableFrom(genClazz)) {
					parameterTypes[i] = genClazz;
				}
			}
			method = target.getClass().getMethod(methodName, parameterTypes);
		}
		
		LOGGER.debug("当前事务方法  " + methodName);
		
		Transactional transactional = method.getAnnotation(Transactional.class);
		//未标识@Transactional注解，或者事务非只读，则标识主库
		if(transactional != null && transactional.readOnly()){
			LOGGER.debug("动态数据源 - 读库");
			DynamicDataSourceHolder.markSlave();
		}else {
			LOGGER.debug("动态数据源 - 写库");
			DynamicDataSourceHolder.markMaster();
		}
	}

	public Class<?> getSuperClassGenricType(Class<?> clazz, int index) {
		Type genType = clazz.getGenericSuperclass();
		if (!(genType instanceof ParameterizedType)) {
			return Object.class;
		}
		Type[] params = ((ParameterizedType) genType).getActualTypeArguments();
		if (!(params[index] instanceof Class)) {
			return Object.class;
		}
		return (Class<?>) params[index];
	}
}
```

### 搞定