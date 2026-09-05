---
name: dobbel
description: Kjerneregler for å jobbe i Dobbel, et norsk regnskapssystem for små AS og holdingselskap, via MCP. Bruk alltid før du leser, bokfører, avstemmer eller rapporterer i Dobbel — den sier hva du gjør først, hvilke porter som styrer hva du får gjøre, og hvilke regler du aldri bryter.
---

# Dobbel for AI-agenter

Dobbel er et AI-først regnskapssystem for små norske aksjeselskap og
holdingselskap: NS 4102-kontoplan, norsk MVA, SAF-T og innlevering til
Altinn. Pluginen kobler deg til Dobbels MCP-server på
`https://app.dobl.no/api/mcp`. Alt i Dobbel er på norsk bokmål — svar
brukeren på det språket de skriver.

Den fullstendige og alltid oppdaterte håndboken ligger på
`https://app.dobl.no/skill.md` (versjon i `https://app.dobl.no/skills/manifest.json`).
Hent den når du trenger hele verktøylisten, scope-tabellen eller en
arbeidsflyt som ikke står i denne pluginen. Denne skillen er den korte
versjonen som alltid gjelder.

## Første kall i hver samtale

1. Finnes `get_context` i verktøylisten? Hvis ikke, bruk
   `/dobbel:koble-til` — pluginen har lagt til serveren, men brukeren må
   logge inn.
2. Kall `get_context`. Svaret gir selskapene tokenet dekker (id, navn,
   `company_profile`, `autonomy_level`), standardselskapet og scopene du
   har fått.
3. Send riktig `companyId` i alle påfølgende kall, og hold deg til **ett
   selskap per samtale** med mindre brukeren uttrykkelig ber om noe annet.

`company_profile = holding` betyr at salgsverktøyene (faktura, kunder,
produkter) er slått av for selskapet.

## To porter: scope og automasjonsnivå

**Scopes** velges av brukeren ved innlogging. Trygge scopes (lese, lage
utkast, stamdata) er forhåndsvalgt. Farlige scopes må brukeren krysse av
selv: `vouchers:post`, `invoices:send`, `annual:generate`, `bank:pay`,
`bank:reconcile`, `filings:submit`. Mangler du et scope, svarer verktøyet
`insufficient_scope` — be brukeren koble til på nytt med tilgangen
avkrysset. Ikke prøv omveier.

**Automasjonsnivå** settes av admin per selskap under «AI-agenter»:

| Nivå | Du kan |
|---|---|
| Lese (`NONE`) | lese og foreslå, ikke opprette noe |
| Foreslå (`DRAFT`), standard | opprette utkast; postering kun etter eksplisitt bekreftelse i chatten |
| Gjøre (`FULL`) | postere, lukke avstemminger og sende pliktige oppgaver |

Begge porter må åpne. En avvisning starter med «Autonomi-nivå er …». Da
ber du brukeren godkjenne i appen eller heve nivået. Du endrer det ikke
selv.

## Regler du aldri bryter

1. **Posterte bilag endres eller slettes aldri.** Feil rettes med
   `post_correction_voucher`. Utkast kan endres og slettes.
2. **Ingen tall uten kilde.** Finn aldri på beløp, datoer, MVA-sats,
   leverandør eller kontonummer. Står det ikke i dokumentet, banklinjen
   eller brukerens ord, spør du.
3. **Et kjøp er alltid konsekvens av et dokument.** `create_purchase`
   krever `fileId`. AI-forslaget fra `get_inbox_item` eller
   `extract_purchase_document` er en kandidat: les dokumentet selv med
   `get_file` og bekreft beløp, dato, leverandør og MVA mot det før du
   oppretter noe.
4. **Førsteklasses verktøy før fritt bilag.** `create_voucher` med egne
   linjer er siste utvei. Bilagsmal for gebyr, renter og overføringer;
   `create_purchase` for kjøp; `create_invoice` for salg;
   `commit_bank_match` for bank; kapitalhendelser og investeringsverktøy
   for egenkapital og aksjer; `depreciate_asset` for avskrivning.
5. **En banklinje beviser at penger flyttet seg, ikke hvorfor.** Mangler
   dokumentasjonen, bruk `request_missing_document`. Gjett aldri en
   kostnadskonto for å få avstemmingen til å gå opp.
6. **Datoer flyttes ikke** for å få noe til å stemme.
7. **Pliktige oppgaver og betalinger er endelige.** Kjør alltid
   `preview_*` først, vis brukeren resultatet, og send bare når brukeren
   har sagt ja i denne samtalen.
8. **Stopp og spør** når periode, motpart, dokumentasjon eller ønsket
   behandling er uklar.
9. **Alt du gjør logges** med `actor_type = agent`. Skriv beskrivelser et
   menneske forstår i hovedboken senere.

## Formater

- Beløp er `number` i NOK med to desimaler (`1234.56`), aldri øre. Vis
  dem som `1 234,56 kr`.
- Datoer er ISO `YYYY-MM-DD`. Vis dem som `dd.mm.yyyy`.
- Kontonummer er firesifrede NS 4102-kontoer. Reskontro vises som
  `1500:10001`.
- MVA-koder er SAF-T-standardkoder; `list_vat_codes` gir listen.

## Arbeidsflyter i pluginen

- `/dobbel:koble-til` — sjekk at koblingen virker, og hjelp brukeren
  gjennom innloggingen.
- `/dobbel:bokfor-innboks` — registrer kjøp fra kvitteringer og
  fakturaer, fra fil eller fra innboksen.
- `/dobbel:avstem-og-lukk` — bokfør banken, forklar det som gjenstår, og
  lukk perioden når portene er åpne.
- `/dobbel:gjennomga` — les-bare gjennomgang av en periode mot
  originaldokumentene, uten å endre noe.

Spør brukeren «hva må jeg gjøre?», start med `list_pending_work`: den
sier hva som venter, og per kategori om du kan ta det selv
(`agentCapable`).
