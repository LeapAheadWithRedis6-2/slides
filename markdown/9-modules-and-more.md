### What we're up to at Redis

- Extending/complementing Spring Data Redis with:
  - Access to module commands via Spring's Templates
  - Multi-model Object-Mapping support
  - JSON object-mapping + RediSearch integration
  - Graph object-mapping
  - RediSearch integration for existing Redis Hash mapped entities

---
### What we're up to at Redis

- Plus...
  - Encapsulated "Use Cases" that can be applies declaratively
  - Graph object-mapping
  - RediSearch integration for existing Redis Hash mapped entities
  - Encapsulated "Use Cases" that can be applies declaratively

---

### Redis Modules Templates

- Follow's Spring Data Redis "`opsForXXX()`" pattern
- Provide's a Spring Native way to interact at the command-level with:
- RedisJSON, RedisGraph, RediSearch, RedisAI, RedisBloom, and RedisTimeSeries

---

### Redis Modules Templates

Inject a `RedisModulesOperations` bean:

```java
@Autowired
RedisModulesOperations<String, String> modulesOperations;
```

Retrieve a module template to use its commands:

```java
GraphOperations<String> graph = modulesOperations.opsForGraph();
// Create both source and destination nodes
graph("social", "CREATE (:person{name:'roi',age:32})");
graph("social", "CREATE (:person{name:'amit',age:30})");
```

---

### RedisJSON Document-Object Mapping

Annotate a Java class with `@Document` annotation:

```java
@Document("company")
public class Company {
  @Id
  private String id;
  @NonNull
  private String name;
  private boolean publiclyListed;
}
```
---

### RedisJSON Document-Object Mapping

Repository support via `RedisDocumentRepository` interface:

```java
interface CompanyRepository extends RedisDocumentRepository<Company, String> {}
```

---

### RedisJSON Document-Object Mapping

In action:

```java
@Autowired CompanyRepository repository;

Company redislabs = repository.save(Company.of("RedisLabs"));
Company microsoft = repository.save(Company.of("Microsoft"));

System.out.println(">>>> Items in repo: " + repository.count());

Optional<Company> maybeRedisLabs = repository.findById(redislabs.getId());
Optional<Company> maybeMicrosoft = repository.findById(microsoft.getId());

System.out.println(">>>> Company: " + maybeRedisLabs.get());
```

---

### RedisJSON/RediSearch Integration

- `@XXXIndexed` Annotations for RediSearch index creation and maintenance

```java
@Document("my-doc")
public class MyDoc {
  @Id
  private String id;
  @NonNull
  @TextIndexed(alias = "title")
  private String title;
  @TagIndexed(alias = "tag")
  private Set<String> tag = new HashSet<String>();
}
```

---
### Search anything...

- `@Query` and `@Aggregation` for powerful native RediSearch Queries and Aggregations:

```java
public interface MyDocRepository
  extends RedisDocumentRepository<MyDoc, String>, MyDocQueries {

  @Query(returnFields = {"$.tag[0]", "AS", "first_tag"})
  SearchResult getFirstTag();

  @Query("@title:$title @tag:{$tags}")
  Iterable<MyDoc> findByTitleAndTags(@Param("title") String title, @Param("tags") Set<String> tags);

  @Aggregation(load = {"$.tag[1]", "AS", "tag2"})
  AggregationResult getSecondTagWithAggregation();
}
```