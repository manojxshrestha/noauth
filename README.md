<h1 align="center">noauth</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.8+-blue" alt="Python">
  <img src="https://img.shields.io/badge/License-MIT-orange" alt="License">
  <img src="https://img.shields.io/badge/PRs-welcome-brightgreen" alt="PRs">
</p>

<p align="center">
  <b>Find unauthenticated web interfaces during authorized testing.</b>
</p>

noauth scans IP ranges, CIDRs, hostnames, and scope files to discover web interfaces exposed without authentication. It performs TCP port scans across 48 common web ports, probes 60+ admin/API/critical paths on each open port, fingerprints technologies against 40+ signatures, scores and classifies risk (Low→Critical), enriches results with reverse DNS, RDAP, and TLS SAN data, and exports findings in JSON, CSV, HTML, and Markdown — all with threaded concurrency, per-host timeouts, resume support, and optional Playwright screenshots.

## Features

- Single host, CIDR, hostname, or scope-file scanning
- Public-scope safety gate (`--allow-public`)
- 48 common web/admin ports or top-10 mode
- Deep path probing — 60+ admin, config, API, debug, and sensitive paths
- Multi-signal auth detection
- Technology fingerprinting via 40+ signatures
- Risk scoring with severity labels
- Reverse DNS / PTR enrichment
- RDAP network & org enrichment (opt-out via `--no-rdap`)
- TLS SAN domain discovery
- Associated domain & link extraction
- Secret-like content and stack trace detection
- Response body hashing
- Per-host timeout budget
- Resume mode with state file
- Optional Playwright screenshots
- Evidence folders per finding
- Team exports — JSON, CSV, HTML, Markdown
- Live triage feed
- IPv6 support

## Installation

```bash
git clone https://github.com/manojxshrestha/noauth.git
cd noauth
pip install requests
```

Optional (screenshots):

```bash
pip install playwright
python3 -m playwright install chromium
```

## Quick Start

```bash
# Scan a single IP
python3 noauth.py 192.168.1.100

# Scan a hostname (resolved automatically)
python3 noauth.py target.example.com

# Scan a subnet
python3 noauth.py 192.168.1.0/24

# Public IP (authorized scope only)
python3 noauth.py 134.209.128.139 --allow-public

# Top 10 ports only
python3 noauth.py 192.168.1.0/24 --top10

# Fast mode (skip deep probing)
python3 noauth.py 192.168.1.0/24 --fast

# Save evidence
python3 noauth.py 192.168.1.0/24 --output-dir output

# Capture screenshots
python3 noauth.py 192.168.1.0/24 --screenshots
```

## Scope Files

```bash
python3 noauth.py --scope-file scope.txt
```

Example `scope.txt`:

```
192.168.1.0/24
10.10.10.5
example.internal
```

Legacy `file:` format also supported:

```bash
python3 noauth.py file:ranges.txt
```

## Public Scope Safety

noauth refuses to scan public IPs unless you explicitly allow it:

```bash
python3 noauth.py 134.209.128.139
# → stops with a warning

python3 noauth.py 134.209.128.139 --allow-public
# → proceeds
```

## Advanced Usage

```bash
# CIDR with exclusions
python3 noauth.py 10.0.0.0/8 --exclude 10.10.0.0/16,10.20.0.0/16

# Randomized scan order
python3 noauth.py 10.0.0.0/16 --random

# Sample a large CIDR
python3 noauth.py 1.0.0.0/8 --allow-public --sample 50000 --random --top10

# Rate-limited
python3 noauth.py 1.0.0.0/8 --allow-public --sample 10000 --delay 50 --random

# Resume interrupted scan
python3 noauth.py 10.0.0.0/16 --resume --state scan_state.json

# Per-host timeout
python3 noauth.py 10.0.0.0/16 --host-timeout 20

# Skip RDAP (privacy)
python3 noauth.py 192.168.1.0/24 --no-rdap
```

