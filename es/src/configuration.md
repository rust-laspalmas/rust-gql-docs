# Configuración sin hardcoding

Ningún literal de red, credencial o selección de estrategia vive en el código
Rust. Todo se resuelve por capas con **figment**:

```rust
Figment::new()
    .merge(Toml::file("config.toml"))      // no-secretos, git-tracked
    .merge(Env::prefixed("GQL_").split("__")) // secretos + overrides
    .extract()
```

Las claves anidadas se parten en `__`: `GQL_DATABASE__URL` → `database.url`.

## `rust-gql-backend/config.toml`

```toml
[server]
host = "127.0.0.1"
port = 4000
base_url = "http://localhost:4000"

[database]
case = "snake_case"
log = false
# url -> GQL_DATABASE__URL (secreto, NO en el fichero)

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

## Validación al arranque (fail-fast)

Tras cargar, `AppConfig::validate()` rechaza una configuración que parseó pero no
es operable: secreto ausente (`database.url`, `storage.bucket`) o una cadena de
estrategia fuera del conjunto que el binario sabe despachar. El error **nombra el
campo exacto** y el proceso no arranca a medias.

Los secretos (`DATABASE_URL`, credenciales de storage) provienen del entorno,
espejo de las mismas variables que el backend Node ya usa.
