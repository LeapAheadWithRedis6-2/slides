### Geo

Review:

GEORADIUS cities -9.1604 38.7452 20 km
GEORADIUSBYMEMBER cities Lisbon 20 km

- Add GEOSEARCH/GEOSEARCHSTORE commands for bounding box spatial queries (#8094)

GEOSEARCH cities FROMLONLAT -9.1604 38.7452 BYRADIUS 20 km
GEOSEARCH cities FROMMEMBER Lisbon BYRADIUS 20 km

Notes:
- This is a consistency of commands update
- Don't remember the commands, use SpringDataRedis

- Add GT and LT options to ZADD for conditional score updates (#7818)
- Add the ANY argument to GEOSEARCH and GEORADIUS (#8259)
- Add the CH, NX, XX arguments to GEOADD (#8227)

Notes:
- DaShaun