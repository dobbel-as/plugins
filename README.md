# Dobbel for AI-agenter

[Dobbel](https://dobl.no/) er et AI-først regnskapssystem for små norske
aksjeselskap og holdingselskap. Denne pluginen kobler agenten din til
regnskapet via Dobbels MCP-server, og gir den oppskriftene for å jobbe
riktig: lese regnskapet, registrere kjøp fra kvitteringer, bokføre banken,
hente rapporter og forberede MVA-melding og årsoppgjør.

Alt agenten fører, blir utkast du godkjenner i Dobbel, med mindre du
bevisst har gitt den mer. Hvert kall logges som en agent-handling.

## Installer for Claude Code

```bash
claude plugin marketplace add dobbel-as/plugins
claude plugin install dobbel@dobbel
```

Start Claude Code på nytt (eller kjør `/reload-plugins`), kjør `/mcp` og
velg `dobbel` for å logge inn. Skills:

- `/dobbel:koble-til` — sjekk koblingen og få hjelp med innloggingen
- `/dobbel:bokfor-innboks` — registrer kjøp fra kvitteringer og fakturaer
- `/dobbel:avstem-og-lukk` — bokfør banken og lukk perioden
- `/dobbel:gjennomga` — les-bare gjennomgang av en periode mot dokumentene

I Claude Code på web (claude.ai/code) aktiveres pluginen i prosjektets
`.claude/settings.json`:

```json
{ "enabledPlugins": { "dobbel@dobbel": true } }
```

## Installer for Codex

```bash
codex plugin marketplace add dobbel-as/plugins
codex plugin add dobbel@dobbel
codex mcp login dobbel
```

Start en ny oppgave etterpå. Codex CLI, ChatGPT-appen og IDE-utvidelsen
deler konfigurasjonen.

## Andre klienter

Bruker du claude.ai, Claude Desktop, ChatGPT eller Gemini uten plugin,
legger du til `https://app.dobl.no/api/mcp` som egendefinert kobling.
Guider: [hjelp.dobl.no/tilkoblinger](https://hjelp.dobl.no/tilkoblinger).
Den fullstendige håndboken for agenter ligger på
[app.dobl.no/skill.md](https://app.dobl.no/skill.md).

## Innlogging og tilganger

Innloggingen er OAuth 2.1 i nettleseren. Du velger hvilke selskap
koblingen dekker, og krysser av tilganger. Trygge tilganger (lese, lage
utkast, stamdata) er forhåndsvalgt; tilganger som gjør noe endelig
(postere, sende, betale, levere til Skatteetaten) må du velge selv. I
tillegg har hvert selskap et automasjonsnivå — Lese, Foreslå eller Gjøre —
som settes under AI-agenter i Dobbel. Begge porter må åpne før agenten
får gjøre noe.

## Eksempler

- «Hva gjenstår i regnskapet mitt akkurat nå?»
- «Registrer kvitteringene i denne mappen som kjøp i Dobbel.»
- «Bokfør banktransaksjonene for august og si hva som mangler bilag.»
- «Gå gjennom andre kvartal mot dokumentene før jeg leverer MVA-meldingen.»

## Oppbygning

- `plugins/dobbel/` — selve pluginen, delt av Claude Code og Codex
  - `.claude-plugin/plugin.json` og `.mcp.json` — Claude Code
  - `.codex-plugin/plugin.json` og `codex-mcp.json` — Codex
  - `skills/` — kjerneskillen `dobbel` og de fire arbeidsflytene
- `.claude-plugin/marketplace.json` — marketplace for Claude Code
- `.agents/plugins/marketplace.json` — marketplace for Codex
- `.github/workflows/validate.yml` — JSON, versjonslikhet og
  `claude plugin validate --strict`

Versjonen i de tre manifestene skal alltid være lik. Bump den når skills
endres; brukere får oppdateringer først når versjonen bumpes.

## Lisens

Apache-2.0. Selve Dobbel er et lukket produkt; pluginen er dokumentasjon
og konfigurasjon for å bruke det.
