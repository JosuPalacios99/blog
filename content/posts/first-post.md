---
title: "Welcome — First Post"
date: 2026-06-12
draft: false
tags: ["meta", "redteam"]
categories: ["notes"]
summary: "How this blog works and a quick syntax reference."
---

First post. Drop a `.md` file in `content/posts/` and it becomes a page.

## Frontmatter

Every post starts with a YAML block:

```yaml
---
title: "AS-REP Roasting Walkthrough"
date: 2026-06-12
draft: false          # true = hidden from build
tags: ["kerberos", "ad"]
categories: ["writeups"]
summary: "Short blurb for the post list."
---
```

## Code blocks

Fenced blocks get syntax highlighting + a copy button:

```bash
# Kerberoast with impacket
GetUserSPNs.py -request -dc-ip 10.10.10.1 corp.local/lowpriv:Password1
```

```python
import requests
r = requests.get("http://target/api", headers={"X-Forwarded-For": "127.0.0.1"})
print(r.status_code)
```

## Images

Put files in `static/img/` and reference them:

```markdown
![beacon flow](/img/beacon.png)
```

## Callouts

> **OPSEC:** sanitize client names, IPs, and creds before publishing.

Done. Set `draft: false` to publish.
