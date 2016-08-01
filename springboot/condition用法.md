#condition in spring boot

`spring boot`中的自动配置，大量使用了`conditional`注解



## ConditionalOnWebApplication

只有当应用为`web`应用时才其注册对应bean

其实现

```java

@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnWebApplicationCondition.class)
public @interface ConditionalOnWebApplication {
}

```

判断`OnWebApplicationCondition.class`信息，继承`SpringBootCondition`，其核心实现为

```java

	@Override
	public final boolean matches(ConditionContext context,
			AnnotatedTypeMetadata metadata) {
		String classOrMethodName = getClassOrMethodName(metadata);  //获取该注解对应的Class或者Method
		try {
			ConditionOutcome outcome = getMatchOutcome(context, metadata);  //子类实现具体的match实现
			logOutcome(classOrMethodName, outcome);  //日志输出
			recordEvaluation(context, classOrMethodName, outcome);  //记录
			return outcome.isMatch();
		}
		catch (NoClassDefFoundError ex) {
		}
	}
```

其中`getMatchOutcome`为抽象方法，返回`ConditionOutcome`对象，该对象包装了匹配结果和匹配信息，那么继承`SpringBootCondition`主要为
实现`getMatchOutcome`方法。


`OnWebApplicationCondition`主要通过`isWebApplication(context, metadata);`来判断结果


```java

/**
	 * 判断是否为web应用
	 * @param context
	 * @param metadata
	 * @return
	 */
	private ConditionOutcome isWebApplication(ConditionContext context,
			AnnotatedTypeMetadata metadata) {

		if (!ClassUtils.isPresent(WEB_CONTEXT_CLASS, context.getClassLoader())) {  //判断指定类是否存在
			return ConditionOutcome.noMatch("web application classes not found");
		}  

		if (context.getBeanFactory() != null) {
			String[] scopes = context.getBeanFactory().getRegisteredScopeNames();  //判断web特有的scope
			if (ObjectUtils.containsElement(scopes, "session")) {
				return ConditionOutcome.match("found web application 'session' scope");
			}
		}

		if (context.getEnvironment() instanceof StandardServletEnvironment) {  //看Environment
			return ConditionOutcome
					.match("found web application StandardServletEnvironment");
		}

		if (context.getResourceLoader() instanceof WebApplicationContext) {  //判断上下文
			return ConditionOutcome.match("found web application WebApplicationContext");
		}

		return ConditionOutcome.noMatch("not a web application");
	}

```

其他的实现结构类似



##ConditionalOnBean

在指定的bean存在的时候创建被标注的bean


##ConditionalOnClass

在指定的Class存在的时候创建被标注的bean


##ConditionalOnExpression

在指定的SpEL返回true时

##ConditionalOnJava

在指定的java版本

##ConditionalOnJndi

获取指定的`Jndi`时

##ConditionalOnMissingBean

在指定的bean不存在时

##ConditionalOnMissingClass

在指定的class不存在时

##ConditionalOnNotWebApplication

非web环境下


##ConditionalOnProperty

检查指定的属性匹配指定值

##ConditionalOnResource

存在指定的资源

##ConditionalOnSingleCandidate

存在指定单例的bean








