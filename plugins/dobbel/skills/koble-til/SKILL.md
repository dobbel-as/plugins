---
name: koble-til
description: Sjekk at Dobbel-koblingen virker og hjelp brukeren gjennom innloggingen. Bruk når Dobbel-verktøyene mangler, når et kall feiler med 401 eller insufficient_scope, eller når brukeren ber om å koble Dobbel til agenten.
---

# Koble til Dobbel

Målet er en aktiv kobling der `get_context` svarer med minst ett selskap.
Du kan ikke logge inn på vegne av brukeren. Din jobb er å si nøyaktig hva
brukeren skal gjøre, og å bekrefte etterpå.

## 1. Sjekk tilstanden

- Finnes `get_context` i verktøylisten? Kall det.
  - Svarer det med selskaper: koblingen er aktiv. Oppsummer selskap,
    `autonomy_level` og scopes for brukeren, og gå videre til oppgaven.
  - Svarer det 401 eller «No OAuth context»: serveren er registrert, men
    brukeren er ikke logget inn. Gå til steg 2.
- Finnes ikke `get_context`: pluginen er installert, men MCP-serveren er
  ikke lastet ennå. En kjørende økt plukker ikke opp nye servere — be
  brukeren starte en ny økt (Claude Code: avslutt og start `claude` på
  nytt, eller `/reload-plugins`; Codex: ny oppgave).

## 2. Be brukeren logge inn

Pluginen har allerede registrert serveren `https://app.dobl.no/api/mcp`.
Innloggingen er OAuth i nettleseren, og går slik:

- **Claude Code:** kjør `/mcp`, velg `dobbel`, og følg nettleseren.
- **Codex:** kjør `codex mcp login dobbel` i terminalen.

I nettleseren logger brukeren inn i Dobbel, velger hvilke selskap
koblingen skal dekke, og krysser av tilganger. Trygge tilganger (lese,
lage utkast, stamdata) er forhåndsvalgt. Tilganger som gjør noe endelig
må brukeren krysse av selv:

| Scope | Gir |
|---|---|
| `vouchers:post` | postere og reversere bilag |
| `invoices:send` | sende fakturaer til kunder |
| `bank:pay` | sende betalinger til banken |
| `bank:reconcile` | lukke bankavstemminger |
| `annual:generate` | årsoppgjør og SAF-T |
| `filings:submit` | sende MVA-melding og aksjonærregisteroppgave til Skatteetaten |

Si til brukeren hvilke av disse oppgaven faktisk trenger, så de ikke gir
mer enn nødvendig. Standard er nok for å lese, registrere kjøp som utkast
og foreslå bank-bokføring.

## 3. Bekreft

Kall `get_context` på nytt. Rapporter:

- selskap(er) og hvilket som er standard
- `autonomy_level` (Lese / Foreslå / Gjøre) og hva det betyr for oppgaven
- scopes, og om noen oppgaven trenger mangler

Mangler et scope, kan brukeren ikke legge det til i etterkant: koblingen
må fjernes under «AI-agenter» i Dobbel og lages på nytt med tilgangen
avkrysset.

## Feilsøking

- **`insufficient_scope`** — tokenet mangler scopet. Se over.
- **«Autonomi-nivå er …»** — selskapets nivå stopper handlingen. Be om
  godkjenning i appen, eller at en admin hever nivået under AI-agenter.
- **`cross_tenant_denied`** — `companyId` er utenfor tokenet. Bruk id-ene
  fra `get_context`.
- **`module_disabled`** — verktøyet er slått av for selskapsprofilen,
  typisk salgsverktøy på et holdingselskap.
- **Innloggingen looper** — be brukeren fjerne koblingen i klienten og
  legge den til på nytt, og sjekke at minst ett selskap ble valgt.

Mer: `https://app.dobl.no/hjelp/tilkoblinger`.
