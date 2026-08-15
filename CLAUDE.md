# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repository holds the content for **"Basic Networking for Developers"**, a YouTube tutorial series that teaches the networking concepts developers actually run into while building and deploying applications — not network engineering theory. The repo is currently empty of source files; this document is the plan future work should follow as episodes, notes, scripts, diagrams, or demo code are added.

## Audience and framing

Target audience is beginner-to-intermediate developers, not network engineers. Every part should be framed around a developer's real workflow (writing an API, deploying an app, debugging a connection issue) rather than abstract networking theory. When adding content, prefer practical, hands-on examples (curl commands, Docker Compose files, real error messages) over protocol-level deep dives.

## Series structure

The series is organized into 20 parts, grouped into 3 levels:

**Level 1 — Networking Fundamentals** (Parts 1–8): what a network is, IP addresses, MAC addresses, ports, TCP vs UDP, HTTP/HTTPS, DNS, OSI & TCP/IP models.

**Level 2 — Practical Networking for Developers** (Parts 9–15): subnetting/CIDR, routing, NAT, firewalls, CLI networking tools, client-server communication, reverse proxies.

**Level 3 — Networking for DevOps & Backend Developers** (Parts 16–20): load balancers, Docker networking, cloud networking (VPC/security groups/gateways), network security basics, and a real-world troubleshooting project.

Full per-part topic breakdown:

1. **What Is a Network?** — LAN/WAN/Internet, client-server, network devices, "what happens when you open a website"
2. **IP Address** — IPv4/IPv6, public vs private, localhost, special addresses
3. **MAC Address** — MAC vs IP, ARP basics
4. **Ports** — common ports (80, 443, 22, 53, 5432, 3306), port conflicts
5. **TCP vs UDP** — 3-way handshake, when to use which
6. **HTTP & HTTPS** — request/response, methods, status codes, headers, TLS
7. **DNS** — resolution flow, record types (A, AAAA, CNAME, MX, TXT), caching
8. **OSI & TCP/IP Models** — layers, how a web request travels through them
9. **Subnet & CIDR Basics** — subnet masks, CIDR notation, relevance to cloud/Docker
10. **Routing** — router vs gateway, routing tables, `route`/`ip route`
11. **NAT** — private-to-public IP, port forwarding
12. **Firewall** — inbound/outbound, `ufw`, common connection problems
13. **Network Tools for Developers** — `ping`, `curl`, `wget`, `traceroute`, `nslookup`, `dig`, `netstat`, `ss`, `ip`, `telnet`/`nc`
14. **Client-Server Communication** — frontend/backend/database communication, timeouts, connection refused/reset
15. **Reverse Proxy** — Nginx basics, SSL termination, domain → proxy → app
16. **Load Balancer** — Layer 4 vs Layer 7, health checks, multi-backend architecture
17. **Docker Networking** — bridge network, container-to-container, port mapping, Compose, backend + PostgreSQL example
18. **Cloud Networking Basics** — VPC, subnets, security groups, internet/NAT gateways, example AWS architecture
19. **Network Security Basics** — TLS/HTTPS, SSH, firewalls, security groups, VPN, not exposing databases publicly
20. **Network Troubleshooting — Real-World Project** — step-by-step debugging of API-to-DB, local-vs-server, DNS, and port issues

## Content conventions

- Each part should stay scoped to its numbered topic above — don't bleed Level 3 (cloud/DevOps) concepts into Level 1 fundamentals content.
- Practical, runnable examples (shell commands, Docker Compose snippets, curl calls) are preferred over prose-only explanations.
- Parts 13, 17, and 20 in particular are meant to be hands-on/demo-driven rather than conceptual — prioritize working commands and reproducible setups over slide-style explanation.
