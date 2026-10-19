# Release notes

<!-- do not remove -->

## 0.0.9

- `purge` reaches Cloudflare only through the client's `_c`, so a stand-in for `CF` that binds `purge` alone purges again; 0.0.8 sent its requests through a second patched method, which such a stand-in replaced with a no-op

## 0.0.8

- `purge` purges everything only when Cloudflare refuses the partial purge, with HTTP 403 or a 400 naming the plan or a permission; it retries rate limits (429 or code 1134) and timeouts up to `tries=4` times with a doubling `wait`, and raises any other error instead of purging everything
- After a fallback, `purge` returns the files purge that succeeded before the refused prefixes purge, then the purge of everything
- `ruleset_upsert` keeps every rule it does not own whole, with keys such as `logging` and `ratelimit`, leaving out only Cloudflare's read-only `version`, `last_updated` and `categories`; it compares its own rules on every key they set

## 0.0.7

- `ruleset_upsert` treats a rule with no `description` or `enabled` as Cloudflare stores it, `''` and true, so running it again with such a rule sends nothing and returns False

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


