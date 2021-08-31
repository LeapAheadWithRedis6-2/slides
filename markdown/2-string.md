### String

Add GET parameter to SET command, for more powerful GETSET (#7852)

```java [9]
@SpringBootApplication
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

  @Bean
  public Function<String, String> uppercase() {
    return value -> value.toUpperCase();
  }
}
```

---
### String

Add GETEX, GETDEL commands (#8327)

---
### String

Add PXAT/EXAT arguments to SET command (#8327)


```java

```

---
### Repository URL

https://github.com/LeapAheadWithRedis6-2/com.redisgeek.function.redis.string.leapahead-wip

Notes:
- DaShaun
