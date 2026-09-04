**Routing table showing the rogue /25 prefix**

```
docker exec clab-spine-leaf-bgp-leaf2 sr_cli -c "show network-instance default route-table ipv4-unicast summary"
```

```
| 10.20.4.0/24               | 0     | bgp        | bgp_mgr              | True     | default  | 0       | 170        | 10.10.1.2/31 (i | ethernet-1/49.0 |                 |                          |
|                            |       |            |                      |          |          |         |            | ndirect/local)  | ethernet-1/50.0 |                 |                          |
|                            |       |            |                      |          |          |         |            | 10.10.2.2/31 (i |                 |                 |                          |
|                            |       |            |                      |          |          |         |            | ndirect/local)  |                 |                 |                          |
| 10.20.4.0/25               | 0     | bgp        | bgp_mgr              | True     | default  | 0       | 170        | 10.10.1.2/31 (i | ethernet-1/49.0 |                 |                          |
|                            |       |            |                      |          |          |         |            | ndirect/local)  |                 |                 |                          |
```

leaf2 carries both prefixes: the legit `10.20.4.0/24` (ECMP, two next-hops -- via spine1 and spine2, points at leaf4) and the rogue `10.20.4.0/25` (single path, only via spine1 -- because only leaf1 originates it, and only spine1 happens to be leaf2's active path to leaf1 here). No ECMP on the /25 since only one AS (leaf1) is advertising it.

Traceroute confirms the hijack takes effect -- traffic dies at leaf1, never reaches leaf4:

```
docker exec clab-spine-leaf-bgp-host2 traceroute -n -w 2 10.20.4.2

traceroute to 10.20.4.2 (10.20.4.2), 30 hops max, 60 byte packets
 1  10.20.2.1  0.995 ms  0.972 ms  0.973 ms
 2  10.10.1.2  1.230 ms  1.261 ms  1.272 ms
 3  10.10.1.1  0.852 ms  0.885 ms  0.903 ms
 4  * * *
 5  * * *
 ...(dies, no more hops respond)
```

Hop 3 (`10.10.1.1`) is leaf1's fabric-facing interface -- traffic gets pulled there instead of continuing to leaf4, then dropped into the `nhg-blackhole` next-hop group with `installed true` (see [[exercise-4-implementation-bug]] for why the naive unresolvable-IP nexthop approach silently fails to install and doesn't reproduce this symptom).

---

**Explanation: why a /25 wins over a /24 (longest-prefix-match)**

Every router's forwarding decision picks the *most specific* matching route, not the first one it learned, not the "correct" one, not the one with better AS-path. Specificity == mask length. Full stop.

`10.20.4.2` (host4) falls inside both prefixes:
- `10.20.4.0/24` -- covers `10.20.4.0` -- `10.20.4.255`, 256 addresses, legitimately originated by leaf4.
- `10.20.4.0/25` -- covers `10.20.4.0` -- `10.20.4.127`, 128 addresses, rogue, originated by leaf1.

`10.20.4.2` sits in the first half of the /24, so it matches *both* prefixes. When two prefixes both match a destination, longest-prefix-match (LPM) says: pick the one with the longer (more specific) mask -- /25 beats /24 because it narrows the candidate address space further, so it's treated as the more precise answer of "where does this address actually live."

This has nothing to do with:
- BGP local-preference, AS-path length, or any other path-selection attribute -- those tiebreakers only apply *among routes for the same prefix*. A /25 and a /24 are different prefixes, so BGP best-path selection never even runs a comparison between them.
- Which route was learned first or is "more trustworthy" -- BGP has no notion of prefix legitimacy. It trusts whoever originates a route within policy.
- Route preference/admin-distance -- both are learned via the same protocol (BGP, preference 170 here), so that tiebreaker doesn't apply either.

LPM is a pure forwarding-plane rule, applied per-packet at the FIB, independent of *how* each route got there. That's exactly what makes prefix hijacking dangerous: the attacker doesn't need to win any BGP tiebreak or metric fight. Announcing *any* more-specific prefix automatically wins for every address inside it, no matter how good the legitimate route's BGP attributes are. This is precisely the mechanism behind real-world incidents like Pakistan Telecom's 2008 hijack of YouTube's block via a bogus /24 (more specific than YouTube's advertised space at the time).

Why this exercise's export-hijack policy on leaf1 could get away with it: default-deny export policy meant leaf1 had to *explicitly* build a policy to push out the /25 (see the `hijack` prefix-set, `mask-length-range exact` matching only /25). Nothing in BGP itself checks "do you actually own this address space" -- that's an operational/RPKI-layer safeguard, absent here by design, which is exactly what the exercise demonstrates.
