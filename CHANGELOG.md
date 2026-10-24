# Release notes

<!-- do not remove -->

## 0.0.10

- `purge` sends its requests through a client with `max_retries=0`, so `tries` is the total number of requests for one purge. The SDK client retried a connection error, a rate limit, HTTP 408, HTTP 409 or a 5xx response twice inside each of the `tries` attempts.
- `purge` retries a connection error, HTTP 408, HTTP 409 and any HTTP 5xx response, as the SDK client did before `max_retries=0`. Each retry counts against `tries`.
- `purge` now defaults to `fallback_all=False`, so a refused partial purge raises and does not empty the whole zone cache. Pass `fallback_all=True` to purge everything after a refusal, as every earlier release did by default.
- `upsert_record` leaves records of other types alone, updates the kept record in place when its content or `proxied` setting differs, and deletes the other records of the same type under the name. It deleted every record under the name, whatever its type.
- `upsert_record` and `tunnel_cname` read `@` as the domain itself, ignore a trailing dot, and treat a name that ends in the domain as complete. A name with a dot, such as `api.internal`, is now a subdomain.
- `upsert_record` creates a new record under the full name. It passed the short name to `create_record`.
- `tunnel_cname` deletes the A and AAAA records at the name before it writes the CNAME record.
- `purge` classifies an API error whose `errors` is null, so the original failure reaches the caller.

## 0.0.9

- `purge` reaches Cloudflare only through the client's `_c`. A test client that attaches only `purge` can now send purge requests. 0.0.8 sent its requests through a second patched method, and such a test client replaced that method with a no-op.

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


