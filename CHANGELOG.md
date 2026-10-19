# Release notes

<!-- do not remove -->

## 0.0.6

- `ruleset_upsert` writes cache or custom firewall rules into a zone's entrypoint for a phase, replacing rules by `ref` and keeping the others; it returns False and sends nothing when the zone already has them
- `purge` purges a zone's cache by `files`, `prefixes` or `everything`, and purges everything when a partial purge is refused unless `fallback_all=False`; it returns the purges sent

## 0.0.5

- `verify` checks the token is active and the account, zones (or one `domain`) and tunnels are reachable, returning `ok`, each result and an `errors` dict; it used to return `{'result': True}` whenever the calls succeeded

## 0.0.4
skill.md



## 0.0.3
setup tunnels and tunnel_id



## 0.0.2

release


## 0.0.1
cloudflare tunnel initial commit


