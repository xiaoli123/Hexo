---
title: SpringBoot之druid
date: 2016-10-20
tags:
    - springboot
categories: springboot
---
>  本文转载自开源中国 [王念博客-SpringBoot之druid][1]，请大家尽量阅读作者原文，本人只是为阅读方便。

## druid介绍 ##
http://www.oschina.net/p/druid

## 集成教程 ##
1. 导入依赖包

        <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <scope>runtime</scope>
        </dependency>
        <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>druid</artifactId>
           <version>1.0.18</version>
        </dependency>

2. 配置数据源
```
        # 数据库访问配置
        # 主数据源，默认的
        spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
        spring.datasource.driver-class-name=com.mysql.jdbc.Driver
        spring.datasource.url=jdbc:mysql://
        spring.datasource.username=
        spring.datasource.password=
        
        # 下面为连接池的补充设置，应用到上面所有数据源中
        # 初始化大小，最小，最大
        spring.datasource.initialSize=5
        spring.datasource.minIdle=5
        spring.datasource.maxActive=20
        
        # 配置获取连接等待超时的时间
        spring.datasource.maxWait=60000
        
        # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
        spring.datasource.timeBetweenEvictionRunsMillis=60000
        
        # 配置一个连接在池中最小生存的时间，单位是毫秒
        spring.datasource.validationQuery=SELECT 1 FROM DUAL
        spring.datasource.testWhileIdle=true
        spring.datasource.testOnBorrow=false
        spring.datasource.testOnReturn=false
        
        # 打开PSCache，并且指定每个连接上PSCache的大小
        spring.datasource.poolPreparedStatements=true
        spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
        
        # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
        spring.datasource.filters=stat,wall,log4j
        
        # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
        spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
        
        # 合并多个DruidDataSource的监控数据
        #spring.datasource.useGlobalDataSourceStat=true
```

3. 注册servlet和过滤器监控
```
        package com.wangnian;
        import com.alibaba.druid.support.http.StatViewServlet;
        import com.alibaba.druid.support.http.WebStatFilter;
        import org.springframework.boot.context.embedded.FilterRegistrationBean;
        import org.springframework.boot.context.embedded.ServletRegistrationBean;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        /**
         * Created by 王念 on 2016/4/11.
         */
        @Configuration
        public class DruidConfig {
            @Bean
            public ServletRegistrationBean druidServlet() {
                ServletRegistrationBean reg = new ServletRegistrationBean();
                reg.setServlet(new StatViewServlet());
                reg.addUrlMappings("/druid/*");
                //reg.addInitParameter("allow", "127.0.0.1");
                //reg.addInitParameter("deny","");
                reg.addInitParameter("loginUsername", "admin");
                reg.addInitParameter("loginPassword", "admin");
                return reg;
            }

            @Bean
            public FilterRegistrationBean filterRegistrationBean() {
                FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
                filterRegistrationBean.setFilter(new WebStatFilter());
                filterRegistrationBean.addUrlPatterns("/*");
                filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
                return filterRegistrationBean;
            }
        }
```
	启动成功,访问：http://localhost:8080/druid/login.html即可！

4. 其他方式配置
http://blog.csdn.net/catoop/article/details/50925337
注意：使用注解servlet方式需要在主方法上加入  @ServletComponentScan 扫描

5. 配置数据源
```
        @Bean
        public DataSource druidDataSource(
                         @Value("${spring.datasource.driverClassName}") String driver,
                         @Value("${spring.datasource.url}") String url,
                         @Value("${spring.datasource.username}") String username,
                         @Value("${spring.datasource.password}") String password) {
                DruidDataSource druidDataSource = new DruidDataSource();
                druidDataSource.setDriverClassName(driver);
                druidDataSource.setUrl(url);
                druidDataSource.setUsername(username);
                druidDataSource.setPassword(password);
                try {
                    druidDataSource.setFilters("stat, wall");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
                return druidDataSource;
        }
```



  [1]: https://my.oschina.net/wangnian/blog/657207