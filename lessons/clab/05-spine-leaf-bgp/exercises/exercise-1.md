**Routing policies: import-all, export-connected, and export-bgp -- why does a default-deny NOS like SR Linux need all three?**

 Default-deny baseline: SR Linux drops all routes on a BGP session, both directions, unless policy explicitly permits. No policy attached =
  neighbor Established but zero routes exchanged.

  Three policies, three separate jobs:

  1. import-all (inbound direction) — without it, leaf1 would establish sessions to both spines but discard every route the spines send.
     Accept-all default since leaf trusts spine-advertised routes unconditionally (spines only carry fabric-legit host subnets).
  2. export-connected (outbound, catches local routes) — leaf1's own host subnet (10.20.1.0/24) isn't a BGP route, it's a directly-connected
     interface route (protocol local). BGP export policy has to explicitly opt that in or it never gets originated into BGP at all — connected
     routes aren't auto-redistributed. Filtered further by the host-subnets prefix-set so only the /24 gets in, not the /31 fabric links.
  3. export-bgp (outbound, catches BGP-learned routes) — separate protocol type (protocol bgp) from protocol local, so needs its own match
     statement. Re-advertises routes leaf1 learned from one spine back out — relevant for spines passing leaf routes to other leaves, and
     structurally needed even on leaves for consistency/symmetry.

  Why can't 1 policy do it all: import and export are opposite directions — separate policy attachments (import-policy vs export-policy) by
  protocol design, can't merge. And within export, "local" and "bgp" are distinct protocol match values in SR Linux's policy match syntax —
  one statement's match.protocol takes one value, so covering both source types needs two statements (here modeled as two chained policies via
  default-action: next-policy, letting export-connected fall through to export-bgp when a route isn't locally-connected).

  Bottom line: default-deny means "no policy = no routes," so every direction and every route-source (connected vs BGP-learned) needs its own
  explicit accept, else the fabric forms sessions but carries no traffic.

**Prefix-set filter: host-subnets on export-connected -- what does 10.20.0.0/16 mask-length-range 24..24 match, and what does it exclude?**

Syntax: ip-prefix = base range to check against.
  mask-length-range = which prefix lengths within that range
  count as a match.

  10.20.0.0/16 mask-length-range 24..24 means: prefix must fall
  inside 10.20.0.0/16 AND have exactly a /24 mask. 24..24 =
  lower bound 24, upper bound 24 — single allowed length, not a
  range spanning multiple lengths (compare to something like
  24..32 which'd match /24 through /32).

  What matches: 10.20.1.0/24, 10.20.2.0/24, 10.20.3.0/24,
  10.20.4.0/24 — exactly the four host subnets (leaf↔host links
  from lab.clab.yml).

  What's excluded:
  - The /31 spine↔leaf fabric links (10.10.1.0/31, 10.10.2.0/31,
    etc.) — wrong base range (10.10.0.0 not 10.20.0.0) and
    wrong mask length (/31 ≠ /24), fails on both counts.
  - Any hypothetical /25 or /26 sub-carved-up piece of
    10.20.x.0/24 — would fall inside 10.20.0.0/16 but fail
    mask-length-range since it's not exactly /24. (This is
    exactly the mechanism Exercise 4's hijack abuses on a
    different policy — leaf1's rogue export-hijack policy
    doesn't use this prefix-set at all, it's a separate
    hand-rolled hijack prefix-set matching /25 exact.)

  Why it matters: combined with export-connected's protocol
  local match, this is the second gate that keeps only host
  subnets in BGP. Without the mask-length-range pinned to
  exactly 24, a looser match (say mask-length-range 24..32)
  would also let more-specific /25+ leaks or misconfigs slip
  into the fabric — locking to exact /24 is a defensive
  constraint, not just a filter. Ties directly to Exercise 2
  step 4 (verifying no /31s show in BGP) and Exercise 4 (the /25
  hijack succeeds precisely because that attack uses its own
  separate policy, bypassing this constraint entirely).

