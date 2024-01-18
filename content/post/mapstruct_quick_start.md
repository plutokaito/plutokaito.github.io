
## 作用
MapStruct 用于简化 Java 对象之间的映射，在编译时生成代码，提供对象映射的性能，并减少手动编写映射代码的工作。 

## 常用注解
### @Mapper
MyBatis 中也有同名的注解，但功能不一样。

注解解析：
```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.CLASS)  
public @interface Mapper {

...
/**
Specifies the component model to which the generated mapper should adhere. Supported values are
default: the mapper uses no component model, instances are typically retrieved via Mappers.getMapper(Class)
cdi: the generated mapper is an application-scoped CDI bean and can be retrieved via @Inject
spring: the generated mapper is a Spring bean and can be retrieved via @Autowired
jsr330: the generated mapper is annotated with @javax.inject.Named and @Singleton, and can be retrieved via @Inject. The annotations will either be from javax.inject or jakarta.inject, depending on which one is available, with javax.inject having precedence.
jakarta: the generated mapper is annotated with @jakarta.inject.Named and @Singleton, and can be retrieved via @Inject.
The method overrides a componentModel set in a central configuration set by config()
Returns:
The component model for the generated mapper.
*/
String componentModel() default MappingConstants.ComponentModel.DEFAULT;

...

}
```


### @Mapping

```java
/**
 * Configures the mapping of one bean attribute or enum constant.
 * <p>
 * The name of the mapped attribute or constant is to be specified via {@link #target()}. For mapped bean attributes it
 * is assumed by default that the attribute has the same name in the source bean. Alternatively, one of
 * {@link #source()}, {@link #expression()} or {@link #constant()} can be specified to define the property source.
 * </p>
 * <p>
 * In addition, the attributes {@link #dateFormat()} and {@link #qualifiedBy()} may be used to further define the
 * mapping.
 * </p>
 *
 * <p>
 * <strong>Example 1:</strong> Implicitly mapping fields with the same name:
 * </p>
 * <pre><code class='java'>
 * // Both classes HumanDto and Human have property with name "fullName"
 * // properties with the same name will be mapped implicitly
 * &#64;Mapper
 * public interface HumanMapper {
 *    HumanDto toHumanDto(Human human)
 * }
 * </code></pre>
 * <pre><code class='java'>
 * // generates:
 * &#64;Override
 * public HumanDto toHumanDto(Human human) {
 *    humanDto.setFullName( human.getFullName() );
 *    // ...
 * }
 * </code></pre>
 *
 * <p><strong>Example 2:</strong> Mapping properties with different names</p>
 * <pre><code class='java'>
 * // We need map Human.companyName to HumanDto.company
 * // we can use &#64;Mapping with parameters {@link #source()} and {@link #target()}
 * &#64;Mapper
 * public interface HumanMapper {
 *    &#64;Mapping(source="companyName", target="company")
 *    HumanDto toHumanDto(Human human)
 * }
 * </code></pre>
 * <pre><code class='java'>
 * // generates:
 * &#64;Override
 * public HumanDto toHumanDto(Human human) {
 *     humanDto.setCompany( human.getCompanyName() );
 *      // ...
 * }
 * </code></pre>
 * <p>
 * <strong>Example 3:</strong> Mapping with expression
 * <b>IMPORTANT NOTE:</b> Now it works only for Java
 * </p>
 * <pre><code class='java'>
 * // We need map Human.name to HumanDto.countNameSymbols.
 * // we can use {@link #expression()} for it
 * &#64;Mapper
 * public interface HumanMapper {
 *    &#64;Mapping(target="countNameSymbols", expression="java(human.getName().length())")
 *    HumanDto toHumanDto(Human human)
 * }
 * </code></pre>
 * <pre><code class='java'>
 * // generates:
 *&#64;Override
 * public HumanDto toHumanDto(Human human) {
 *    humanDto.setCountNameSymbols( human.getName().length() );
 *    //...
 * }
 * </code></pre>
 * <p>
 * <strong>Example 4:</strong> Mapping to constant
 * </p>
 * <pre><code class='java'>
 * // We need map HumanDto.name to string constant "Unknown"
 * // we can use {@link #constant()} for it
 * &#64;Mapper
 * public interface HumanMapper {
 *    &#64;Mapping(target="name", constant="Unknown")
 *    HumanDto toHumanDto(Human human)
 * }
 * </code></pre>
 * <pre><code class='java'>
 * // generates
 * &#64;Override
 * public HumanDto toHumanDto(Human human) {
 *   humanDto.setName( "Unknown" );
 *   // ...
 * }
 * </code></pre>
 * <p>
 * <strong>Example 5:</strong> Mapping with default value
 * </p>
 * <pre><code class='java'>
 * // We need map Human.name to HumanDto.fullName, but if Human.name == null, then set value "Somebody"
 * // we can use {@link #defaultValue()} or {@link #defaultExpression()} for it
 * &#64;Mapper
 * public interface HumanMapper {
 *    &#64;Mapping(source="name", target="name", defaultValue="Somebody")
 *    HumanDto toHumanDto(Human human)
 * }
 * </code></pre>
 * <pre><code class='java'>
 * // generates
 * &#64;Override
 * public HumanDto toHumanDto(Human human) {
 *    if ( human.getName() != null ) {
 *       humanDto.setFullName( human.getName() );
 *    }
 *    else {
 *       humanDto.setFullName( "Somebody" );
 *    }
 *   // ...
 * }
 * </code></pre>
 *
 * <b>IMPORTANT NOTE:</b> the enum mapping capability is deprecated and replaced by {@link ValueMapping} it
 * will be removed in subsequent versions.
 *
 * @author Gunnar Morling
 */
@Repeatable(Mappings.class)  
@Retention(RetentionPolicy.CLASS)  
@Target({ ElementType.METHOD, ElementType.ANNOTATION_TYPE })  
public @interface Mapping {
	...
}
```

