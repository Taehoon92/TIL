# Type Converter

When using `@RequestParam`, `@ModelAttribute`, and `@PathVariable` in Spring Boot, the query data is originally `String` type. At this time, Spring convert the type in the middle specified by the developer.

>For example,
>```java
>* Controller
>@GetMapping("/converter")
>public String converter(@RequestParam Integer data) {
>    System.out.println("data = " + data);
>    return "ok";
>}
>```
>
>**Execute**  
>http://localhost:8080/converter?data=10  
>  
>  In the query string `data=10`, 10 is not a number 10, it is string 10. But using @RequestParam makes this string 10 is converted to `Integer` type number 10.

In addition Spring automatically converts types in,
* Read yml data using `@Value` etc.
* Spring MVC Request parameter
* Rendering view    
  
  
### Custom Converter
Spring provide extensible converter interface. To use a type converter, implement `org.springframework.core.convert.converter.Converter` interface. 

**Converter Interface**
```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {
    T convert(S source);
}
```

##### String to Integer Converter example
```java
// Implementation
public class MyConverter implements Converter<String, Integer> {
    @Override
    public Integer converter(String source) {
        return Integer.valueOf(source);
    }
}

// Test
@Test
void test() {
    MyConverter converter = new MyConverter();
    Integer result = converter.convert("10");
    Assertions.assertThat(result).isEqualTo(10);
}
```

##### Custom type Converter example
Convert a String such as `'127.0.0.1:8080'` into IpPort object.
```java
// IpPort Object
@Getter
@EqualsAndHashCode
public class IpPort {
    private String ip;
    private int port;

    public IpPort(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
}


// String to IpPort Converter Implementation
public class IpPortConverter implement Converter<String, IpPort> {

    @Override
    public IpPort convert(String source) {
        String[] split = source.split(":");
        String ip = split[0];
        int port = Integer.parseInt(split[1]);

        return new IpPort(ip, port);
    }
}

// Test
@Test
void test() {
    IpPortConverter converter = new IpPortConverter();
    String source = "127.0.0.1:8080";
    IpPort result = converter.convert(source);
    Assertions.assertThat(result).isEqualTo(new IpPort("127.0.0.1", 8080));
}
```

### Conversion Service
It's inconvenient to find the converter directly to use it every time. So, Spring provide a function to collect each converters and to be able to used by binding them.

In conversion interface, there are two functions, `canConvert` and `convert`.
`canConvert` is function that check if conversion is possible. `convert` is literally convert function.

For example,
```java
//Test
public class ConversionServiceTest {

    @Test
    void conversionService() {
        //register
        DefaultConversionService conversionService = new DefaultConversionService();
        conversionService.addConverter(new MyConverter());
        conversionService.addConverter(new IpPortConverter());

        //use
        Integer strToInt = conversionService.convert("10", Integer.class);
        Assersion.assertThat(strToInt).isEqualTo(10);

        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class)
        Assersion.assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));
    }
}
```

To use the conversion service, enter the value to be converted and the type to be converted as shown below.  
`Integer result = conversionService.conver("10", Integer.class)`

When register converter, we must know type converter clearly such as `MyConverter`, `IpPortConverter`. However, from the point of view of using converter, we don't have to know type converter at all. 

### Apply converter in Spring

To use convert in web application, you need to register converter in WebConfig. 

```java
//WebConfig
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormmaterRegistry registry) {
        registry.addConverter(new MyConverter());
        registry.addConverter(new IpPortConverter());
    }
}
```

Spring provide `ConversionService` inside. You can register the converter using `addFormatters()` provided by `WebMvcConfigurer`.

This is an example to check whether the registered converter works well.
```java
//Controller
@GetMapping("/ip-port")
public String ipPort(@RequestParam IpPort ipPort) {

    // String type input "127.0.0.1:8080" -> IpPort Object mapping!
    System.out.println("ip = " + ipPort.getIp());
    System.out.println("port = " + ipPort.getPort());
    return "ok";
}
```

**Execute**  
http://localhost:8080/ip-port?ipPort=127.0.0.1:8080  

**log**  
ip = 127.0.0.1  
port = 8080

Query string `?ipPort=127.0.0.1:8080` converts well at `@RequestParam IpPort ipPort`

