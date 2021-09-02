### Set

- Collections of unique, unsorted string elements.
- Set Operations (Union/Intersection/Subtraction)
- Most operations in constant time (`O(n)`)

---

### Set Use Cases

- Unique item management (tags/folksonomies)
- Tracking IPs, content filtering
- As a support data structure to manage membership
  - SDR maintains Primary Keys for mapped classes in a Redis Set

---

### Working with Sets

- Add/Remove: `SADD` / `SPOP` / `SREM`
- Access/Retrieve: `SMEMBERS` / `SRANDMEMBERS` / `SSCAN`
- Set Info: `SCARD` / `SISMEMBER` / `SMISMEMBER`
- Set Ops: `SDIFF*`/  `SINTER*` / `SUNION*` / `SMOVE`

---
### An Example

Playing with sets:

```bash
redis> SADD colors "red" "yellow" "green" "fushia"
redis> SADD colors "yellow"
redis> SMEMBERS colors
redis> SISMEMBER colors "green"
redis> SISMEMBER colors "magenta"
redis> SREM colors "green"
redis> SREM colors "green"
redis> SMEMBERS colors
```

---
### Sets in Spring Data Redis

```java
@Test
void testSimpleExample() {
  setOps.add("colors", "red", "yellow", "green", "fushia");
  setOps.add("colors", "yellow");
  Set<String> members = setOps.members("colors");
  assertTrue(members.containsAll(List.of("red", "yellow", "green", "fushia")));
  assertTrue(setOps.isMember("colors", "green"));
  assertFalse(setOps.isMember("colors", "magenta"));
  assertEquals(1, setOps.remove("colors", "green"));
  members = setOps.members("colors");
  assertTrue(members.containsAll(List.of("red", "yellow", "fushia")));
}
```

---
### What's new with Sets in 6.2?

- Add `SMISMEMBER` command that checks multiple members (#7615)

---
###  Multiple members check w/ `SMISMEMBER`

```bash
redis> SADD colors "red" "yellow" "green" "fushia"
redis> SMISMEMBER colors "red" "black" "green"
1) (integer) 1
2) (integer) 0
3) (integer) 1
```

---

### `SMISMEMBER` on SDR as a JUnit Test

```java
@Test
void testSMISMEMBER() {
  setOps.add("colors", "red", "yellow", "green", "fushia");
  Map<Object, Boolean> memberCheck = setOps.isMember("colors", "red", "black", "green");
  assertTrue(memberCheck.get("red"));
  assertFalse(memberCheck.get("black"));
  assertTrue(memberCheck.get("green"));
}
```


Notes:
- Brian

