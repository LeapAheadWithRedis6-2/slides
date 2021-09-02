### Sorted Set

Sorted sets, similar to Sets but where every string element is associated to a floating number value, called score. The elements are always taken sorted by their score, so unlike Sets it is possible to retrieve a range of elements (for example you may ask: give me the top 10, or the bottom 10).

### Sorted Set

- Add ZMSCORE command that returns an array of scores (#7593)
- Add ZDIFF and ZDIFFSTORE commands (#7961)
- Add ZINTER and ZUNION commands (#7794)
- Add ZRANDMEMBER command (#8297)
- Add the REV, BYLEX and BYSCORE arguments to ZRANGE, and the ZRANGESTORE command (#7844)

Notes:
- Brian


Let's look at an example where we have two sorted sets: game1 and game2, containing players and their scores in two different games.

ZADD game1 100 Frank 740 Jennifer 200 Pieter 512 Dave 690 Ana
ZADD game2 212 Joe 230 Jennifer 450 Mary 730 Tom 512 Dave 200 Frank


Step 2
Find the players that are playing both games (the elements that are present in both sets) and show their summed up scores.

ZINTER 2 game1 game2 WITHSCORES
Note: The argument 2 indicates how many keys are we doing the operation on.

Repeat the same query, showing their maximum score in either of the games.

ZINTER 2 game1 game2 WITHSCORES AGGREGATE max
Note the AGGREGATE argument; it defines how do we want to aggregate the scores of the results of the intersection. The default option is SUM, so when you only use WITHSCORES, you will get the sum of the scores of an element across both sorted sets. If you use WITHSCORES AGGREGATE MIN, or WITHSCORES AGGREGATE MAX, you will get the minimum, or maximum score across both sorted sets.

The WITHSCORES and AGGREGATE options are possible for all commands that act on multiple sorted sets: ZINTER, ZINTERSTORE, ZUNION, ZUNIONSTORE, ZDIFF and ZDIFFSTORE.

Step 3
Find all the players that are playing game 1, but not game 2 (members present in sorted set 1, but not sorted set 2) and show their scores.

ZDIFF 2 game1 game2 WITHSCORES
Store the result in a key named only_game_1:

ZDIFFSTORE only_game_1 2 game1 game2
Confirm the creation of the new key:

ZRANGE only_game_1 0 -1 WITHSCORES


Step 4
Show all the players, across both games and their summed scores:

ZUNION 2 game1 game2 WITHSCORES

Step 5
Redis 6.2.0 introduced the REV, BYLEX and BYSCORE arguments to the ZRANGE command, replacing the commands: ZREVRANGE, ZRANGEBYSCORE, ZREVRANGEBYSCORE, ZRANGEBYLEX and ZREVRANGEBYLEX.

So instead of using the ZREVRANGE command, now you would use the ZRANGE command, with the REV argument:

ZRANGE game1 0 -1 WITHSCORES        # Ascending order
ZRANGE game1 0 -1 REV WITHSCORES    # Descending order
And instead of using the ZRANGEBYSCORE command, we now use the ZRANGE command with the BYSCORE argument:

ZRANGE game1 100 512 BYSCORE