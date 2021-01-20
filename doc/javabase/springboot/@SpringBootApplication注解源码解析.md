## 一、`@SpringBootApplication` 的作用是什么？

​	**Q：springboot项目的启动类上，都会有个注解`@SpringBootApplication`，这个注解起到了什么作用？**

```java
@SpringBootApplication
public class MicroServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MicroServiceApplication.class, args);
    }

}
```



​	我们进入`@SpringBootApplication`注解，发现它**等价于三个注解**：`@SpringBootConfiguration` `@EnableAutoConfiguration` `@ComponentScan`  

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	...
}
```

1） `@SpringBootConfiguration` 就相当于 `@Configuration`

```jav
@Configuration
public @interface SpringBootConfiguration {

}
```



2） `@EnabelAutoConfiguration ` 相当于将这两个类的实例加入到容器中`AutoConfigurationImportSelector.class`   `AutoConfigurationPackages.Registrar.class `

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    ...
}
```

```java
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

}
```

- **`AutoConfigurationImportSelector.class`的作用是，注入spring.factories文件中EnableAutoConfiguration对应的类的实例，当然要经过spring.factories文件中AutoConfigurationImportFilter对应的过滤器（`OnBeanCondition`、`OnClassCondition`、`OnWebApplicationCondition`等等）的过滤。还要排除掉`@EnableAutoConfiguration`中的exclude和excludeName**

  具体见`ConfigurationClassParser`的getImports方法，其中调用了`AutoConfigurationImportSelector`的process方法和selectImports方法。

  ```java
  public Iterable<Group.Entry> getImports() {
  			for (DeferredImportSelectorHolder deferredImport : this.deferredImports) {
  				this.group.process(deferredImport.getConfigurationClass().getMetadata(),
  						deferredImport.getImportSelector());
  			}
  			return this.group.selectImports();
  		}
  ```

    `AutoConfigurationImportSelector`实现了`DeferredImportSelector`，所以它是解析`@Configuration`的最后一步。`DeferredImportSelector`可以和`@Order`搭配使用。`AutoConfigurationImportSelector`的意义在于引入其他包时，可以直接注入其他包的`@Configuration`，当然需要在其他包的resources文件夹下，新建META-INF目录，在META-INF目录下新建spring.factories文件，加入org.springframework.boot.autoconfigure.EnableAutoConfiguration = @Configuration标注的类的路径



- **`AutoConfigurationPackages.Registrar.class`的作用是，注入一个名称为AutoConfigurationPackages的`BasePackages.class`实例。这个实例的作用在于保存自动扫描的包路径，供以后使用（比如JPA 的entity扫描）**

  ```java
  public static void register(BeanDefinitionRegistry registry, String... packageNames) {
  		if (registry.containsBeanDefinition(BEAN)) {
  			BeanDefinition beanDefinition = registry.getBeanDefinition(BEAN);
  			ConstructorArgumentValues constructorArguments = beanDefinition
  					.getConstructorArgumentValues();
  			constructorArguments.addIndexedArgumentValue(0,
  					addBasePackages(constructorArguments, packageNames));
  		}
  		else {
  			GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
  			beanDefinition.setBeanClass(BasePackages.class);
  			beanDefinition.getConstructorArgumentValues().addIndexedArgumentValue(0,
  					packageNames);
  			beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
  			registry.registerBeanDefinition(BEAN, beanDefinition);
  		}
  	}
  
  	private static String[] addBasePackages(
  			ConstructorArgumentValues constructorArguments, String[] packageNames) {
  		String[] existing = (String[]) constructorArguments
  				.getIndexedArgumentValue(0, String[].class).getValue();
  		Set<String> merged = new LinkedHashSet<>();
  		merged.addAll(Arrays.asList(existing));
  		merged.addAll(Arrays.asList(packageNames));
  		return StringUtils.toStringArray(merged);
  	}
  ```

  

