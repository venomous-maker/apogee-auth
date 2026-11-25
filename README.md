# apogee-auth

Authentication and authorization for [Apogee](https://github.com/venomous-maker):
password hashing, guards that find a request's user (session with
remember-me, bearer token, HTTP Basic), the `auth`/`guest`/`can`
middleware, and a gate with abilities and policies.

```typescript
import "std"
import "apogee-auth"

let gate = new Gate()
gate.discoverPolicies()                               // app.PostPolicy decides app.Post
let auth = new AuthManager(gate)
auth.addGuard(new SessionGuard("web", users, new CookieJar(appKey)))
auth.addGuard(new TokenGuard("api", users))
router.aliasMiddleware("auth", fn(a: Array<string>) -> Middleware { return new Authenticate(auth, a) })
router.aliasInstance("guest", new RedirectIfAuthenticated(auth, "/home"))
router.aliasMiddleware("can", fn(a: Array<string>) -> Middleware { return new Authorize(auth, a) })

r.post("/login", fn(req: Request) -> Response {
    let ok = auth.session().attempt(req, {"email": req.string("email"), "password": req.string("password")}, req.boolean("remember"))
    return ok ? Response.redirect("/home") : Response.back(req)
}).middleware(["guest"])
r.put("/posts/{post}", ...).middleware(["auth", "can:update,post"])
```

## Passwords

`Passwords.make(pw)` gives `$pbkdf2-sha256$600000$<salt>$<key>`: a random
salt, PBKDF2-HMAC-SHA256 at 600,000 iterations (about 0.7 s), and the
parameters stored with it. `Passwords.check(pw, hash)` compares in constant
time. `Passwords.needsRehash(hash)` says when `Passwords.iterations` was
raised since: make a new hash at the next sign-in.

## Users

A user implements `Authenticatable` (`authId`, `authPassword`, and
`rememberToken`/`setRememberToken` for remember-me). A `UserProvider`
finds users: by id, by credentials (every key but `password` is a field to
match), by remember token, and by API token hash.
`MemoryUserProvider(users, "email")` keeps them in memory and issues API
tokens; the framework adds one for Orbit models.

## Guards

- `SessionGuard`:
  - `attempt(req, credentials, remember)`, `login(req, user, remember)`,
    `loginUsingId`, `once` (this request only), `logout`, `validate`,
    `user`/`check`/`guest`/`id`.
  - Signing in regenerates the session id, against session fixation.
  - Remember-me is a signed `id|token` cookie: a fresh session finds the
    user by it, and logout rotates the token.
- `TokenGuard`: `Authorization: Bearer <token>` (or `?api_token=`), looked
  up by its SHA-256, so stored hashes can't be replayed.
- `BasicGuard`: HTTP Basic credentials on every request.

## Middleware

- `auth` / `auth:api,web`: the first guard with a user lets the request
  through. With none: a 401 for JSON clients, else a redirect to `loginUrl`
  (with the intended URL flashed).
- `guest`: a signed-in user is sent home.
- `can:update,post`: the gate decides. The arguments after the ability are
  route parameters (their bound models) or class names (`can:create,app.Post`).

## The gate

```typescript
gate.define("see-reports", fn(user: any, args: Array<any>) -> any { return (user as User).isAdmin() })
gate.policy::<Post, PostPolicy>()
gate.before(fn(user: any, ability: string) -> bool? { return isSuperAdmin(user) ? true : null })

class PostPolicy extends Policy {
    fn update(user: User, post: Post) -> bool { return user.id == post.authorId }
    fn delete(user: User, post: Post) -> Decision {
        return user.id == post.authorId ? Decision.allow() : Decision.deny("You do not own this post.")
    }
    @Guests fn view(user: User, post: Post) -> bool { return post.published }
    fn create(user: User) -> bool { return user.isEditor() }      // gate.allows(user, "create", ["app.Post"])
}
```

- **Checks:** `allows`, `denies`, `check` (all of), `any`, `inspect` (the
  `Decision`), `authorize` (throws `AuthorizationException`: a 403, or the
  decision's status, as with `Decision.denyAsNotFound()`), and
  `forUser(user)`.
- **Abilities and policies:** an ability returns a bool or a `Decision`. A
  policy's `before` runs first.
- **Guests:** denied unless an ability is defined with `guests: true`, or a
  policy method is marked `@Guests`.
- **Discovery:** `discoverPolicies()` maps every `<Model>Policy` to
  `<Model>` in its namespace.

## Tests

```bash
grav test tests/auth.grav
```

MIT licensed.
