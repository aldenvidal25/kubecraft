**How many hops between any two hosts?**
 4 hops, always — matches your traceroute output exactly (4
  lines).

**How many equal-cost paths exist between any two leaves?**
2 paths — one via spine1, one via spine2. Directly tied to spine count: each spine gives exactly one leaf→spine→leaf path, so N spines = N equal-cost paths. Fabric has 2 spines → 2 ECMP paths, matches maximum-paths: 2 config and the traceroute alternation you saw.

**What happens to bandwidth if you add a third spine**
 +50% aggregate bandwidth, 3 ECMP paths.

---
 docker exec clab-spine-leaf-bgp-host1 traceroute -n -w 2 10.20.4.2
traceroute to 10.20.4.2 (10.20.4.2), 30 hops max, 60 byte packets
 1  10.20.1.1  0.838 ms  0.801 ms  0.791 ms
 2  10.10.1.0  1.422 ms  1.314 ms  1.307 ms
 3  10.10.1.7  1.175 ms 10.10.2.7  1.206 ms 10.10.1.7  1.178 ms
 4  10.20.4.2  0.312 ms  0.315 ms  0.311 ms

 docker exec clab-spine-leaf-bgp-host1 traceroute -n -w 2 10.20.4.2
traceroute to 10.20.4.2 (10.20.4.2), 30 hops max, 60 byte packets
 1  10.20.1.1  1.059 ms  1.030 ms  1.020 ms
 2  10.10.1.0  1.452 ms 10.10.2.0  1.486 ms 10.10.1.0  1.439 ms
 3  10.10.2.7  1.554 ms 10.10.1.7  1.509 ms 10.10.2.7  1.550 ms
 4  10.20.4.2  2.057 ms  2.116 ms  2.128 ms

 docker exec clab-spine-leaf-bgp-host1 traceroute -n -w 2 10.20.4.2
traceroute to 10.20.4.2 (10.20.4.2), 30 hops max, 60 byte packets
 1  10.20.1.1  0.854 ms  0.848 ms  0.850 ms
 2  10.10.1.0  1.360 ms 10.10.2.0  1.311 ms 10.10.1.0  1.625 ms
 3  10.10.2.7  1.961 ms  2.025 ms 10.10.1.7  1.979 ms
 4  10.20.4.2  1.353 ms  1.318 ms  1.437 ms

#### Deliverables

