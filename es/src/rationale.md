# Por qué

## Contrato verificado por el compilador a ambos lados del cable

GraphQL deja de ser "formato de API" para convertirse en un contrato que el
compilador comprueba en los dos clientes Rust:

- El SDL emitido (`schema.graphql`) alimenta a graphql-client (Leptos) y a
  graphql-codegen (SPA Node). Un cambio incompatible rompe la generación.
- El crate `domain` añade un **segundo** contrato — reglas de dominio que corren
  idénticas en el resolver (native) y en el navegador (wasm) — que Node no puede
  expresar. Una regla de validación se escribe una vez y no puede divergir entre
  servidor y cliente porque es el mismo código compilado a dos targets.

## Por qué 3 carpetas y no un workspace único

Separar `domain` como carpeta hermana mantiene la propiedad clave (un cambio de
schema rompe la compilación de ambos lados) **sin** meter un backend con `sqlx`
+ `tokio` y un frontend `wasm` en el mismo workspace, donde las features y
targets se estorban.

## Por qué auth y subscriptions como puntos de extensión

La flexibilidad se modela como `trait` + `feature`, no como decisión prematura:

- **Auth** — `trait AuthProvider` con impls `session` (replica el contrato de
  cookie/sesión de Better-Auth que el SPA necesita), `jwt` y `oidc`,
  seleccionables por config.
- **Subscriptions** — `SubscriptionRoot` común con adaptadores `sse` (compat SPA
  Node) y `ws` nativo.

El conjunto por defecto es el que el SPA Node existente necesita; el resto queda
tras feature, sin construir las cinco implementaciones a la vez.

## Por qué config-driven

Los literales dispersos del backend Node (puerto `4000`, `baseURL`, casing) se
consolidan en `config.toml` + entorno. La configuración es dato, no código: se
cambia de estrategia de auth o de transporte de subs **sin recompilar lógica**,
sólo tocando el TOML o una variable de entorno.