## 使用
### Apache Maven 设置
```XML
...
<properties>
    <!-- https://github.com/mapstruct/mapstruct/releases -->
	<org.mapstruct.version>1.6.0.Beta1</org.mapstruct.version>
	<lombok-mapstruct.version>0.2.0</lombok-mapstruct.version>
</properties>
...

<dependencies>
	<dependency>  
	   <groupId>org.mapstruct</groupId>  
	   <artifactId>mapstruct</artifactId>  
	   <version>${org.mapstruct.version}</version>  
	</dependency>
    <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
    </dependency>
	...
</dependencies>

<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-compiler-plugin</artifactId>  
            <version>3.11.0</version>  
            <configuration>  
                <annotationProcessorPaths>  
                    <path>  
                        <groupId>org.mapstruct</groupId>  
                        <artifactId>mapstruct-processor</artifactId>  
                        <version>${org.mapstruct.version}</version>  
                    </path>
                    <!-- 支持lombok -->  
                    <path>  
                        <groupId>org.projectlombok</groupId>  
                        <artifactId>lombok</artifactId>  
                        <version>${lombok.version}</version>  
                    </path>  
                    <path>  
                        <groupId>org.projectlombok</groupId>  
                        <artifactId>lombok-mapstruct-binding</artifactId>  
                        <version>${lombok-mapstruct.version}</version>  
                    </path>  
                </annotationProcessorPaths>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>
```

### Java 代码
```java
public class SimpleSource {
	private String name;  
	private String description;
	private Integer simpleLevel;
	private UserDTO userDTO;
	// getter setter
}

public class SimpleDestination {  
    private String name;  
    private String description;
    private Integer level;
    private User user;
    // getter setter
}

public class User {  
    private Integer id;  
    private String userName;
	// getter setter
}


public class UserDTO {  
    private Integer id;  
    private String name;
	// getter setter 
}

// lombok 对象的使用
@Data  
public class SimpleLombokDestination {  
    private String name;  
    private String description;  
    private Integer level;  
}

// 这里使用 spring 组件模式，可以更好的和 spring bean 相结合
@Mapper(componentModel = "spring") 
public interface SimpleSourceDestinationMapper {  
    @Mapping(source = "simpleLevel", target = "level")
    @Mapping(source = "userDTO", target = "user")
    @Mapping(source = "userDTO.name", target = "user.userName")
    SimpleDestination fromSource(SimpleSource simpleSource);
    
    @Mapping(source = "level", target = "simpleLevel")
    SimpleSource fromDistination(SimpleDestination simpleDestination);
}

```

### 编译后自动生成的代码
此代码为编译后自动生成的代码，位置在：`target\generated-sources\annotations` 中

