---
layout: post
category : java
title : Spring MVC参数绑定过程
tag : [java, Spring, Data bind]
blog: true
---

对于http://domain/path1/path2/mothod?k1=v1&k2=v2这种请求

对应的Controller方法为

```
@RequestMapping("/path1/path2/mothod")
// model中含有k2属性
public ModelAndView mothod( String k1, Model model) {
  return null;
}
```

那么Spring是如何把k1, k2的值是如何注入到相应的位置上呢?

{{ site.excerpt_separator }}

Spring源码在:org.springframework.web.method.support.InvocableHandlerMethod

### 执行请求 ###

```
public final Object invokeForRequest(NativeWebRequest request, ModelAndViewContainer mavContainer, 
	Object... providedArgs) throws Exception {
	Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);

	if (logger.isTraceEnabled()) {
		StringBuilder builder = new StringBuilder("Invoking [");
		builder.append(this.getMethod().getName()).append("] method with arguments ");
		builder.append(Arrays.asList(args));
		logger.trace(builder.toString());
	}

	Object returnValue = invoke(args);

	if (logger.isTraceEnabled()) {
		logger.trace("Method [" + this.getMethod().getName() + "] returned [" + returnValue + "]");
	}

	return returnValue;
}
```

### 解析参数列表 ###

```
private Object[] getMethodArgumentValues(NativeWebRequest request, ModelAndViewContainer mavContainer, 
	Object... providedArgs) throws Exception {
	MethodParameter[] parameters = getMethodParameters();
	Object[] args = new Object[parameters.length];
	for (int i = 0; i < parameters.length; i++) {
		MethodParameter parameter = parameters[i];
		parameter.initParameterNameDiscovery(parameterNameDiscoverer);
		GenericTypeResolver.resolveParameterType(parameter, getBean().getClass());

		args[i] = resolveProvidedArgument(parameter, providedArgs);
		if (args[i] != null) {
			continue;
		}

		if (argumentResolvers.supportsParameter(parameter)) {
			try {
				args[i] = argumentResolvers.resolveArgument(parameter, mavContainer, request, dataBinderFactory);
				continue;
			} catch (Exception ex) {
				if (logger.isTraceEnabled()) {
					logger.trace(getArgumentResolutionErrorMessage("Error resolving argument", i), ex);
				}
				throw ex;
			}
		}

		if (args[i] == null) {
			String msg = getArgumentResolutionErrorMessage("No suitable resolver for argument", i);
			throw new IllegalStateException(msg);
		}
	}
	return args;
}
```

getMethodParameters(); 方法返回对应的Controller实现方法的参数列表, 然后对每个参数绑定(注入)数据
代码关键在于:args[i] = argumentResolvers.resolveArgument(parameter, mavContainer, request, dataBinderFactory);


### 获取每个参数值 ###
可以看到, 对每种类型的参数, 获取方式在org.springframework.web.method.support.HandlerMethodArgumentResolverComposite,

```
public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, 
	NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
	HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
	Assert.notNull(resolver, "Unknown parameter type [" + parameter.getParameterType().getName() + "]");
	return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
}
```

可以看到每种参数类型的处理逻辑是不同的, Spring通过HandlerMethodArgumentResolver接口实现了常见类型的处理逻辑
对于基本类型, 枚举类型调用: org.springframework.web.method.annotation.AbstractNamedValueMethodArgumentResolver实现
代码很简单, 基本上就是

```
String[] paramValues = webRequest.getParameterValues(name);
if (paramValues != null) {
	arg = paramValues.length == 1 ? paramValues[0] : paramValues;
}
```


对于自定义model调用:
org.springframework.web.method.annotation.ModelAttributeMethodProcessor,
org.springframework.web.servlet.mvc.method.annotation.ServletModelAttributeMethodProcessor 

```
@Override
protected final Object createAttribute(String attributeName, MethodParameter parameter, 
	WebDataBinderFactory binderFactory, NativeWebRequest request) throws Exception {
	String value = getRequestValueForAttribute(attributeName, request);
	if (value != null) {
		Object attribute = createAttributeFromRequestValue(value, attributeName, parameter, binderFactory, request);
		if (attribute != null) {
			return attribute;
		}
	}

	return super.createAttribute(attributeName, parameter, binderFactory, request);
}
```

先从request中取出属性值, 如果不为空且有自定义类型转换服务, 把request中的返回的字符串按用户自定的类型转换实现进行解析

```
protected Object createAttributeFromRequestValue(String sourceValue, String attributeName, 
	MethodParameter parameter, WebDataBinderFactory binderFactory, NativeWebRequest request) throws Exception {
	DataBinder binder = binderFactory.createBinder(request, null, attributeName);
	ConversionService conversionService = binder.getConversionService();
	if (conversionService != null) {
		TypeDescriptor source = TypeDescriptor.valueOf(String.class);
		TypeDescriptor target = new TypeDescriptor(parameter);
		if (conversionService.canConvert(source, target)) {
			return binder.convertIfNecessary(sourceValue, parameter.getParameterType(), parameter);
		}
	}
	return null;
}
```

如果为空, 则调用默认的数据绑定逻辑进行参数注入

```
public final Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, 
	NativeWebRequest request, WebDataBinderFactory binderFactory) throws Exception {
	String name = ModelFactory.getNameForParameter(parameter);
	Object attribute = (mavContainer.containsAttribute(name)) ?
		mavContainer.getModel().get(name) : createAttribute(name, parameter, binderFactory, request);

	WebDataBinder binder = binderFactory.createBinder(request, attribute, name);
	if (binder.getTarget() != null) {
		bindRequestParameters(binder, request);
		validateIfApplicable(binder, parameter);
		if (binder.getBindingResult().hasErrors()) {
			if (isBindExceptionRequired(binder, parameter)) {
				throw new BindException(binder.getBindingResult());
			}
		}
	}

	// Add resolved attribute and BindingResult at the end of the model

	Map<String, Object> bindingResultModel = binder.getBindingResult().getModel();
	mavContainer.removeAttributes(bindingResultModel);
	mavContainer.addAllAttributes(bindingResultModel);

	return binder.getTarget();
}
```

其中bindRequestParameters(binder, request);是关键

### 自定义类型转换 ###
Spring3的类型转换接口实现逻辑如下:

```
public class AndroidLoginContextConverter implements Converter<String, AndroidLoginContext> {

	@Override
	public AndroidLoginContext convert( String source ) {
		return JSON.parseObject( source, AndroidLoginContext.class );
	}

}
```

在context.xml中配置

```
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<property name="converters">
	       	<list>
	            	<bean class="com.meizu.reader.web.android.AndroidLoginContextConverter"/>  
	        </list>
    	</property>
</bean>
	
<bean id="webBindingInitializer" class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">  
    	<property name="conversionService" ref="conversionService"/>  
</bean>
	
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">  
	<property name="webBindingInitializer" ref="webBindingInitializer"/>  
</bean>
```

然后对request需要做包装

```
public class AndroidUserinfoRequestWrapper extends HttpServletRequestWrapper {
  	@Override
	public String getParameter(String name) {
		if( "androidLoginContext".equals( name ) ) {
			return JSON.toJSONString( login );
		}
		
		String[] values = this.getParameterValues(name);
		if (values == null) {
			return null;
		}
		else {
			return values[0];
		}
	}
}
```

其中如果参数类型为model对象, 参数名为类名的SimpleName, 首字母小写
如果为数据或者为集合, 则为类名的SimpleName+List



