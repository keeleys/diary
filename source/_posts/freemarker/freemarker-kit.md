title: 用Freemarker生成静态HTML
tags: freemarker
categories: web
date: 2015-07-10 15:13:51
---

## 配置

>javaweb struts2 spring3 mybatis环境下配置

### maven

```xml
<dependency>
	<groupId>org.freemarker</groupId>
	<artifactId>freemarker</artifactId>
	<version>2.3.19</version>
</dependency>
```
<!-- more -->

### web.xml设置servlet监听

```xml
<listener>
	<listener-class>com.ttianjun.listener.ContextListener</listener-class>
</listener>
```
对应的java类

```java
public class ContextListener implements ServletContextListener{
	private static Log log = LogFactory.getLog(ContextListener.class);
	private static ApplicationContext springContext = null;
	private static ServletContext servletContext = null;
	public static ApplicationContext getContext(){
	return springContext;
	}
	@Override
	public void contextInitialized(ServletContextEvent sce){
		servletContext = sce.getServletContext();
		//用servlet上下文初始化freemarker工具
		FreeMarkerKit.init(servletContext);
		//spring 上下文
		springContext = WebApplicationContextUtils.getWebApplicationContext(servletContext); 
		//web工程名
		servletContext.setAttribute("ctx",  servletContext.getContextPath());    
	}
}
```
## freemarker工具类书写

```java
public class FreeMarkerKit {
	private static Log log = LogFactory.getLog(FreeMarkerKit.class);
	private static Configuration cfg;
	private static ServletContext servletContext;
	public static void init(ServletContext servletContext){
		FreeMarkerKit.servletContext = servletContext;
		cfg = new Configuration();

		// webapp/WEB-INF/templates 目录
		cfg.setServletContextForTemplateLoading(servletContext, "/WEB-INF/templates"); 

		cfg.setEncoding(Locale.getDefault(), "utf-8");
	}
	public static void CreateHtml(String ftl,String htmlName,Map<String,Object> param) throws  Exception{
		
		if(param==null) param = new HashMap<String,Object>();
		param.put("ctx", servletContext.getAttribute("ctx").toString());
		
		Template template = cfg.getTemplate(ftl);
		template.setEncoding("utf-8");

		// webapp/html 目录
		String path = servletContext.getRealPath("/html"); 
		File file = new File(path , htmlName);
		
		if(!file.exists())file.getParentFile().mkdir(); 
		
		Writer writer = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(file), "utf-8"));
		template.process(param, writer);
		log.debug("生成静态文件:"+file.getAbsolutePath());
		writer.flush();
		writer.close();
		
	}
}
```

## 使用方法

index.ftl
```
welcome ${name}
```
调用
```java
Map<String, Object> params = new HashMap<String, Object>();
params.put("name","ttianjun")
FreeMarkerKit.CreateHtml("index.ftl","index.html",params);
```



