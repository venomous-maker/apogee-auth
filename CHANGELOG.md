# Changelog

Every release of apogee-auth, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

### Fixes

- A Redis connection shared by several threads (the cache, session and queue-dashboard stores, used by a multi-threaded server or queue worker) no longer mixes up replies or stalls: every command is one locked round trip (grav-redis 0.2.2). Uses the new releases of apogee-session, grav-redis.

## 0.2.2 (2026-11-25)

### Dependencies

- Uses apogee-http 0.3.0, apogee-session 0.2.2, apogee-routing 0.3.0 (from their GitHub tags).

## 0.2.1 (2026-11-20)

### Requirements

- Uses the 0.2.1 releases of apogee-session and grav-redis.

## 0.2.0 (2026-11-20)

### Requirements

- Needs grav 1.9.0 or later, and uses the 0.2.0 releases of the packages it
  depends on.

## 0.1.0 (2026-11-19)

The first tagged release. Needs grav 1.7.0 or newer.

Authentication and authorization for [Apogee](https://github.com/venomous-maker): password hashing, guards that find a request's user (session with remember-me, bearer token, HTTP Basic), the `auth`/`guest`/`can` middleware, and a gate with abilities and policies.
