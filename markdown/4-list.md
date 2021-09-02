### List

- An ordered sequence of strings
- Comparable to a Java ArrayList
- Multi-purpose:
  - Stacks
  - Queues
- A single List can hold over 4 billion entries.

---

### List Use Cases

- Most Recently added elements (RSS feeds, posts, updates, news, etc.)
- Consumer-Producer pattern - Redis (lists and sets accessor can block/wait for date)
- Anything that resembles a Queue or a Stack!

---
### Adding to a List

- You can **add** things by:
  - **Pushing** in: `LPUSH`/`LPUSHX`, `RPUSH`/`RPUSHX`
  - **Inserting** before/after: `LINSERT`
  - **Setting** the value at an **index**: `LSET`

---
### Remove from a List

- You can **remove things** things by:
  - **Popping** off: `LPOP`/`RPOP`/`BLPOP`/`BRPOP`
  - **By value** `LREM`
  - **By index** range: `LTRIM`

---
### Accessing Elements

- **By index**: `LINDEX`
- **By range**: `LRANGE`

---
### Between Lists

- **Last** from one list, **to first** in another: `RPOPLPUSH`/`BRPOPLPUSH`
- **Pop** and then **Push**: `LMOVE`/`BLMOVE`

---
### An Example

- Creating and manipulating a list of funny words, in the Redis CLI:

```bash
redis> RPUSH funny_words "Shenanigans" "Bamboozle" "Bodacious"
redis> LRANGE funny_words 0 -1
redis> LPUSH funny_words "Bumfuzzle"
redis> LRANGE funny_words 0 -1
redis> LRANGE funny_words 1 3
redis> LINSERT funny_words BEFORE "Bamboozle" "Brouhaha"
redis> LSET funny_words -2 "Flibbertigibbet"
redis> LRANGE funny_words 0 -1
redis> LPOP funny_words
redis> LRANGE funny_words 0 -1
```

Notes:

CLI Transcript:

```
127.0.0.1:6379> RPUSH funny_words "Shenanigans" "Bamboozle" "Bodacious"
(integer) 3
127.0.0.1:6379> LRANGE funny_words 0 -1
1) "Shenanigans"
2) "Bamboozle"
3) "Bodacious"
127.0.0.1:6379> LPUSH funny_words "Bumfuzzle"
(integer) 4
127.0.0.1:6379> LRANGE funny_words 0 -1
1) "Bumfuzzle"
2) "Shenanigans"
3) "Bamboozle"
4) "Bodacious"
127.0.0.1:6379> LRANGE funny_words 1 3
1) "Shenanigans"
2) "Bamboozle"
3) "Bodacious"
127.0.0.1:6379> LINSERT funny_words BEFORE "Bamboozle" "Brouhaha"
(integer) 5
127.0.0.1:6379> LSET funny_words -2 "Flibbertigibbet"
OK
127.0.0.1:6379> LRANGE funny_words 0 -1
1) "Bumfuzzle"
2) "Shenanigans"
3) "Brouhaha"
4) "Flibbertigibbet"
5) "Bodacious"
127.0.0.1:6379> LPOP funny_words
"Bumfuzzle"
127.0.0.1:6379> LRANGE funny_words 0 -1
1) "Shenanigans"
2) "Brouhaha"
3) "Flibbertigibbet"
4) "Bodacious"
127.0.0.1:6379>
```

---
### In Spring Data Redis

```java
public class ListExample {

  @Autowired
  private StringRedisTemplate redisTemplate;

  @Resource(name="stringRedisTemplate")
  private ListOperations<String, String> listOps;

  public void playWithLists() {
    //...
  }
}
```

---
### In Spring Data Redis

```java
public void playWithLists() {
  listOps.rightPushAll("funny_words", "Shenanigans", "Bamboozle", "Bodacious");
  List<String> range = listOps.range("funny_words", 0, -1);
  System.out.println(range.toArray());

  listOps.leftPush("funny_words", "Bumfuzzle");
  range = listOps.range("funny_words", 1, 3);

  listOps.leftPush("funny_words", "Bamboozle", "Brouhaha");
  // ...

  listOps.set("funny_words", -2, "Flibbertigibbet");
  // ...
  System.out.println(listOps.size("funny_words"));
}
```

---
### What's new with Lists in 6.2?

- Add `LMOVE` and `BLMOVE` commands that pop and push arbitrarily (#6929)
- Add the `COUNT `argument to `LPOP` and `RPOP` (#8179)

---

### `LMOVE` on the CLI

- Right-most from `list_one` to the left of `list_two`
- Left-most from `list_one` to the left of `list_two`

```bash
redis> RPUSH list_one "one" "two" "three"
redis> LMOVE list_one list_two RIGHT LEFT
redis> LMOVE list_one list_two LEFT RIGHT
redis> LRANGE list_one 0 -1
1) "two"
redis> LRANGE list_two 0 -1
1) "three"
2) "one"
```

---

### `LMOVE` on SDR as a JUnit Test

```java
@Test
void testLMOVE() {
  listOps.rightPushAll("list_one", "one", "two", "three");
  listOps.move("list_one", RIGHT, "list_two", LEFT);
  listOps.move("list_one", LEFT, "list_two", RIGHT);

  List<String> listOne = listOps.range("list_one", 0, -1);
  List<String> listTwo = listOps.range("list_two", 0, -1);

  assertTrue(listOne.containsAll(List.of("two")));
  assertTrue(listTwo.containsAll(List.of("three", "one")));
}
```

---

### `COUNT` on `LPOP`/`RPOP`

Pop "n" things from the left or the right

```bash
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LPOP mylist 2
"one"
redis> LRANGE mylist 0 -1
1) "three"
```

---

### `COUNT` on `LPOP`/`RPOP` on SDR as a JUnit Test

```java
@Test
void testLPOP() {
  listOps.rightPush("mylist", "one");
  listOps.rightPush("mylist", "two");
  listOps.rightPush("mylist", "three");
  listOps.leftPop("mylist", 2);
  List<String> myList = listOps.range("mylist", 0, -1);
  assertTrue(myList.containsAll(List.of("three")));
}
```

Notes:
- Brian