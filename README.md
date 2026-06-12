# pullist

Firewall openings needed to pull container images — like endoflife.date, but for
the "one URL that wasn't opened" problem.

Registries are never one hostname: auth lives on one domain, manifests on another,
and layer downloads redirect to a CDN nobody thought to open. Each product page
lists every host with its purpose, ports, whether it's required, sources, and a
last-verified date.

## Layout

```
data/*.yaml   one file per product - this is the only thing contributors touch
build.py      reads data/, writes site/ (only dependency: PyYAML)
site/         generated static output - host anywhere
```

Per product the build generates:

- `<slug>.html` — human-readable page
- `<slug>.txt`  — plain newline-separated domain list (paste into firewall/proxy)
- `<slug>.json` — machine-readable (also `all.json` with everything)
- `check-<slug>.sh` — POSIX sh + curl connectivity test to run from inside the
  customer's network: `curl -fsSL https://your-site/check-redhat.sh | sh`

## Build

```
pip install pyyaml
python3 build.py
```

Serve `site/` with anything: GitHub Pages, `python3 -m http.server`, or nginx.

## Data format

```yaml
title: GitHub (ghcr.io)
slug: ghcr
description: GitHub Container Registry. Two hosts, both required.
hosts:
  - host: ghcr.io
    ports: [443]
    purpose: Registry API and token authentication
    required: true
  - host: "*.data.example.com"      # wildcards are fine, but then set:
    test_host: eastus.data.example.com   # a concrete host for the check script
notes: |
  Free-form. Explain the failure mode when a host is missing.
sources:
  - https://...
last_verified: 2026-06-12
```

Contributions = edit or add a YAML file, bump `last_verified`, link a source.

## Deploy on GitHub Pages

Commit, then add `.github/workflows/pages.yml` that runs `python3 build.py` and
publishes `site/` — or just commit `site/` and point Pages at it. No build step
is also a valid lifestyle.
