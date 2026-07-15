# bc-dh-customer-api

API kontrakt služby **bc-dh-customer** (evidence zákazníků — majitelů psů) systému
**Dog Hotel**. Repo drží `openapi.yaml` jako zdroj pravdy a přes
[openapi-generator](https://openapi-generator.tech/) (Apache-2.0) z něj generuje:

- **klienta** — Spring **HTTP Interface** (`@HttpExchange` + `RestClient`),
  balíček `cz.doghotel.customer.api.client` — konzumují BFF a jiné BC;
- **server stub** — interfaces + DTO, **delegate pattern**, balíček
  `cz.doghotel.customer.api.server` — implementuje `bc-dh-customer`.

Artefakt `cz.doghotel:bc-dh-customer-api` se publikuje do **GitHub Packages**
(SemVer, start `0.1.0`).

Související repa: [dh-infra](https://github.com/Miraskazik/dh-infra) ·
[bc-dh-auth](https://github.com/Miraskazik/bc-dh-auth) · `bc-dh-customer` (služba, implementace).

## Contract-first

`openapi.yaml` je **jediný zdroj pravdy**. Změna API = PR do **tohoto** repa
(nikdy ne přímo ve službě). CI vygeneruje klienta + server stub a na push do
`main` publikuje nový artefakt.

## Endpointy v1 (`/api/customers`)

| Metoda | Cesta | Popis |
|---|---|---|
| POST | `/api/customers` | Vytvoří zákazníka |
| GET | `/api/customers/{id}` | Detail zákazníka |
| GET | `/api/customers` | Stránkovaný seznam + hledání dle jména/telefonu (`page`, `size`, `search`) |
| PUT | `/api/customers/{id}` | Úprava zákazníka |
| DELETE | `/api/customers/{id}` | Soft delete (nastaví `deletedAt`, data zůstávají) |

Entita `Customer`: `id` (UUID), `name`, `phone`, `notes`, `createdAt`, `updatedAt`.
Chyby: RFC 9457 `application/problem+json`. Zabezpečení: Keycloak JWT (`bearerAuth`).
Zákazník nedrží počty psů ani id rezervací — detail skládá BFF.

## Build

```bash
mvn verify          # vygeneruje klienta + server stub a zkompiluje (validuje spec)
mvn deploy          # publikace do GitHub Packages (dělá CI na push do main)
```

## Použití artefaktu (konzument)

```xml
<dependency>
  <groupId>cz.doghotel</groupId>
  <artifactId>bc-dh-customer-api</artifactId>
  <version>0.1.0</version>
</dependency>
```

Stažení z GitHub Packages vyžaduje v `~/.m2/settings.xml` server `github`
s tokenem se scope `read:packages` (v CI stačí `GITHUB_TOKEN`).

## CI

- **PR do `main`:** `mvn verify` — build z `openapi.yaml` (nevalidní spec build shodí).
- **Push do `main`:** `mvn deploy` → GitHub Packages. Nová verze = bump `<version>`
  v `pom.xml` (GitHub Packages release verzi nepřepisuje).
- **PR review:** `claude.yml` (`@claude` mention / automaticky při otevření PR) —
  vyžaduje secret `ANTHROPIC_API_KEY`.