## Output

```
noauth_results/
├── findings.json
├── findings.csv
├── findings.html
├── findings.md
├── evidence/
│   └── <host_port_severity_score>/
│       ├── finding.json
│       ├── headers.json
│       ├── body_preview.txt
│       └── links.txt
├── screenshots/
└── raw/
```

## Example Terminal Output

```
[3 no-auth] 134.209.128.139 — Apache2 Ubuntu Default Page: It works
Network: DIGITALOCEAN-134-209-0-0 ['134.209.0.0/16']
  Associated domains: 134.209.128.139, bugs.launchpad.net, ...
  ──────────────────────────────────────────────────────────────────────
  Low      score=15  ✔ :80 (200, 10671B) [Apache/2.4.52 (Ubuntu)] Apache
      title: Apache2 Ubuntu Default Page: It works
      reasons: public_ip

  Medium   score=25  ✔ :8080 (200, 90997B)
      title: Visual Schedule Builder - KCTCS
      final: http://134.209.128.139:8080/criteria.jsp
      reasons: public_ip, nonstandard_web_port
```

## Risk Scoring

| Severity | Score | Signal |
|---|---|---|
| Critical | 60+ | Secret-like content, dangerous paths, critical admin exposure |
| High | 40–59 | Sensitive paths, high-value tech, exposed API surfaces |
| Medium | 20–39 | Nonstandard port, admin language, useful metadata |
| Low | 0–19 | Public page or benign open web UI |

Score factors: `public_ip`, `nonstandard_web_port`, `sensitive_path_accessible`, `secret_like_content`, `stacktrace_detected`, `json_endpoint`, `high_value_admin_tech`, `admin_or_dashboard_language`, `metrics_exposed`.

## Enrichment

For each host, noauth collects reverse DNS / PTR, RDAP network name & CIDR (opt-out via `--no-rdap`), TLS certificate SAN domains, associated domains from page links, and redirect chains.

## Reports

```bash
python3 noauth.py 192.168.1.0/24 --output-format json
python3 noauth.py 192.168.1.0/24 --output-format csv
python3 noauth.py 192.168.1.0/24 --output-format html
python3 noauth.py 192.168.1.0/24 --output-format md
```

## All Options

| Option | Description |
|---|---|
| `target` | CIDR, IP, hostname, or `file:ranges.txt` |
| `--scope-file` | File with CIDRs/IPs/hostnames |
| `--allow-public` | Allow scanning public IP space |
| `--ports` | Ports: `web`, `top10`, or comma/space list |
| `--top10` | Shorthand for `--ports top10` |
| `--timeout N` | Request timeout in seconds (default 5) |
| `--host-timeout N` | Per-host timeout budget |
| `--threads N` | Thread count (default 50) |
| `--fast` | Skip deep path probing |
| `--random` | Randomize host scan order |
| `--sample N` | Sample N IPs from CIDR |
| `--delay MS` | Delay in ms between hosts |
| `--exclude CIDRS` | Comma-separated CIDRs to exclude |
| `--resume` | Skip completed hosts via state file |
| `--state FILE` | Resume state file (default `scan_state.json`) |
| `--screenshots` | Capture Playwright screenshots |
| `--no-rdap` | Skip RDAP enrichment |
| `--output-dir DIR` | Output folder (default `noauth_results/`) |
| `--report FILE` | Primary report path |
| `--output-format` | `json`, `csv`, `html`, or `md` |
| `--markdown-report FILE` | Also write Markdown report |
| `--ipv6` | Enable IPv6 scanning |
| `--no-color` | Disable colored output |
| `--no-live-feed` | Disable live triage |
| `--open-feed` | Show live intel for all hosts |

## License

MIT &mdash; see [LICENSE](LICENSE).

## Disclaimer

For authorized security testing, research, and defensive assessment only. Unauthorized scanning may violate applicable laws.