**Multipath: maximum-paths: 2 under afi-safi ipv4-unicast -- what is the SR Linux default, and why does ECMP require changing it?**

SR Linux default: maximum-paths 1 — single best-path only, standard BGP behavior (best-path selection algorithm picks one winner, installs
  only that one).

  Why ECMP needs it changed: In this fabric, leaf1 gets the same route (e.g. 10.20.4.0/24, leaf4's subnet) from both spine1 and spine2 — two
  BGP paths, equally good. Standard best-path selection runs its tiebreak algorithm (AS-path length, origin, MED, etc.) down to some final
  tiebreaker (like lowest router-id or peer address) and picks exactly one winner even when both paths are functionally equal cost. With
  maximum-paths 1, leaf1 installs only spine1's path (say) and silently drops spine2's — one spine sits idle, no load-sharing, and losing that
  one active spine means full traffic disruption until reconvergence.

  maximum-paths 2 tells BGP: when N paths tie as "equally good" through best-path selection (up to the point where they're indistinguishable —
  same AS-path length, etc.), install all N as active ECMP next-hops instead of forcing a single winner. That's what makes leaf1 program both
  spine1 and spine2 as next-hops for 10.20.4.0/24 — traffic hashed across both, using full fabric bandwidth, and losing one spine just drops
  the ECMP set to 1 path instead of killing the route entirely (Exercise 3's whole point).

  Why 2 specifically: matches spine count — 2 spines, so at most 2 equal paths ever exist. Setting it higher wouldn't matter (nothing to fill
  the extra slots); setting it to 1 (the default) is what causes Exercise 2's "single spine, no ECMP" broken state the lab has you contrast
  against.


**Shared spine ASN: Why do both spine neighbors have the same peer-as (65000)? What does RFC 7938 say about spine ASN assignment?**

Config fact: both spine1 and spine2 configured with autonomous-system: 65000, and every leaf's neighbor entries for both spines use peer-as:
  65000 — same AS number for two physically distinct devices.

  Why this is unusual: normally in eBGP every router gets a unique AS (that's what makes it "external" BGP — peers exchange routes across AS
  boundaries, AS-path grows by one hop per AS traversed). Here, spine1 and spine2 are separate boxes but share one ASN — deliberate design,
  not a mistake.

  RFC 7938 (Use of BGP for Routing in Large-Scale Data Centers) rationale:
  - Recommends each leaf gets a unique private ASN, but all spines in the same tier can share one ASN (or even: all devices in a tier share an
    ASN, tier by tier).
  - Reasoning: AS-path loop prevention. eBGP's core loop-detection mechanism is "reject a route if your own ASN already appears in its
    AS-path." If spine1 and spine2 had different ASNs, a route leaf1 sends to spine1 could legally propagate spine1→leaf2→spine2→leaf1 without
    ever seeing a repeated ASN in the path — a real risk in denser fabrics (more spines, more leaves) where multiple transit paths exist.
  - By giving both spines the same ASN, any route that loops back through a second spine now carries a duplicate ASN in its AS-path (65000
    appears twice) — eBGP's own loop-detection kills it automatically. Free protection, no extra policy needed.
  - Bonus: it also lets you scale spine count without ASN exhaustion or renumbering — add spine3, spine4, all AS 65000, no new ASN needed per
    spine, and loop-prevention property holds regardless of fabric width.

  Why leaves stay unique (65001-65004): leaves are where routes originate (each leaf's own host subnet) — need unique ASN per leaf so AS-path
  is meaningful for path selection between different leaf-originated routes and multi-hop loop protection still functions leaf-to-leaf via
  spine transit.

  Net: shared spine ASN isn't a "them and us" quirk — it's the standard Clos-fabric BGP loop-prevention pattern from RFC 7938, purpose-built
  so eBGP's normal AS-path checking substitutes for what iBGP would otherwise need route-reflectors or split-horizon rules to solve.

#### Deliverables

 docker exec -it clab-spine-leaf-bgp-spine1 sr_cli -c "show network-instance default protocols bgp neighbor"
----------------------------------------------------------------------------------------------------------------------------------------------
BGP neighbor summary for network-instance "default"
Flags: S static, D dynamic, L discovered by LLDP, B BFD enabled, - disabled, * slow
----------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------
+----------------+-----------------------+----------------+------+---------+-------------+-------------+-----------+-----------------------+
|    Net-Inst    |         Peer          |     Group      | Flag | Peer-AS |    State    |   Uptime    | AFI/SAFI  |    [Rx/Active/Tx]     |
|                |                       |                |  s   |         |             |             |           |                       |
+================+=======================+================+======+=========+=============+=============+===========+=======================+
| default        | 10.10.1.1             | leaves         | S    | 65001   | established | 0d:0h:36m:2 | ipv4-     | [1/1/3]               |
|                |                       |                |      |         |             | 8s          | unicast   |                       |
| default        | 10.10.1.3             | leaves         | S    | 65002   | established | 0d:0h:36m:7 | ipv4-     | [1/1/3]               |
|                |                       |                |      |         |             | s           | unicast   |                       |
| default        | 10.10.1.5             | leaves         | S    | 65003   | established | 0d:0h:36m:1 | ipv4-     | [1/1/3]               |
|                |                       |                |      |         |             | s           | unicast   |                       |
| default        | 10.10.1.7             | leaves         | S    | 65004   | established | 0d:0h:35m:5 | ipv4-     | [1/1/3]               |
|                |                       |                |      |         |             | 4s          | unicast   |                       |
+----------------+-----------------------+----------------+------+---------+-------------+-------------+-----------+-----------------------+
----------------------------------------------------------------------------------------------------------------------------------------------
Summary:
4 configured neighbors, 4 configured sessions are established, 0 disabled peers
0 dynamic peers

---
 docker exec -it clab-spine-leaf-bgp-spine1 sr_cli -c "show network-instance default protocols bgp neighbor"                                 
-----------------------------------------------------------------------------
BGP neighbor summary for network-instance "default"
Flags: S static, D dynamic, L discovered by LLDP, B BFD enabled, - disabled, * slow
-----------------------------------------------------------------------------
-----------------------------------------------------------------------------
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
| Net-  | Peer  | Group | Flags | Peer- | State | Uptim | AFI/S | [Rx/A |
| Inst  |       |       |       |  AS   |       |   e   |  AFI  | ctive |
|       |       |       |       |       |       |       |       | /Tx]  |
+=======+=======+=======+=======+=======+=======+=======+=======+=======+
| defau | 10.10 | leave | S     | 65001 | estab | 0d:0h | ipv4- | [1/1/ |
| lt    | .1.1  | s     |       |       | lishe | :37m: | unica | 3]    |
|       |       |       |       |       | d     | 56s   | st    |       |
| defau | 10.10 | leave | S     | 65002 | estab | 0d:0h | ipv4- | [1/1/ |
| lt    | .1.3  | s     |       |       | lishe | :37m: | unica | 3]    |
|       |       |       |       |       | d     | 34s   | st    |       |
| defau | 10.10 | leave | S     | 65003 | estab | 0d:0h | ipv4- | [1/1/ |
| lt    | .1.5  | s     |       |       | lishe | :37m: | unica | 3]    |
|       |       |       |       |       | d     | 29s   | st    |       |
| defau | 10.10 | leave | S     | 65004 | estab | 0d:0h | ipv4- | [1/1/ |
| lt    | .1.7  | s     |       |       | lishe | :37m: | unica | 3]    |
|       |       |       |       |       | d     | 22s   | st    |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
-----------------------------------------------------------------------------
Summary:
4 configured neighbors, 4 configured sessions are established, 0 disabled peers
0 dynamic peers
---