3） `@ComponentScan` 当然是将路径下合适的类加载到容器中

  **Q：它为什么要用到`TypeExcludeFilter.class` 和`AutoConfigurationExcludeFilter.class` 这两个过滤器**

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	...
}
```

- **这两个过滤器在`ComponentScanAnnotationParser`的parse方法中，被加入**

  ```java
  public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
      //此处加入了三个默认的includeFilter，一个用来筛选@Componment标记的类，一个用来筛选javax.annotation.ManagedBean标记的类，一个用来筛选JSR-330中javax.inject.Named标记的类（如果引入了JSR-330依赖注入标准的话，即引入javax.inject包）
      ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,
  				componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);
  		...
              
  		for (AnnotationAttributes filter : componentScan.getAnnotationArray("excludeFilters")) {
  			for (TypeFilter typeFilter : typeFiltersFor(filter)) {
  				scanner.addExcludeFilter(typeFilter);
  			}
  		}
  
  		...
  
  		scanner.addExcludeFilter(new AbstractTypeHierarchyTraversingFilter(false, false) {
  			@Override
  			protected boolean matchClassName(String className) {
  				return declaringClass.equals(className);
  			}
  		});
  		return scanner.doScan(StringUtils.toStringArray(basePackages));
  	}
  ```

	**也是在parse这个方法里调用了 typeFiltersFor方法，对`TypeFilter.class`的实现类进行了实例化**。（也就是说`TypeExcludeFilter.class`  `AutoConfigurationExcludeFilter.class` `AbstractTypeHierarchyTraversingFilter.class`都在这里进行了实例化，但是没有加入spring的bean池）

  ```java
  private List<TypeFilter> typeFiltersFor(AnnotationAttributes filterAttributes) {
  		List<TypeFilter> typeFilters = new ArrayList<>();
  		FilterType filterType = filterAttributes.getEnum("type");
  
  		for (Class<?> filterClass : filterAttributes.getClassArray("classes")) {
  			switch (filterType) {
  				case ANNOTATION:
  					Assert.isAssignable(Annotation.class, filterClass,
  							"@ComponentScan ANNOTATION type filter requires an annotation type");
  					@SuppressWarnings("unchecked")
  					Class<Annotation> annotationType = (Class<Annotation>) filterClass;
  					typeFilters.add(new AnnotationTypeFilter(annotationType));
  					break;
  				case ASSIGNABLE_TYPE:
  					typeFilters.add(new AssignableTypeFilter(filterClass));
  					break;
  				case CUSTOM:
  					Assert.isAssignable(TypeFilter.class, filterClass,
  							"@ComponentScan CUSTOM type filter requires a TypeFilter implementation");
  					TypeFilter filter = BeanUtils.instantiateClass(filterClass, TypeFilter.class);
  					ParserStrategyUtils.invokeAwareMethods(
  							filter, this.environment, this.resourceLoader, this.registry);
  					typeFilters.add(filter);
  					break;
  				default:
  					throw new IllegalArgumentException("Filter type not supported with Class value: " + filterType);
  			}
  		}
  
  		for (String expression : filterAttributes.getStringArray("pattern")) {
  			switch (filterType) {
  				case ASPECTJ:
  					typeFilters.add(new AspectJTypeFilter(expression, this.resourceLoader.getClassLoader()));
  					break;
  				case REGEX:
  					typeFilters.add(new RegexPatternTypeFilter(Pattern.compile(expression)));
  					break;
  				default:
  					throw new IllegalArgumentException("Filter type not supported with String pattern: " + filterType);
  			}
  		}
  
  		return typeFilters;
  	}
  ```

	`ComponentScanAnnotationParser`的parse方法中，scanner最后为什么又加了一个`AbstractTypeHierarchyTraversingFilter`呢？我们看一下它的match方法，发现是**过滤掉启动类，不让它作为@configuration标记的候选类，避免再一次解析启动类上的各种注解**（因为它的两个参数considerInherited和considerInterfaces在scanner.addExcludeFilter这条语句中，设置成了false，导致下面的判断语句不生效）。

  ```java
  public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
  			throws IOException {
  
  		...
  		ClassMetadata metadata = metadataReader.getClassMetadata();
  		if (matchClassName(metadata.getClassName())) {
  			return true;
  		}
  
  		if (this.considerInherited) {
  			...
  		}
  
  		if (this.considerInterfaces) {
  			...
  		}
  
  		return false;
  	}
  ```

  **注：启动类本身还是会注入到spring的bean池中的，具体见`SpringApplication`的load方法**

  

- **ExcludeFilter在`ClassPathScanningCandidateComponentProvider`的isCandidateComponent方法中被使用**。

  ```java
  protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
  		for (TypeFilter tf : this.excludeFilters) {
  			if (tf.match(metadataReader, getMetadataReaderFactory())) {
  				return false;
  			}
  		}
  		for (TypeFilter tf : this.includeFilters) {
  			if (tf.match(metadataReader, getMetadataReaderFactory())) {
  				return isConditionMatch(metadataReader);
  			}
  		}
  		return false;
  	}
  ```



- **`AutoConfigurationExcludeFilter` 的作用是过滤掉会自动配置的配置类，避免重复**

  ```java
  @Override
  	public boolean match(MetadataReader metadataReader,
  			MetadataReaderFactory metadataReaderFactory) throws IOException {
  		//如果这个类被@Configuration标注，且属于自动加载的配置，那么过滤它，避免重复
          return isConfiguration(metadataReader) && isAutoConfiguration(metadataReader);
  	}
  
  	private boolean isConfiguration(MetadataReader metadataReader) {
  		return metadataReader.getAnnotationMetadata()
  				.isAnnotated(Configuration.class.getName());
  	}
  
  	private boolean isAutoConfiguration(MetadataReader metadataReader) {
  		return getAutoConfigurations()
  				.contains(metadataReader.getClassMetadata().getClassName());
  	}
  
  	protected List<String> getAutoConfigurations() {
  		if (this.autoConfigurations == null) {
              /**
              从META-INF/spring.factories文件中，找出EnableAutoConfiguration.class
              多个jar包中，都存在spring.factories文件
              其中包含EnableAutoConfiguration.class的spring.factories文件，位于spring-boot-autoconfigure
              **/
  			this.autoConfigurations = SpringFactoriesLoader.loadFactoryNames(
  					EnableAutoConfiguration.class, this.beanClassLoader);
  		}
  		return this.autoConfigurations;
  	}
  ```

  

- **`TypeExcludeFilter` 的作用是加载spring bean池中所有针对TypeExcludeFilter的扩展，并循环遍历这些扩展类调用其match方法**

  ```java
  public boolean match(MetadataReader metadataReader,
  			MetadataReaderFactory metadataReaderFactory) throws IOException {
  		if (this.beanFactory instanceof ListableBeanFactory
  				&& getClass() == TypeExcludeFilter.class) {
              //加载spring bean池中所有针对TypeExcludeFilter的拓展
  			Collection<TypeExcludeFilter> delegates = ((ListableBeanFactory) this.beanFactory)
  					.getBeansOfType(TypeExcludeFilter.class).values();
              // 循环遍历，调用其match方法
  			for (TypeExcludeFilter delegate : delegates) {
  				if (delegate.match(metadataReader, metadataReaderFactory)) {
  					return true;
  				}
  			}
  		}
  		return false;
  	}
  ```

>也欢迎关注微信公众号【Ccww技术博客】，原创技术文章第一时间推出  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200414231212786.png)
>如果此文对你有帮助、喜欢的话，那就点个赞呗，点个关注呗！
