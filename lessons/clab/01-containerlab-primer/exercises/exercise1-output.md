# Deliverables

Hostname             : srl1
Chassis Type         : 7220 IXR-D2L
Part Number          : Sim Part No.
Serial Number        : Sim Serial No.
System HW MAC Address: 1A:07:00:FF:00:00
OS                   : SR Linux
Software Version     : v24.10.1
Build Number         : 492-gf8858c5836
Architecture         : x86_64
Last Booted          : 2026-08-26T02:05:28.619Z
Total Memory         : 15697203 kB
Free Memory          : 6181457 kB

---

Hostname             : srl2
Chassis Type         : 7220 IXR-D2L
Part Number          : Sim Part No.
Serial Number        : Sim Serial No.
System HW MAC Address: 1A:28:01:FF:00:00
OS                   : SR Linux
Software Version     : v24.10.1
Build Number         : 492-gf8858c5836
Architecture         : x86_64
Last Booted          : 2026-08-26T02:05:28.804Z
Total Memory         : 15697203 kB
Free Memory          : 6746066 kB

---

Hostname             : srl3
Chassis Type         : 7220 IXR-D2L
Part Number          : Sim Part No.
Serial Number        : Sim Serial No.
System HW MAC Address: 1A:6A:02:FF:00:00
OS                   : SR Linux
Software Version     : v24.10.1
Build Number         : 492-gf8858c5836
Architecture         : x86_64
Last Booted          : 2026-08-26T02:05:28.733Z
Total Memory         : 15697203 kB
Free Memory          : 6321214 kB

---

+---------------------+---------+---------+---------+---------+---------+
|        Port         |  Admin  |  Oper   |  Speed  |  Type   | Descrip |
|                     |  State  |  State  |         |         |  tion   |
+=====================+=========+=========+=========+=========+=========+
| ethernet-1/1        | enable  | up      | 25G     |         |         |
| ethernet-1/2        | disable | down    | 25G     |         |         |
| ethernet-1/3        | disable | down    | 25G     |         |         |
| ethernet-1/4        | disable | down    | 25G     |         |         |
| ethernet-1/5        | disable | down    | 25G     |         |         |
| ethernet-1/6        | disable | down    | 25G     |         |         |
| ethernet-1/7        | disable | down    | 25G     |         |         |
| ethernet-1/8        | disable | down    | 25G     |         |         |
| ethernet-1/9        | disable | down    | 25G     |         |         |
| ethernet-1/10       | disable | down    | 25G     |         |         |
| ethernet-1/11       | disable | down    | 25G     |         |         |
| ethernet-1/12       | disable | down    | 25G     |         |         |
| ethernet-1/13       | disable | down    | 25G     |         |         |
| ethernet-1/14       | disable | down    | 25G     |         |         |
| ethernet-1/15       | disable | down    | 25G     |         |         |
| ethernet-1/16       | disable | down    | 25G     |         |         |
| ethernet-1/17       | disable | down    | 25G     |         |         |
| ethernet-1/18       | disable | down    | 25G     |         |         |
| ethernet-1/19       | disable | down    | 25G     |         |         |
| ethernet-1/20       | disable | down    | 25G     |         |         |
| ethernet-1/21       | disable | down    | 25G     |         |         |
| ethernet-1/22       | disable | down    | 25G     |         |         |
| ethernet-1/23       | disable | down    | 25G     |         |         |
| ethernet-1/24       | disable | down    | 25G     |         |         |
| ethernet-1/25       | disable | down    | 25G     |         |         |
| ethernet-1/26       | disable | down    | 25G     |         |         |
| ethernet-1/27       | disable | down    | 25G     |         |         |
| ethernet-1/28       | disable | down    | 25G     |         |         |
| ethernet-1/29       | disable | down    | 25G     |         |         |
| ethernet-1/30       | disable | down    | 25G     |         |         |
| ethernet-1/31       | disable | down    | 25G     |         |         |
| ethernet-1/32       | disable | down    | 25G     |         |         |
| ethernet-1/33       | disable | down    | 25G     |         |         |
| ethernet-1/34       | disable | down    | 25G     |         |         |
| ethernet-1/35       | disable | down    | 25G     |         |         |
| ethernet-1/36       | disable | down    | 25G     |         |         |
| ethernet-1/37       | disable | down    | 25G     |         |         |
| ethernet-1/38       | disable | down    | 25G     |         |         |
| ethernet-1/39       | disable | down    | 25G     |         |         |
| ethernet-1/40       | disable | down    | 25G     |         |         |
| ethernet-1/41       | disable | down    | 25G     |         |         |
| ethernet-1/42       | disable | down    | 25G     |         |         |
| ethernet-1/43       | disable | down    | 25G     |         |         |
| ethernet-1/44       | disable | down    | 25G     |         |         |
| ethernet-1/45       | disable | down    | 25G     |         |         |
| ethernet-1/46       | disable | down    | 25G     |         |         |
| ethernet-1/47       | disable | down    | 25G     |         |         |
| ethernet-1/48       | disable | down    | 25G     |         |         |
| ethernet-1/49       | disable | down    | 100G    |         |         |
| ethernet-1/50       | disable | down    | 100G    |         |         |
| ethernet-1/51       | disable | down    | 100G    |         |         |
| ethernet-1/52       | disable | down    | 100G    |         |         |
| ethernet-1/53       | disable | down    | 100G    |         |         |
| ethernet-1/54       | disable | down    | 100G    |         |         |
| ethernet-1/55       | disable | down    | 100G    |         |         |
| ethernet-1/56       | disable | down    | 100G    |         |         |
| ethernet-1/57       | disable | down    | 10G     |         |         |
| ethernet-1/58       | disable | down    | 10G     |         |         |
| mgmt0               | enable  | up      | 1G      |         |         |
+---------------------+---------+---------+---------+---------+---------+
e1-1 up cuz link define connect it: srl1:e1-1 <-> srl2:e1-1. Both ends wired, both ends up.

Other ethernet ports (e1-2 on srl1, e1-3+ etc) down cuz no link entry for them — unconnected iface, no carrier, stay down. Only e1-1 (both nodes) and e1-2 on srl2 (linked to srl3) show up. Rest idle.
