# Configuration without hardcoding

No network literal, credential, or strategy selection lives in the Rust source.
Everything is resolved in layers with **figment**:

```rust
Figment::new()
    .merge(Toml::file("config.toml"))         // non-secret, git-tracked
    .merge(Env::prefixed("GQL_").split("__"))  // secrets + overrides
    .extract()
```

Nested keys split on `__`: `GQL_DATABASE__URL` → `database.url`.

## `rust-gql-backend/config.toml`

```toml
[server]
host = "127.0.0.1"
port = 4000
base_url = "http://localhost:4000"

[database]
case = "snake_case"
log = false
# url -> GQL_DATABASE__URL (secret, NOT in the file)

[storage]
provider = "supabase"   # supabase | s3
bucket = "media"

[auth]
provider = "session"    # session | jwt | oidc

[subscriptions]
transport = "sse"       # sse | ws

[cors]
origins = ["http://localhost:5173"]
```

## Boot-time validation (fail-fast)

After loading, `AppConfig::validate()` rejects a configuration that parsed but is
not operable: a missing secret (`database.url`, `storage.bucket`) or a strategy
string outside the set the binary can dispatch on. The error **names the exact
field** and the process does not start half-configured.

Secrets (`DATABASE_URL`, storage credentials) come from the environment,
mirroring the same variables the Node backend already uses.
