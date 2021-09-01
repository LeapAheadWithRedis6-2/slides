### String

SET

```java
@Component
class ValueSet implements Function<KeyValue, String> {
	private final StringRedisTemplate redisTemplate;
	public ValueSet(StringRedisTemplate redisTemplate){
		this.redisTemplate = redisTemplate;
	}
	@Override
	public String apply(KeyValue input) {
		redisTemplate
				.opsForValue()
				.set(input.getKey(), input.getValue());
		return "OK";
	}
}
```
---

GET

```java
@Component
class Get implements Function<String, String> {
	private final StringRedisTemplate redisTemplate;
	public ValueGet(StringRedisTemplate redisTemplate){
		this.redisTemplate = redisTemplate;
	}
	@Override
	public String apply(String input) {
		return redisTemplate
                .opsForValue()
                .get(input);
	}
}
```

---
GETSET

```java
@Component
class GetAndSet implements Function<KeyValue, String> {
	private final StringRedisTemplate redisTemplate;
	public GetAndSet(StringRedisTemplate redisTemplate){
		this.redisTemplate = redisTemplate;
	}
	@Override
	public String apply(KeyValue input) {
		return redisTemplate
				.opsForValue()
				.getAndSet(input.getKey(), input.getValue());
	}
}
```

Notes:
Add GET parameter to SET command, for more powerful GETSET (#7852)

---
### String

GETEX -> getAndExpire

```java
@Component
class GetEx implements Function<KeyValue, String> {
	private final StringRedisTemplate redisTemplate;
	public GetEx(StringRedisTemplate redisTemplate){
		this.redisTemplate = redisTemplate;
	}
	@Override
	public String apply(KeyValue input) {
		Duration t = Duration.ofSeconds(Long.parseLong(input.getValue()));
		return redisTemplate
				.opsForValue()
				.getAndExpire(input.getKey(), t);
	}
}
```

---

GETDEL -> getAndDelete

```java
@Component
class GetDel implements Function<String, String> {
	private final StringRedisTemplate redisTemplate;
	public GetDel(StringRedisTemplate redisTemplate){
		this.redisTemplate = redisTemplate;
	}
	@Override
	public String apply(String input) {
		return redisTemplate
				.opsForValue()
				.getAndDelete(input);
	}
}
```

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
