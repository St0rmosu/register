# register — Subdomain `lollo.is-a.dev`

[![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![DNSControl](https://img.shields.io/badge/DNSControl-v4.14-5C2D91?style=for-the-badge)](https://dnscontrol.org/)
[![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)](https://www.cloudflare.com/)

Fork del repository ufficiale [is-a-dev/register](https://github.com/is-a-dev/register) usato per gestire il dominio **`lollo.is-a.dev`**. Il repository registra i record DNS (CNAME verso Vercel + TXT di verifica) di questo sottodominio e si sincronizza periodicamente con l'upstream per mantenere la validazione dei contributi.

## Cos'è is-a.dev

**is-a.dev** è un servizio gratuito che permette agli sviluppatori di ottenere un sottodominio `.is-a.dev` per i propri siti personali. La registrazione avviene tramite pull request: ogni sottodominio è un file JSON in `domains/`, il cui contenuto viene validato automaticamente da test e poi pubblicato sui DNS di Cloudflare.

## Caratteristiche

- **Dominio registrato**: `lollo.is-a.dev` con CNAME verso `cname.vercel-dns.com` (hosting Vercel).
- **Verifica Vercel**: record TXT `vc-domain-verify` per la validazione del dominio sulla piattaforma.
- **Fork sincronizzato con upstream**: mantiene la pipeline completa di CI/CD del progetto originale.
- **Struttura dichiarativa**: configurazione DNS in `dnsconfig.js` (DNSControl).

## Tech Stack

| Tecnologia | Ruolo |
|---|---|
| DNSControl 4.14 | Linguaggio dichiarativo per generare/applicare i record DNS |
| Cloudflare | Provider DNS del dominio `is-a.dev` |
| Node.js 20 | Runtime per lo script di generazione del dataset (`util/raw-api.js`) |
| AVA (Node) | Test runner per la validazione dei file in `domains/` |
| GitHub Actions | CI/CD: validazione PR e `dnscontrol push` al merge |

## Architettura

Flusso di registrazione e pubblicazione:

```
   Utente                     CI (GitHub Actions)                Produzione
   ┌──────┐                    ┌────────────────┐                ┌─────────┐
   │ fork │  ── PR ──►         │  ci.yml        │                │         │
   │+JSON │                    │  · AVA tests   │                │  DNS    │
   └──────┘                    │  · dnscontrol  │                │Cloudflare│
                               │    check       │                │         │
                               └───────┬────────┘                └────┬────┘
                                       │ merge su main               │
                                       ▼                             ▼
                               ┌──────────────┐    push    ┌────────────────┐
                               │  publish.yml │ ─────────► │ dnscontrol push│
                               └──────────────┘            └────────────────┘
```

I file in `domains/*.json` vengono letti da `dnsconfig.js`, che genera i record per la zona `is-a.dev`. La validazione è a due livelli: test AVA sui file (schema, valori dei record, ownership del PR) e `dnscontrol check` sulla configurazione. Al merge, `dnscontrol push` applica le modifiche a Cloudflare.

## Project Structure

```
register/
├── domains/
│   ├── lollo.json           # Il sottodominio di questo fork (CNAME → Vercel)
│   └── _vercel.lollo.json   # TXT di verifica dominio Vercel
├── dnsconfig.js             # Configurazione zona DNSControl
├── util/                    # Script di supporto (reserved, raw-api, validazioni)
├── tests/                   # Suite AVA (domains, json, proxy, pr, records)
├── .github/workflows/       # CI/CD: ci, dnscontrol, publish, raw-api, stale
└── package.json             # Script di test (npx ava tests/*.test.js)
```

## Installation & Setup

Clonare e testare localmente (per validare modifiche ai file dei domini):

```bash
git clone https://github.com/St0rmosu/register.git
cd register
npm install
npm test          # esegue la suite AVA su tutti i file in domains/
```

## Usage

Questo fork non va modificato per richiedere nuovi domini (le richieste si aprono su [is-a-dev/register](https://github.com/is-a-dev/register)). Il suo scopo è possedere e gestire `lollo.is-a.dev`:

1. `domains/lollo.json` definisce il CNAME verso l'hosting (attualmente Vercel).
2. `domains/_vercel.lollo.json` contiene la TXT di verifica richiesta da Vercel.
3. Le modifiche vengono applicate al merge su `main` dalla pipeline `publish.yml`.
4. Per cambiare hosting, aggiorna il CNAME e il record TXT di verifica di conseguenza.

## API Documentation

Nessuna API pubblica in questo fork. Nel progetto upstream, `util/raw-api.js` genera il dataset JSON pubblico per il progetto [is-a-dev/raw-api](https://github.com/is-a-dev/raw-api), che espone l'elenco dei sottodomini.

## Engineering Decisions

- **Configurazione dichiarativa**: `dnsconfig.js` usa DNSControl, che mappa ~14.000 file JSON in record DNS in modo deterministico e riproducibile, senza scripting manuale.
- **Validazione a due livelli**: test unitari AVA (regole di business: ownership, record validi, CNAME non proxied, ecc.) + `dnscontrol check` (validità della zona), entrambi in CI.
- **Ownership enforcement in CI**: i metadati del PR (autore, file modificati) vengono iniettati come env var nei test, così ogni utente può toccare solo i propri sottodomini.
- **Sincronizzazione con upstream**: mantenere il fork aggiornato è essenziale per ricevere correzioni di test e regole di validazione.

## Testing

```bash
npm test
```

La suite copre: naming dei file (`json.test.js`), regole di nested e ownership (`domains.test.js`, `pr.test.js`), valori dei record DNS (IPv4/IPv6, MX, NS, SRV, DS, CAA, TLSA, TXT, URL) e vincoli di proxy (`records.test.js`, `proxy.test.js`).

## Limitations & Future Improvements

- Fork personale: le modifiche locali al fork non devono confliggere con l'upstream (merge frequenti consigliati).
- La verifica di proprietà del dominio è legata al record TXT di Vercel: cambiare hosting richiede l'aggiornamento di entrambi i file.
- Prossimi passi: automatizzare il sync con upstream (es. GitHub Actions di rebase), aggiungere redirect/record aggiuntivi se il sito cresce.

---

*Fork di [is-a-dev/register](https://github.com/is-a-dev/register) — gestito da St0rmosu.*
