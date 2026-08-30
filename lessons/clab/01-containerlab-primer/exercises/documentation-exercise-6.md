
`docker ps -a`
`docker ps -a --filter name=clab-first-lab`

docker ps default show only running containers. Stopped container = "exited" state, filtered out.

-a (--all) flag show containers in every state — running, exited, created, paused. Without it, stopped container basically invisible, look like it don't exist even tho still there (just not running) — container object persist til explicit rm.


