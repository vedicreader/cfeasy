---
name: cfeasy
description: >
  Cloudflare DNS and Zero Trust tunnel management from Python. Use when a task
  needs DNS records, tunnel creation, or token verification against the
  Cloudflare API.
---

# cfeasy: Cloudflare DNS and tunnels for coding agents

`cfeasy` is a small wrapper over the official `cloudflare` Python SDK. The SDK is
verbose for the two jobs most VPS automation tasks need: write a DNS record, and
set up a Zero Trust tunnel. Use cfeasy before you run `wrangler` or write REST
calls by hand.

## How to invoke

```python
from cfeasy import CF
c = CF()                  # reads CLOUDFLARE_API_TOKEN from the env
c = CF('your-token')      # or pass one explicitly
c.verify()                # -> {'ok': True, 'token': 'active', 'account': ..., 'zones': [...], 'tunnels': 0, 'errors': {}}
c.verify('example.com')   # check one zone instead of listing them all
```

If `verify()` returns `ok=False`, `errors` names each failed check (`token`,
`account`, `zones`, `tunnels`). A token that is not `active`, or a failing `zones` or
`tunnels` check, means the token is expired or lacks scopes. The required custom token permissions are:

| Capability | Resource | Permission |
|------------|----------|------------|
| DNS read/write | Zone -> DNS | Edit (scoped to your zone) |
| Account info | Account -> Account Settings | Read |
| Tunnel management | Account -> Cloudflare Tunnel | Edit |

A Global API Key will not work. Create a Custom Token at
`dash.cloudflare.com/profile/api-tokens`.

---

## The decision loop

Most cfeasy tasks reduce to one of three questions. Pick the matching call.

| Question | Call |
|----------|------|
| Is my token valid for what I'm about to do? | `c.verify()` |
| Make `name.domain` resolve to an IP or CNAME, idempotently | `c.upsert_record(domain, name, content, type='A')` |
| Stand up a Zero Trust tunnel and wire DNS to it | `c.setup_tunnel(domain, name)` |

`upsert_record` and `setup_tunnel` are both idempotent. Call them on every
deploy without checking state first.

---

## DNS records

`upsert_record` is the only DNS method you usually need. It looks at the records
of the requested type under the name and leaves records of other types alone. If none exists, it creates the
record. Otherwise it keeps one record: the first with the requested content, else
the first of that type. It updates the kept record in place when its content or
`proxied` setting differs. It deletes the other records of that type under the name.
Repeat calls give the same result.

```python
# Plain A record
c.upsert_record('example.com', 'app', '1.2.3.4')

# CNAME with Cloudflare proxy on
c.upsert_record('example.com', 'docs', 'docs.pages.dev',
                type='CNAME', proxied=True)

# Apex (use the bare domain as the name)
c.upsert_record('example.com', 'example.com', '1.2.3.4')
```

Lower-level methods are there if you need them: `zones()`, `zone_id(domain)`,
`dns_records(zone_id)`, `create_record(...)`, `delete_record(zone_id, rid)`.
Use them only when `upsert_record` does not fit, for example when listing or
auditing existing state.

---

## Tunnels

A Cloudflare Tunnel is an outbound connection from your server to Cloudflare's
edge. Traffic to `app.example.com` reaches Cloudflare. Cloudflare sends it
through the tunnel to your local `cloudflared` process, which forwards it to the
service the tunnel is configured for. The server needs no open firewall ports
and no public IP.

Three pieces have to line up:

1. A named tunnel registered with your account.
2. A DNS CNAME pointing the hostname at `<tunnel-id>.cfargotunnel.com`.
3. A token passed to `cloudflared` so it knows which tunnel to join.

`setup_tunnel` does all three in one call. It creates the tunnel if it does not
exist, reuses it if it does, sets up the CNAME, and returns `(tunnel_id, token)`.

```python
# Subdomain: app.example.com
tid, token = c.setup_tunnel('example.com', 'app')

# Apex domain (Cloudflare flattens the CNAME at the root)
tid, token = c.setup_tunnel('example.com')

# Custom tunnel name (defaults to the subdomain)
tid, token = c.setup_tunnel('example.com', 'app', tunnel_name='prod-app')
```

Pass `token` to `cloudflared` as `CF_TUNNEL_TOKEN`. In a Docker Compose setup
this is a `cloudflared tunnel --no-autoupdate run` service with the token in its
environment.

If you only have the name and need the ID, or want to clean up:

```python
tid = c.tunnel_id('app')          # look up by name
c.delete_tunnel(tid)              # remove the tunnel
c.tunnels()                       # list all tunnels on the account
```

---

## Quick reference

| Method | Use |
|--------|-----|
| `c.verify()` | Read-only token check. Run before anything destructive. |
| `c.upsert_record(domain, name, content, type='A', proxied=False)` | Idempotent DNS write. |
| `c.setup_tunnel(domain, name=None, tunnel_name=None, proxied=True)` | Full tunnel + CNAME setup. Returns `(tid, token)`. |
| `c.tunnels()` / `c.tunnel_id(name)` | List or look up tunnels. |
| `c.tunnel_cname(domain, name, tid, proxied=True)` | Point a name at a tunnel you already have. It deletes A and AAAA records at that exact name, then upserts the CNAME. Other record types stay. |
| `c.tunnel_token(tid)` | Fetch the token string for an existing tunnel. |
| `c.delete_tunnel(tid)` | Remove a tunnel by ID. |
| `c.zones()` / `c.zone_id(domain)` | Zone lookup, mainly for the lower-level DNS methods. |

---

## Helpers

```python
from cfeasy import repo_root, mv_skill_md

repo_root()              # Path to the nearest .git ancestor, or None
mv_skill_md(dry_run=True)  # preview where SKILL.md would be installed
mv_skill_md(dry_run=False) # copy it into .agents/skills/cfeasy/ and .claude/skills/cfeasy/
```

`mv_skill_md` copies the bundled SKILL.md into a project's agent skill
directories, where coding agents load it. Run it once after you install cfeasy
in a project that needs this skill.