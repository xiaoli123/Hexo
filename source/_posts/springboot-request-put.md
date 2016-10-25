---
title: springboot写rest风格接口request method为put取不到参数
date: 2016-10-21
tags:
    - springboot
categories: springboot
---

今天在写rest的风格的接口，更新的时候愣是取不到参数，代码如下：
```
    @RequestMapping(method = RequestMethod.PUT)
    public Map<String, Object> update(User user) {
        Map<String, Object> result = new LinkedHashMap<String, Object>();
        userServer.updateById(user);
        result.put("message", "更新成功");
        result.put("user", user);
        return result;
    }
```
于是进行一番google和百度，有人说加上HttpPutFormContentFilter就好了。但他是写在web.xml中的，如下：
```
<filter>
    <filter-name>httpPutFormContentFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>httpPutFormContentFilter</filter-name>
    <servlet-name>rest</servlet-name>
</filter-mapping>
```
我进行如下改写：
```
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean httpPutFormContentFilter() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        HttpPutFormContentFilter httpPutFormContentFilter = new HttpPutFormContentFilter();
        registration.setFilter(httpPutFormContentFilter);
        registration.addUrlPatterns("/*");
        return registration;
    }

}
```
然而，并没有卵用。
我测试的时候用的是postman，它的默认的content-type为form-data

![小李飞菜刀](http://ofdzt5639.bkt.clouddn.com/postman.png)
改为x-www-form-urlencoded就好了。
但是如果form标签都要指定那么就麻烦了，查询【HTML form标签的 enctype 属性】得知默认为x-www-form-urlencoded

![form](http://ofdzt5639.bkt.clouddn.com/form.png)

ajax的默认contentType也是x-www-form-urlencoded，就放心了！

![ajax](http://ofdzt5639.bkt.clouddn.com/ajax.png)