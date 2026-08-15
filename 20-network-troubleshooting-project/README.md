# Part 20: Network Troubleshooting — Real-World Project

## Table of Contents

- [1. A General Troubleshooting Workflow](#1-a-general-troubleshooting-workflow)
- [2. Scenario: "My API Can't Connect to PostgreSQL"](#2-scenario-my-api-cant-connect-to-postgresql)
- [3. Scenario: "It Works Locally but Not on the Server"](#3-scenario-it-works-locally-but-not-on-the-server)
- [4. Scenario: "Connection Refused" vs. "Connection Timed Out"](#4-scenario-connection-refused-vs-connection-timed-out)
- [5. Scenario: "DNS Is Not Resolving"](#5-scenario-dns-is-not-resolving)
- [6. Scenario: "Port Is Not Accessible from Outside"](#6-scenario-port-is-not-accessible-from-outside)
- [7. Quick Reference Checklist](#7-quick-reference-checklist)

---

## 1. A General Troubleshooting Workflow

![Work outward from the app, one layer at a time: DNS, then port, then app response, then correctness](./assets/troubleshooting-flow.svg)

Every "it doesn't work" report can be broken into the same four questions, checked in order — this is the workflow the rest of this part walks through with real commands:

1. **Can I resolve the name?** — does the hostname point to an IP at all?
2. **Can I reach the port?** — is something listening, and is the network path open?
3. **Does the app respond?** — is the process alive and answering requests?
4. **Is the response correct?** — right status code, right data?

Each layer uses the tools from [Part 13](../13-network-tools-for-developers): `dig`/`nslookup`, `nc`/`telnet`, and `curl`.

## 2. Scenario: "My API Can't Connect to PostgreSQL"

You deploy the [Part 17](../17-docker-networking) Compose setup, and the API logs show:

```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

Walk it through:

```bash
docker compose ps
```

```
NAME      STATUS
api       Up
db        Up
```

Both containers are running — so the problem isn't that the database crashed. The real clue is `127.0.0.1` in the error.

```bash
docker exec -it api env | grep DATABASE_URL
```

```
DATABASE_URL=postgres://user:pass@localhost:5432/mydb
```

**Found it.** Inside a container, `localhost` means *the container itself* ([Part 17, Section 5](../17-docker-networking#5-localhost-means-something-different-inside-docker)) — not the `db` container. The fix is changing the connection string to use the service name:

```diff
- DATABASE_URL=postgres://user:pass@localhost:5432/mydb
+ DATABASE_URL=postgres://user:pass@db:5432/mydb
```

## 3. Scenario: "It Works Locally but Not on the Server"

The app runs fine with `npm run dev`, but on the deployed server, requests to the API hang and eventually time out.

```bash
curl -v https://api.example.com/health
```

```
* Trying 203.0.113.10:443...
* connect() timed out!
curl: (28) Failed to connect after 21034 ms
```

A hang followed by a timeout — not an instant refusal — almost always means a firewall or security group is silently dropping the packets before they reach the server ([Part 18, Section 6](../18-cloud-networking-basics#6-how-a-developer-connects-to-a-cloud-server)).

```bash
# on the server itself
sudo ufw status
```

```
Status: active
80/tcp     ALLOW    Anywhere
```

Port 443 is missing from the allowed list — the app runs identically in both places, but the server's firewall was never opened for HTTPS. Adding the rule fixes it:

```bash
sudo ufw allow 443/tcp
```

## 4. Scenario: "Connection Refused" vs. "Connection Timed Out"

These two errors point to different problems, and knowing which one you're looking at saves a lot of guessing (recap of [Part 14](../14-client-server-communication)):

| Error | What it means | Where to look |
|---|---|---|
| `Connection refused` | Something answered and said "no" — nothing is listening on that port | Is the app process actually running? Is it bound to the right port? |
| `Connection timed out` | Nothing answered at all | Firewall, security group, or routing dropping the packets silently |

```bash
nc -zv 203.0.113.10 3000
```

```
nc: connect to 203.0.113.10 port 3000 (tcp) failed: Connection refused
```

`Connection refused` here means the OS itself is rejecting the connection — check that the process is actually running and listening:

```bash
ss -tlnp | grep 3000
```

If nothing is returned, the app either crashed or is bound to `127.0.0.1` instead of `0.0.0.0`, so it only accepts connections from inside the server itself.

## 5. Scenario: "DNS Is Not Resolving"

A teammate reports `api.example.com` isn't loading — but the server is confirmed up.

```bash
dig api.example.com
```

```
;; ANSWER SECTION:
;; (empty)
```

No answer section means the DNS record doesn't exist yet, or hasn't propagated ([Part 7](../07-dns)). Compare against a known-working record and check with a different resolver to rule out local caching:

```bash
dig api.example.com @8.8.8.8
nslookup api.example.com 1.1.1.1
```

If a public resolver like `8.8.8.8` also returns nothing, the problem is the DNS record itself — check the DNS provider's dashboard for a missing or misconfigured `A`/`CNAME` record, not the server.

## 6. Scenario: "Port Is Not Accessible from Outside"

The app works when you `curl localhost:3000` on the server directly, but not from your laptop.

```bash
# from your laptop
curl http://203.0.113.10:3000
# curl: (7) Failed to connect to 203.0.113.10 port 3000: Connection refused
```

This is the classic gap between "the app is running" and "the network lets you reach it." Check each layer that sits between your laptop and the process:

1. **Security group** ([Part 18](../18-cloud-networking-basics)) — is inbound 3000 allowed from your IP?
2. **OS firewall** ([Part 12](../12-firewall)) — is `ufw`/`iptables` also blocking it?
3. **Docker port mapping** ([Part 17](../17-docker-networking)) — was the container started with `-p 3000:3000`, or only exposed internally?
4. **Bind address** — is the app listening on `0.0.0.0`, or only `127.0.0.1` (which refuses everything except local connections)?

Fixing whichever layer is closing the door — most often the security group or a missing `-p` flag — resolves it.

## 7. Quick Reference Checklist

```
[ ] Does the hostname resolve?        dig <host> / nslookup <host>
[ ] Is the port reachable?            nc -zv <host> <port>
[ ] Is the app actually listening?    ss -tlnp | grep <port>
[ ] Is it bound to 0.0.0.0, not 127.0.0.1?
[ ] Does the firewall allow the port? ufw status / security group rules
[ ] Does the app respond over HTTP?   curl -v <url>
[ ] Are container/service names used instead of "localhost"?
```

That's the full series — from "what is a network" in [Part 1](../01-what-is-a-network) to debugging one end to end here in Part 20. Every scenario above is really just the same four-question workflow from [Section 1](#1-a-general-troubleshooting-workflow) applied to a real error message.
