# RunStack MCP — external infrastructure checks for AI agents

Remote MCP server that checks your public infrastructure **from the outside** — the way users, browsers and other mail servers see it — and returns a **diagnosis**: verdict, findings with cause and fix, and a shareable report link.

Nothing runs locally and nothing runs on your servers: only public data is checked.

- Endpoint: `https://runstack.pro/api/mcp` (Streamable HTTP, stateless)
- Auth: header `X-RunStack-Key: rsk_...` (or `Authorization: Bearer rsk_...`)
- Free key: https://runstack.pro/account — 200 calls/day
- Registry name: `pro.runstack/checks`

## Tools

| Tool | What it checks |
|---|---|
| `check_tls` | Certificate validity and chain, expiry, hostname, TLS 1.0–1.3, ciphers, HSTS |
| `check_http` | Status, redirect chain, security headers (HSTS, CSP, X-Frame-Options…), caching, compression |
| `check_email_domain` | MX, SPF (10-lookup limit), DKIM, DMARC, PTR/FCrDNS, STARTTLS, MTA-STS, TLS-RPT, BIMI |
| `check_dns` | Answers from several public resolvers compared — propagation, NXDOMAIN, inconsistencies |
| `check_port` | TCP ports open / closed / filtered from the internet |
| `ip_lookup` | IP or ASN owner, prefix, PTR, RPKI, geolocation |

All tools are read-only.

## Example (abridged, real output)

> "Check the mail setup of runstack.ru"

```
FAIL: Mail domain runstack.ru: critical issues
- MX: 10 mxs1.reg.ru, 20 mxs2.reg.ru
- SPF: ~all · lookups 4/10
- DMARC: p=none
- DKIM: dkim

Findings:
- [error] Multiple DMARC records
  why: _dmarc.runstack.ru has 2 v=DMARC1 records — receivers then ignore DMARC entirely. Keep one.
  how-to: https://runstack.ru/docs/fix-dmarc-p-none
- [info] SPF ~all (softfail)
  why: Soft fail — common during monitoring.
- [info] MX certificate not valid for the server name
  why: STARTTLS works, but the MX certificate doesn't match the name. Matters for MTA-STS and DANE.
```

Yes, that is our own domain — the check found it.

## Setup

### Claude Code

```bash
claude mcp add --transport http runstack https://runstack.pro/api/mcp --header "X-RunStack-Key: rsk_..."
```

### Cursor (`~/.cursor/mcp.json`) / VS Code / other clients

```json
{
  "mcpServers": {
    "runstack": {
      "url": "https://runstack.pro/api/mcp",
      "headers": { "X-RunStack-Key": "rsk_..." }
    }
  }
}
```

### Codex (`~/.codex/config.toml`)

```toml
[mcp_servers.runstack]
url = "https://runstack.pro/api/mcp"
http_headers = { "X-RunStack-Key" = "rsk_..." }
```

## Language

Diagnoses are in English on runstack.pro. Russian: `https://runstack.ru/api/mcp` or `?lang=ru`.

## Links

- Website: https://runstack.pro
- REST API: https://runstack.pro/developers
- Telegram bot: https://t.me/RunStackru_bot