**Routing table showing ECMP entries**
-----------------------------------------------------------------------------
IPv4 unicast route table of network instance default
-----------------------------------------------------------------------------
+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| Pre | ID  | Rou | Rou | Act | Ori | Met | Pre | Nex | Nex | Bac | Bac |
| fix |     | te  | te  | ive | gin | ric |  f  | t-  | t-  | kup | kup |
|     |     | Typ | Own |     | Net |     |     | hop | hop | Nex | Nex |
|     |     |  e  | er  |     | wor |     |     | (Ty | Int | t-  | t-  |
|     |     |     |     |     | k I |     |     | pe) | erf | hop | hop |
|     |     |     |     |     | nst |     |     |     | ace | (Ty | Int |
|     |     |     |     |     | anc |     |     |     |     | pe) | erf |
|     |     |     |     |     |  e  |     |     |     |     |     | ace |
+=====+=====+=====+=====+=====+=====+=====+=====+=====+=====+=====+=====+
| 10. | 3   | loc | net | Tru | def | 0   | 0   | 10. | eth |     |     |
| 10. |     | al  | _in | e   | aul |     |     | 10. | ern |     |     |
| 1.0 |     |     | st_ |     | t   |     |     | 1.1 | et- |     |     |
| /31 |     |     | mgr |     |     |     |     | (di | 1/4 |     |     |
|     |     |     |     |     |     |     |     | rec | 9.0 |     |     |
|     |     |     |     |     |     |     |     | t)  |     |     |     |
| 10. | 3   | hos | net | Tru | def | 0   | 0   | Non | Non |     |     |
| 10. |     | t   | _in | e   | aul |     |     | e ( | e   |     |     |
| 1.1 |     |     | st_ |     | t   |     |     | ext |     |     |     |
| /32 |     |     | mgr |     |     |     |     | rac |     |     |     |
|     |     |     |     |     |     |     |     | t)  |     |     |     |
| 10. | 4   | loc | net | Tru | def | 0   | 0   | 10. | eth |     |     |
| 10. |     | al  | _in | e   | aul |     |     | 10. | ern |     |     |
| 2.0 |     |     | st_ |     | t   |     |     | 2.1 | et- |     |     |
| /31 |     |     | mgr |     |     |     |     | (di | 1/5 |     |     |
|     |     |     |     |     |     |     |     | rec | 0.0 |     |     |
|     |     |     |     |     |     |     |     | t)  |     |     |     |
| 10. | 4   | hos | net | Tru | def | 0   | 0   | Non | Non |     |     |
| 10. |     | t   | _in | e   | aul |     |     | e ( | e   |     |     |
| 2.1 |     |     | st_ |     | t   |     |     | ext |     |     |     |
| /32 |     |     | mgr |     |     |     |     | rac |     |     |     |
|     |     |     |     |     |     |     |     | t)  |     |     |     |
| 10. | 2   | loc | net | Tru | def | 0   | 0   | 10. | eth |     |     |
| 20. |     | al  | _in | e   | aul |     |     | 20. | ern |     |     |
| 1.0 |     |     | st_ |     | t   |     |     | 1.1 | et- |     |     |
| /24 |     |     | mgr |     |     |     |     | (di | 1/1 |     |     |
|     |     |     |     |     |     |     |     | rec | .0  |     |     |
|     |     |     |     |     |     |     |     | t)  |     |     |     |
| 10. | 2   | hos | net | Tru | def | 0   | 0   | Non | Non |     |     |
| 20. |     | t   | _in | e   | aul |     |     | e ( | e   |     |     |
| 1.1 |     |     | st_ |     | t   |     |     | ext |     |     |     |
| /32 |     |     | mgr |     |     |     |     | rac |     |     |     |
|     |     |     |     |     |     |     |     | t)  |     |     |     |
| 10. | 2   | hos | net | Tru | def | 0   | 0   | Non |     |     |     |
| 20. |     | t   | _in | e   | aul |     |     | e ( |     |     |     |
| 1.2 |     |     | st_ |     | t   |     |     | bro |     |     |     |
| 55/ |     |     | mgr |     |     |     |     | adc |     |     |     |
| 32  |     |     |     |     |     |     |     | ast |     |     |     |
|     |     |     |     |     |     |     |     | )   |     |     |     |
| 10. | 0   | bgp | bgp | Tru | def | 0   | 170 | 10. | eth |     |     |
| 20. |     |     | _mg | e   | aul |     |     | 10. | ern |     |     |
| 2.0 |     |     | r   |     | t   |     |     | 1.0 | et- |     |     |
| /24 |     |     |     |     |     |     |     | /31 | 1/4 |     |     |
|     |     |     |     |     |     |     |     | (in | 9.0 |     |     |
|     |     |     |     |     |     |     |     | dir | eth |     |     |
|     |     |     |     |     |     |     |     | ect | ern |     |     |
|     |     |     |     |     |     |     |     | /lo | et- |     |     |
|     |     |     |     |     |     |     |     | cal | 1/5 |     |     |
|     |     |     |     |     |     |     |     | )   | 0.0 |     |     |
|     |     |     |     |     |     |     |     | 10. |     |     |     |
|     |     |     |     |     |     |     |     | 10. |     |     |     |
|     |     |     |     |     |     |     |     | 2.0 |     |     |     |
|     |     |     |     |     |     |     |     | /31 |     |     |     |
|     |     |     |     |     |     |     |     | (in |     |     |     |
|     |     |     |     |     |     |     |     | dir |     |     |     |
|     |     |     |     |     |     |     |     | ect |     |     |     |
|     |     |     |     |     |     |     |     | /lo |     |     |     |
|     |     |     |     |     |     |     |     | cal |     |     |     |
|     |     |     |     |     |     |     |     | )   |     |     |     |
| 10. | 0   | bgp | bgp | Tru | def | 0   | 170 | 10. | eth |     |     |
| 20. |     |     | _mg | e   | aul |     |     | 10. | ern |     |     |
| 3.0 |     |     | r   |     | t   |     |     | 1.0 | et- |     |     |
| /24 |     |     |     |     |     |     |     | /31 | 1/4 |     |     |
|     |     |     |     |     |     |     |     | (in | 9.0 |     |     |
|     |     |     |     |     |     |     |     | dir | eth |     |     |
|     |     |     |     |     |     |     |     | ect | ern |     |     |
|     |     |     |     |     |     |     |     | /lo | et- |     |     |
|     |     |     |     |     |     |     |     | cal | 1/5 |     |     |
|     |     |     |     |     |     |     |     | )   | 0.0 |     |     |
|     |     |     |     |     |     |     |     | 10. |     |     |     |
|     |     |     |     |     |     |     |     | 10. |     |     |     |
|     |     |     |     |     |     |     |     | 2.0 |     |     |     |
|     |     |     |     |     |     |     |     | /31 |     |     |     |
|     |     |     |     |     |     |     |     | (in |     |     |     |
|     |     |     |     |     |     |     |     | dir |     |     |     |
|     |     |     |     |     |     |     |     | ect |     |     |     |
|     |     |     |     |     |     |     |     | /lo |     |     |     |
|     |     |     |     |     |     |     |     | cal |     |     |     |
|     |     |     |     |     |     |     |     | )   |     |     |     |
| 10. | 0   | bgp | bgp | Tru | def | 0   | 170 | 10. | eth |     |     |
| 20. |     |     | _mg | e   | aul |     |     | 10. | ern |     |     |
| 4.0 |     |     | r   |     | t   |     |     | 1.0 | et- |     |     |
| /24 |     |     |     |     |     |     |     | /31 | 1/4 |     |     |
|     |     |     |     |     |     |     |     | (in | 9.0 |     |     |
|     |     |     |     |     |     |     |     | dir | eth |     |     |
|     |     |     |     |     |     |     |     | ect | ern |     |     |
|     |     |     |     |     |     |     |     | /lo | et- |     |     |
|     |     |     |     |     |     |     |     | cal | 1/5 |     |     |
|     |     |     |     |     |     |     |     | )   | 0.0 |     |     |
|     |     |     |     |     |     |     |     | 10. |     |     |     |
|     |     |     |     |     |     |     |     | 10. |     |     |     |
|     |     |     |     |     |     |     |     | 2.0 |     |     |     |
|     |     |     |     |     |     |     |     | /31 |     |     |     |
|     |     |     |     |     |     |     |     | (in |     |     |     |
|     |     |     |     |     |     |     |     | dir |     |     |     |
|     |     |     |     |     |     |     |     | ect |     |     |     |
|     |     |     |     |     |     |     |     | /lo |     |     |     |
|     |     |     |     |     |     |     |     | cal |     |     |     |
|     |     |     |     |     |     |     |     | )   |     |     |     |
+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
-----------------------------------------------------------------------------
IPv4 routes total                    : 10
IPv4 prefixes with active routes     : 10
IPv4 prefixes with active ECMP routes: 3

