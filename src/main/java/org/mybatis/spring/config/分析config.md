

主要是用于解析xml配置中的关键定义, 然后设置到mapperFactoryBean中, 就给它使用
```
private static final String ATTRIBUTE_BASE_PACKAGE = "base-package"; // 基本包
  private static final String ATTRIBUTE_ANNOTATION = "annotation";
  private static final String ATTRIBUTE_MARKER_INTERFACE = "marker-interface"; // 指定mapper接口
  private static final String ATTRIBUTE_NAME_GENERATOR = "name-generator";
  private static final String ATTRIBUTE_TEMPLATE_REF = "template-ref";  //指定sqlSessionTemplate
  private static final String ATTRIBUTE_FACTORY_REF = "factory-ref";  // 指定sqlSessionFactory
  
  //指定生成mapper代理类的工厂类, 一般是org.mybatis.spring.mapper.MapperFactoryBean
  private static final String ATTRIBUTE_MAPPER_FACTORY_BEAN_CLASS = "mapper-factory-bean-class"; 
```
