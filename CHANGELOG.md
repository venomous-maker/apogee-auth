# Changelog

Every release of apogee-auth, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

The first tagged release. Needs grav 1.7.0 or newer.

Authentication and authorization for [Apogee](https://github.com/venomous-maker): password hashing, guards that find a request's user (session with remember-me, bearer token, HTTP Basic), the `auth`/`guest`/`can` middleware, and a gate with abilities and policies.
