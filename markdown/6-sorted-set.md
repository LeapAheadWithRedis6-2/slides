### Sorted Set

- A weighted Sets: A mix between a `Set` and a `Hash`
- Elements
  - are tuples with a **value** and a **score**
  - are always taken sorted by their score
  - can be retrieved in ranges

---
### Sorted Set Use Cases

- Priority queues
- Low-latency leaderboards
- Secondary indexing in general

---
### Working with Sorted Sets

```bash
redis> ZADD game1 100 "Frank" 740 "Jennifer" 200 "Pieter" 512 "Dave" 690 "Ana"
redis> ZADD game2 212 "Joe" 230 "Jennifer" 450 "Mary" 730 "Tom" 512 "Dave" 200 "Frank"

redis> ZRANGE game1 0 -1
redis> ZRANGE game2 0 -1 WITHSCORES

redis> ZINTER 2 game1 game2 WITHSCORES
redis> ZINTER 2 game1 game2 WITHSCORES AGGREGATE max

redis> ZDIFF 2 game1 game2 WITHSCORES
```

Notes:
- ZINTER 2 game1 game2 WITHSCORES ==> "playing both games"
- ZINTER 2 game1 game2 WITHSCORES AGGREGATE max ==> "playing both games find max score"
- ZDIFF 2 game1 game2 WITHSCORES ==> "playing only game 1"

---
### `ZADD` in SDR

```java
Set<TypedTuple<String>> game1 = Set.of( //
    TypedTuple.of("Frank", 100.0), TypedTuple.of("Jennifer", 740.0),
    TypedTuple.of("Pieter", 200.0), TypedTuple.of("Dave", 512.0),
    TypedTuple.of("Ana", 690.0));

Set<TypedTuple<String>> game2 = Set.of( //
    TypedTuple.of("Joe", 212.0), TypedTuple.of("Jennifer", 230.0),
    TypedTuple.of("Mary", 450.0), TypedTuple.of("Tom", 730.0),
    TypedTuple.of("Dave", 512.0), TypedTuple.of("Frank", 200.0));

zSetOps.add("game1", game1);
zSetOps.add("game2", game2);
```

---
### `ZRANGE` in SDR

```java
Set<String> game1Players = zSetOps.range("game1", 0, -1);
assertArrayEquals(new String[] { "Frank", "Pieter", "Dave", "Ana", "Jennifer"}, game1Players.toArray());

Set<TypedTuple<String>> game2PlayersWithScores = zSetOps.rangeWithScores("game2", 0, -1);
TypedTuple<String> frankInGame2 = game2PlayersWithScores.iterator().next();
assertEquals("Frank", frankInGame2.getValue());
assertEquals(200.0, frankInGame2.getScore());
```

---
### `ZINTER` in SDR

```java
Set<TypedTuple<String>> inBothGames = zSetOps.intersectWithScores("game1", "game2");
TypedTuple<String> frankInBothGamesTotal = inBothGames.iterator().next();
assertEquals("Frank", frankInBothGamesTotal.getValue());
assertEquals(300.0, frankInBothGamesTotal.getScore());

Set<TypedTuple<String>> inBothGamesWithMax = zSetOps.intersectWithScores("game1", Set.of("game2"), Aggregate.MAX);
TypedTuple<String> frankInBothGamesMax = inBothGamesWithMax.iterator().next();
assertEquals("Frank", frankInBothGamesMax.getValue());
assertEquals(200.0, frankInBothGamesMax.getScore());
```

---
### `ZDIFF` in SDR

```java
Set<TypedTuple<String>> onlyInGame1 = zSetOps.differenceWithScores("game1", "game2");
List<String> players = onlyInGame1.stream().map(t -> t.getValue()).collect(Collectors.toList());
assertTrue(players.containsAll(Set.of("Pieter", "Ana")));
```

---
### Sorted Set

- Add ZMSCORE command that returns an array of scores (#7593)
- Add ZDIFF and ZDIFFSTORE commands (#7961)
- Add ZINTER and ZUNION commands (#7794)
- Add ZRANDMEMBER command (#8297)
- Add the REV, BYLEX and BYSCORE arguments to ZRANGE, and the ZRANGESTORE command (#7844)

---
### ZMSCORE w/ an array of scores

```bash
redis> ZADD myzset 1 "one"
redis> ZADD myzset 2 "two"
redis> ZMSCORE myzset "one" "two" "nofield"
1) "1"
2) "2"
3) (nil)
```

---
### ZMSCORE w/ an array of scores

```java
@Test
void testZMSCORE() {
  zSetOps.add("myzset", "one", 1);
  zSetOps.add("myzset", "two", 2);
  List<Double> scores = zSetOps.score("myzset", "one", "two", "nofield");

  assertArrayEquals(new Double[] { 1.0, 2.0, null }, scores.toArray());
}
```

---
### ZDIFF commands

```bash
redis> ZADD zset1 1 "one"
redis> ZADD zset1 2 "two"
redis> ZADD zset1 3 "three"
redis> ZADD zset2 1 "one"
redis> ZADD zset2 2 "two"
redis> ZDIFF 2 zset1 zset2
1) "three"
redis> ZDIFF 2 zset1 zset2 WITHSCORES
1) "three"
2) "3"
```

---
### ZDIFF commands

```java
@Test
void testZDIFF() {
  zSetOps.add("zset1", "one", 1);
  zSetOps.add("zset1", "two", 2);
  zSetOps.add("zset1", "three", 3);
  zSetOps.add("zset2", "one", 1);
  zSetOps.add("zset2", "two", 2);

  Set<String> diffs = zSetOps.difference("zset1", "zset2");
  assertArrayEquals(new String[] { "three" }, diffs.toArray());

  Set<TypedTuple<String>> diffsWScores = zSetOps.differenceWithScores("zset1", "zset2");
  assertEquals(1, diffsWScores.size());
  TypedTuple<String> dtt = diffsWScores.iterator().next();
  assertEquals("three", dtt.getValue());
  assertEquals(3.0, dtt.getScore());
}
```

---
### ZDIFFSTORE commands

```bash
redis> ZADD zset1 1 "one"
redis> ZADD zset1 2 "two"
redis> ZADD zset1 3 "three"
redis> ZADD zset2 1 "one"
redis> ZADD zset2 2 "two"
redis> ZDIFFSTORE out 2 zset1 zset2
redis> ZRANGE out 0 -1 WITHSCORES
1) "three"
2) "3"
```

---
### ZDIFFSTORE commands

```java
@Test
void testZDIFFSTORE() {
  zSetOps.add("zset1", "one", 1);
  zSetOps.add("zset1", "two", 2);
  zSetOps.add("zset1", "three", 3);
  zSetOps.add("zset2", "one", 1);
  zSetOps.add("zset2", "two", 2);

  zSetOps.differenceAndStore("zset1", List.of("zset2"), "out");
  Set<TypedTuple<String>> withScores = zSetOps.rangeWithScores("out", 0, -1);
  assertEquals(1, withScores.size());
  TypedTuple<String> dtt = withScores.iterator().next();
  assertEquals("three", dtt.getValue());
  assertEquals(3.0, dtt.getScore());
}
```

Notes:
- Brian
