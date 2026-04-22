# Reverse Proxy System
### A High-Performance Web Application Firewall & Reverse Proxy in Go

> A cybersecurity-focused reverse proxy server with built-in WAF capabilities, intelligent rate limiting, and a live admin dashboard — designed to sit in front of your services and block malicious traffic before it reaches your application.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [How It Works](#how-it-works)
- [Configuration](#configuration)
- [WAF Rules](#waf-rules)
- [Rate Limiting](#rate-limiting)
- [Admin Dashboard](#admin-dashboard)
- [Test Results](#test-results)
- [Technologies Used](#technologies-used)
- [Future Scope](#future-scope)

---

## Overview

SecureProxy is a lightweight, production-ready **reverse proxy with an embedded Web Application Firewall (WAF)** built entirely in Go. It inspects every incoming HTTP request before forwarding it to the target service — detecting and blocking SQL injection, XSS attacks, rate abuse, and blacklisted IPs in real time.

All behavior is driven by a `config.json` file that can be **reloaded at runtime** without restarting the server, making it suitable for dynamic environments.

---

## Features

| Feature | Description |
|---|---|
| 🔀 Dynamic Reverse Proxy | Route traffic to backend services via `config.json` |
| 🚦 Sliding Window Rate Limiting | Per-endpoint, per-IP request throttling |
| 🛡️ WAF — SQL Injection Detection | Detects and blocks SQLi patterns in requests |
| 🛡️ WAF — XSS Detection | Detects and blocks Cross-Site Scripting payloads |
| 🚫 IP Blacklisting | Instantly block known malicious IP addresses |
| 🔄 Dynamic Config Reload | Update rules and routes without server restart |
| 📊 Live Admin Dashboard | Real-time traffic and threat monitoring at `/dashboard` |

---

## How It Works

Every incoming request passes through a multi-layer inspection pipeline before reaching the backend:

```
Incoming Request
      ↓
┌─────────────────────┐
│   IP Blacklist Check │  → Block if IP is blacklisted
└────────┬────────────┘
         ↓
┌─────────────────────┐
│  Rate Limit Check    │  → Block if requests exceed threshold (sliding window)
└────────┬────────────┘
         ↓
┌─────────────────────┐
│  WAF — SQLi Check    │  → Block if SQL injection pattern detected
└────────┬────────────┘
         ↓
┌─────────────────────┐
│  WAF — XSS Check     │  → Block if XSS payload detected
└────────┬────────────┘
         ↓
┌─────────────────────┐
│   Reverse Proxy      │  → Forward clean request to backend
└─────────────────────┘
```

Blocked requests receive an appropriate HTTP error response (403/429) and are logged for dashboard display.

---

## Configuration

All routing rules, rate limits, and blacklists are managed through `config.json`:

```json
{
  "routes": [
    {
      "path": "/api/users",
      "target": "http://localhost:8081",
      "rate_limit": {
        "requests": 100,
        "window_seconds": 60
      }
    },
    {
      "path": "/api/orders",
      "target": "http://localhost:8082",
      "rate_limit": {
        "requests": 50,
        "window_seconds": 30
      }
    }
  ],
  "blacklisted_ips": [
    "192.168.1.105",
    "10.0.0.23"
  ]
}
```

To apply changes, trigger a config reload via the admin dashboard or send a reload signal — **no server restart required**.

---

## WAF Rules

### SQL Injection Detection
The WAF scans request URLs, query parameters, and body payloads for common SQLi signatures, including:

- `' OR '1'='1` style authentication bypass
- `UNION SELECT` data extraction attempts
- `DROP TABLE`, `INSERT`, `DELETE` statement injections
- Comment-based obfuscation (`--`, `#`, `/*`)

### XSS Detection
Incoming data is scanned for Cross-Site Scripting patterns, including:

- `<script>` tag injections
- Event handler attributes (`onerror=`, `onload=`, `onclick=`)
- JavaScript URI schemes (`javascript:`)
- Encoded payload variants

All matched requests are **blocked immediately** and the offending IP is flagged in the dashboard.

---

## Rate Limiting

SecureProxy uses a **Sliding Window** algorithm for rate limiting — more accurate than fixed windows as it smooths out burst traffic at window boundaries.

- Limits are configured **per endpoint** and **per IP address**
- When a limit is breached, the IP receives a `429 Too Many Requests` response
- The window resets smoothly as old requests age out
- Repeat offenders can be automatically promoted to the blacklist

---

## Admin Dashboard

Access the live dashboard at:

```
http://localhost:<port>/dashboard
```

The dashboard provides real-time visibility into:

- 📈 Total requests processed
- 🚫 Blocked requests (WAF + rate limit + blacklist)
- 🌐 Per-IP traffic breakdown
- ⚠️ Active threats and flagged IPs
- 🔄 Config reload trigger

---

## Test Results

The proxy was tested in a local network environment with real traffic from multiple devices.

| Test Scenario | Result |
|---|---|
| High-volume legitimate traffic (lakhs of requests) | ✅ All forwarded successfully |
| SQL injection payloads | ✅ Detected and blocked |
| XSS payloads | ✅ Detected and blocked |
| Rate limit breach (per IP, per endpoint) | ✅ Throttled with 429 response |
| Blacklisted IP requests | ✅ Blocked at entry |
| Config reload without restart | ✅ Applied instantly |
| Multi-device IP-based isolation | ✅ Each IP tracked and limited independently |

> Lakhs of requests were routed through the proxy during testing. Malicious and over-limit requests were classified and blocked accurately, while legitimate traffic passed through without disruption.

---

## Technologies Used

| Category | Technology |
|---|---|
| Language | Go (Golang) |
| Proxy Layer | `net/http` + `httputil.ReverseProxy` |
| WAF Engine | Custom pattern matching (Go) |
| Rate Limiter | Sliding Window algorithm (in-memory) |
| Config Management | JSON (`config.json`) with hot reload |
| Dashboard | Go HTTP server + live frontend |
| Testing | Custom endpoints on local network devices |

---

## Future Scope

- 🤖 ML-based anomaly detection for zero-day attack patterns
- 📜 Detailed request logging with export (CSV / JSON)
- 🔐 TLS/HTTPS termination support
- 📬 Webhook alerts on threat detection
- 🐳 Docker support for containerized deployment
- ☁️ Integration with threat intelligence feeds (IP reputation APIs)

---

## Conclusion

SecureProxy demonstrates how a purpose-built Go application can serve as an effective first line of defense for web services. With WAF capabilities, intelligent rate limiting, dynamic configuration, and a live dashboard — it provides meaningful protection against common web attacks while remaining lightweight and easy to configure.

---

> ⚠️ **Disclaimer:** This project is built for educational and research purposes in the cybersecurity domain. Always complement application-layer security with network-level defenses in production environments.