```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    comments = "version: 1.6.0.Beta1, compiler: javac, environment: Java 17.0.7 (Oracle Corporation)"
)
@Component
public class SimpleSourceDestinationMapperImpl implements SimpleSourceDestinationMapper {

    @Override
    public SimpleDestination fromSource(SimpleSource simpleSource) {
        if ( simpleSource == null ) {
            return null;
        }

        SimpleDestination simpleDestination = new SimpleDestination();

        simpleDestination.setUser( userDTOToUser( simpleSource.getUserDTO() ) );
        simpleDestination.setLevel( simpleSource.getSimpleLevel() );
        simpleDestination.setName( simpleSource.getName() );
        simpleDestination.setDescription( simpleSource.getDescription() );

        return simpleDestination;
    }

    @Override
    public SimpleLombokDestination lombokFromSource(SimpleSource simpleSource) {
        if ( simpleSource == null ) {
            return null;
        }

        SimpleLombokDestination simpleLombokDestination = new SimpleLombokDestination();

        simpleLombokDestination.setLevel( simpleSource.getSimpleLevel() );
        simpleLombokDestination.setName( simpleSource.getName() );
        simpleLombokDestination.setDescription( simpleSource.getDescription() );

        return simpleLombokDestination;
    }

    protected User userDTOToUser(UserDTO userDTO) {
        if ( userDTO == null ) {
            return null;
        }

        User user = new User();

        user.setUserName( userDTO.getName() );
        user.setId( userDTO.getId() );

        return user;
    }
 ...
}
```

## 测试代码

```java
@RunWith(SpringRunner.class)  
@SpringBootTest  
public class SimpleSourceDestinationMapperTest {  
  
    @Autowired  
    private SimpleSourceDestinationMapper simpleSourceDestinationMapper;  
  
    @Test  
    public void testFromSource() {  
        SimpleSource simpleSource = new SimpleSource();  
        simpleSource.setDescription("xxxxxx1");  
        simpleSource.setName("aaaaaa");  
        simpleSource.setSimpleLevel(111111);  
        UserDTO userDTO = new UserDTO();  
        userDTO.setId(1111);  
        userDTO.setName("name111111");  
        simpleSource.setUserDTO(userDTO);  
        SimpleDestination simpleDestination = simpleSourceDestinationMapper.fromSource(simpleSource);  
        System.out.println(simpleDestination);  
        assertEquals(simpleDestination.getDescription(), simpleSource.getDescription());  
        assertEquals(simpleDestination.getName(), simpleSource.getName());  
        assertEquals(simpleDestination.getLevel(), simpleSource.getSimpleLevel());  
        assertEquals(simpleDestination.getUser().getUserName(), simpleSource.getUserDTO().getName());  
    }  
  
    @Test  
    public void testLombokFromSource() {  
        SimpleSource simpleSource = new SimpleSource();  
        simpleSource.setDescription("xxxxxx1");  
        simpleSource.setName("aaaaaa");  
        simpleSource.setSimpleLevel(111111);  
        SimpleLombokDestination simpleDestination = simpleSourceDestinationMapper.lombokFromSource(simpleSource);  
        assertEquals(simpleDestination.getDescription(), simpleSource.getDescription());  
        assertEquals(simpleDestination.getName(), simpleSource.getName());  
        assertEquals(simpleDestination.getLevel(), simpleSource.getSimpleLevel());  
    }  
}
```


## 和 BeanUtils.copyProperties 的对比

copyProperties 使用的反射实现，而 MapStruct 是直接生成了代码。所以性能上要比 BeanUtils.copyProperties 好一些

```java
private static void copyProperties(Object source, Object target, @Nullable Class<?> editable, @Nullable String... ignoreProperties) throws BeansException {
	...
	PropertyDescriptor[] targetPds = getPropertyDescriptors(actualEditable);  
Set<String> ignoredProps = (ignoreProperties != null ? new HashSet<>(Arrays.asList(ignoreProperties)) : null);  
CachedIntrospectionResults sourceResults = (actualEditable != source.getClass() ?  
      CachedIntrospectionResults.forClass(source.getClass()) : null);  
  
for (PropertyDescriptor targetPd : targetPds) {  
   Method writeMethod = targetPd.getWriteMethod();  
   if (writeMethod != null && (ignoredProps == null || !ignoredProps.contains(targetPd.getName()))) {  
      PropertyDescriptor sourcePd = (sourceResults != null ?  
            sourceResults.getPropertyDescriptor(targetPd.getName()) : targetPd);  
      if (sourcePd != null) {  
         Method readMethod = sourcePd.getReadMethod();  
         if (readMethod != null) {  
            if (isAssignable(writeMethod, readMethod, sourcePd, targetPd)) {  
               try {  
                  ReflectionUtils.makeAccessible(readMethod);  
                  Object value = readMethod.invoke(source);  
                  ReflectionUtils.makeAccessible(writeMethod);  
                  writeMethod.invoke(target, value);  
               }  
               catch (Throwable ex) {  
                  throw new FatalBeanException(  
                        "Could not copy property '" + targetPd.getName() + "' from source to target", ex);  
               }  
            }  
         }  
      }  
    }  
  }
}
```


## 参考资料

1. [MapStruct 1.6.0.Beta1 Reference Guide](https://mapstruct.org/documentation/dev/reference/html/)