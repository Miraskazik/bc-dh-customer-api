# CLAUDE.md — bc-dh-customer-api

## Role repa

**API kontrakt** (Typ A) služby `bc-dh-customer` v systému **Dog Hotel**.
`openapi.yaml` je zdroj pravdy; openapi-generator z něj generuje HTTP Interface
klienta a server stub (delegate pattern), artefakt jde do GitHub Packages.
Sám o sobě to není běžící aplikace — jen generovaný kontrakt.

Stack systému: Java 25 · Spring Boot 4.1.0 (Spring Framework 7, Jakarta EE 11,
Jackson 3) · Maven · PostgreSQL · Kafka · Keycloak.

## Související repa

- `bc-dh-customer` — služba, která implementuje generované server interfaces
- `bff-dog-hotel`, `bc-dh-reservation`, `bc-dh-billing` — konzumenti klienta
- [dh-infra](https://github.com/Miraskazik/dh-infra) — docker-compose, event katalog, architektura
- [bc-dh-auth](https://github.com/Miraskazik/bc-dh-auth) — Keycloak realm (JWT issuer)

## Contract-first (tvrdé pravidlo)

- Změna API se dělá **jen zde** (PR do tohoto repa úpravou `openapi.yaml`),
  **nikdy** přímo ve službě `bc-dh-customer`.
- CI z `openapi.yaml` vygeneruje klienta + server stub; služba i konzumenti
  jen přidají dependency na publikovaný artefakt.

## Konvence

- Maven souřadnice: `cz.doghotel:bc-dh-customer-api`, SemVer, start `0.1.0`.
  **Nová release verze = bump `<version>` v `pom.xml`** (GitHub Packages release
  verzi nepřepisuje — deploy stejné verze skončí konfliktem).
- Java balíčky: `cz.doghotel.customer.api.client` (+ `.client.model`),
  `cz.doghotel.customer.api.server` (+ `.server.model`).
- Supporting files generátoru jsou omezené (jen `ApiUtil.java` u server stubu),
  aby se mezi 4 `*-api` repy nekolidovala třída `org.openapitools.configuration.*`.
  Tohle **neměnit** bez rozmyslu.
- Chyby v kontraktu: RFC 9457 ProblemDetail. Soft delete: DELETE jen nastavuje příznak.

## Příkazy

- `mvn verify` — vygeneruje klienta + server stub a zkompiluje (validace `openapi.yaml`).
- `mvn deploy` — publikace do GitHub Packages (v CI na push do `main`).

## Git workflow

- default branch `main`; bootstrap = jeden initial commit; další práce ve feature
  branchích `feat/…`, `fix/…`, `chore/…` → PR → merge.
- **Conventional Commits** (`feat:`, `fix:`, `chore:`, `docs:`…), ucelené celky práce.
- **Každý PR musí projít `mvn verify` (build z openapi.yaml) před mergem.**
- PR review přes `claude.yml` (`@claude` / automaticky při otevření PR).

## Open-source / free (tvrdé pravidlo)

Jen OSI-approved licence: Apache-2.0, MIT, BSD, EPL, MPL, LGPL. Zakázané bez
souhlasu: placené služby, omezené free tiery, BSL/BUSL, SSPL, Elastic License,
Confluent Community License, „source available". U každé nové dependency uvádět
licenci. Když free varianta neexistuje nebo je licence nejasná — **zastavit,
popsat možnosti, počkat na rozhodnutí**.

## Ověřování API (tvrdé pravidlo)

Stack (Java 25, Spring Boot 4.1, Jakarta EE 11, Jackson 3) je novější než
trénovací data modelu. Spring Boot API, properties, Maven souřadnice a
openapi-generator konfiguraci **nepsat z paměti** — ověřit přes Context7 nebo
docs.spring.io. Když ověřit nejde, říct to.
