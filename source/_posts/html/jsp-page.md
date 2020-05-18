title: 做一个jsp自定义tag的bootstrap3公共分页
tags: 
 - jsp
 - bootcss
 - html
categories: 
 - html
 - jsp
date: 2015-07-20 11:35:35
---

## 预览
![分页](https://dn-tianjun.qbox.me/ttianjunQQ截图20150720135230.png)

## 想法
本来用模版分页更优雅的，但项目很多同事不熟悉，只好先做个jsp版本的。
方案就是写一个servlet的tag的分页tag实现，根据tag里面的参数拼凑html代码。
有几个关键参数，当前页，总页数，以及跳转链接
tag中获取当前页和总页数是从request中获取分页bean来获得的，有些耦合分页bean,但好处是前端分页标签可以不用传分页参数。
跳转链接这个实现，依赖前端的查询from，因为业务上，有分页的基本上都会有条件查询，所以直接用form提交了，点击分页跳转的时候会在form中插入当前页的数据到隐藏域供后端查询。

<!-- more -->

## 自定义tag
* 继承`javax.servlet.jsp.tagext.Tag`
* 实现方法

```java

public class PageTag implements Tag {
	private PageContext pageContext;
	 //当前页
    private Integer pageNum;

    //总页数
    private Integer pages;
    
    private String seachForm;
    
    private String paramEncoding = "UTF-8";

    /**
     * page bean的属性名
     */
    private String pageBeanName;



	public String getPageBeanName() {
		return pageBeanName;
	}

	public void setPageBeanName(String pageBeanName) {
		this.pageBeanName = pageBeanName;
	}

	public Integer getPageNum() {
		return pageNum;
	}

	public void setPageNum(Integer pageNum) {
		this.pageNum = pageNum;
	}

	public Integer getPages() {
		return pages;
	}

	public void setPages(Integer pages) {
		this.pages = pages;
	}

	public String getSeachForm() {
		return seachForm;
	}

	public void setSeachForm(String seachForm) {
		this.seachForm = seachForm;
	}

	public String getParamEncoding() {
		return paramEncoding;
	}

	public void setParamEncoding(String paramEncoding) {
		this.paramEncoding = paramEncoding;
	}


	@Override
	public void setPageContext(PageContext pc) {
		pageContext = pc;
	}

	
	@Override
	public void setParent(Tag t) {
		// TODO Auto-generated method stub
		
	}

	@Override
	public Tag getParent() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public void release() {
		// TODO Auto-generated method stub
		
	}

	@Override
	public int doStartTag() throws JspException {//标签调用前执行的方法
		if(pageBeanName==null)
			pageBeanName="pagination";
		
		Pagination pagination=(Pagination) pageContext.getRequest().getAttribute(pageBeanName);
		if(pagination!=null){
				pageNum = pagination.getPageNum();//当前页数
				pages = pagination.getTotalPage();//总页数
		}
		return 0;
	}
    
	@Override
	public int doEndTag() throws JspException {
		JspWriter out = pageContext.getOut();
		
		try {
			out.write(buildPage());
		} catch (IOException e) {
			e.printStackTrace();
			 throw new RuntimeException(e);
		}
		return 0;
	}
	private String buildPage(){
		if(pages==1){//如果只有一页 那么不显示分页
			return "";
		}
		int start=0,end=10;
		 if(pageNum>=10||(pageNum>5&&pages>10))         
		        start=pageNum-5;
		 if(pages>pageNum+5)
	            end=pageNum+5;
	        else
	            end=pages;
		 
		StringBuffer sb =new StringBuffer();
		sb.append("<nav class=\"pull-right pull-right-1\">");
		sb.append("<ul class=\"pagination\">");
		if(pageNum>1)
			sb.append("<li class=\"previous\"><a href=\"javascript:void(0)\" onclick="+getPageMethod(pageNum-1)+">上一页</a></li>");
		if(start>0)
			sb.append("<li class='nomal'  ><a href='javascript:void(0)' onclick="+getPageMethod(1)+" >"+1+"</a></li>");
		for(int i=start;i<end;i++){
			if((i+1)==pageNum)
				sb.append("<li class='active'><a href='javascript:void(0)' onclick="+getPageMethod(i+1)+" >"+(i+1)+"</a></li>");
	        else
	        	sb.append("<li class='nomal'><a href='javascript:void(0)' onclick="+getPageMethod(i+1)+">"+(i+1)+"</a></li>");
		}
		if(pages>10&&pageNum<pages-4)
			sb.append("<li class='nomal'><a href='javascript:void(0)' onclick="+getPageMethod(pages)+">"+pages+"</a></li>");
		if(pageNum!=pages&&pages!=0)
			sb.append("<li class=\"next\"><a href=\"javascript:void(0)\" onclick="+getPageMethod(pageNum+1)+">下一页</a></li>");
		sb.append("</ul>");
		sb.append("</nav>");
		return sb.toString();
	}
    private String  getPageMethod(int pageNum){
    	return "\"pageJunmp('"+seachForm+"',"+pageNum+",'"+pageBeanName+"')\"";
    }
}
```

## page.tld文件
在webapp/WEB-INF/tld下面新增page.tld，定义标签描述

```xml
<?xml version="1.0" encoding="UTF-8"?>
<taglib version="2.0" xmlns="http://java.sun.com/xml/ns/j2ee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee web-jsptaglibrary_2_0.xsd">
    <description>bootcss3 分页</description>
    <tlib-version>1.0</tlib-version>
    <short-name>page</short-name>
    <uri>http://xxx.com/tag/page</uri>
    <tag>
        <name>pager</name>
        <tag-class>com.xxx.xxx.core.PageTag</tag-class>
        <body-content>empty</body-content>
        <attribute>
            <name>seachForm</name>
            <required>true</required>
            <rtexprvalue>true</rtexprvalue>
        </attribute>
        <attribute>
            <name>pageBeanName</name>
            <required>false</required>
            <rtexprvalue>true</rtexprvalue>
        </attribute>
        <attribute>
            <name>paramEncoding</name>
            <required>false</required>
            <rtexprvalue>true</rtexprvalue>
        </attribute>
    </tag>
</taglib>
```

## Action层

```java
@Action("customerList")
	public String customerList() {
		//对应的struts2调用例子，struts2自动会把属性放到request作用域,分页会使用这个bean来获取分页信息.
		pagination = customerService.selectAll(pagination, null);//pagination 是一个分页bean，自行实现..
		return SUCCESS;
	}
```

## html层

### 头部引用

```
<%@taglib prefix="p" uri="http://xxx.com/tag/page" %>
```



### 这个是tag里面写的需要调用的js方法,写到一个公共引用的地方
```
function pageJunmp(form,pageNum,pageBeanName){
	var inputName = pageBeanName+".pageNum";
	form=$("#"+form);
	var input = "<input type='hidden' id='pageNum' value='"+pageNum+"' name='"+inputName+"'>";
	form.append(input);
	form.submit();
}	form.attr("method","post");

```

### 查询form
在要使用分页的地方，必须要一个查询的form，分页的时候会调用这个form，实现带条件跳转
```
<form action="${ctx}/customer/customerList" id="seachForm">
	<input type="text" name="account">
</form>
```

### 在需要分页的地方调用
```
<p:pager seachForm="seachForm"/>
```