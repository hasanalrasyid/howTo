Server Bridge
=============

Scenario
--------

Server T:
* behind a firewall, local network intraT.
* there is ssh-server in T, serving on port 22.

machineT:
* a local service (maybe web), only available for intraT

Server L:
* "outside" server.
* there is ssh-server in L, serving on port 443 (https).

machineL:
* can ssh to serverL




Aim
---

1. Bidirectional access:
   * intraT -> T -> L -> theWorld
   * theWorld -> L -> T -> intraT

Overview (tl;dr)
----------------

1. Create reverse forwarding to serverT from serverL using ssh.
2. Create a SOCKS proxy using connection of (1), this would fulfill (2) and (3).

Detailed Steps (in orderly fashion)
-----

### at serverT

We need forward our ssh service in serverT:22 to serverL.

```
ssh -p 443 serverL -R 2222:localhost:22 -D 8800
```

If we daemonize this command,
  then we can access serverT through serverL from the World anytime,
  otherwise we should have someone around serverT to run this command in our need.

By setting the proxy of our web-client to serverT:8800, then we can have a web-access through serverT -> serverL -> theWorld.

### at serverL

We can connect to serverT by

```
ssh localhost -p 2222
```

If we open port 2222 access in serverL
  then all theWorld can rightaway access serverT by invoking `ssh serverTIP -p 2222`.

### at machineL

Under assumption that port 2222 is closed from outside world, then we can:

```
ssh serverL -L 4444:localhost:2222 &
ssh localhost -p 4444 -D 8000
```

The last command would grant us a shell inside serverT and a SOCKS proxy connecting us through serverL -> serverT -> intraT
