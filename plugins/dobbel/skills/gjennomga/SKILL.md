---
name: gjennomga
description: Uavhengig, les-bare gjennomgang av en periode i Dobbel mot originaldokumentene — hvert bilag, hver banklinje, hvert innbokselement får en status. Bruk når brukeren vil kontrollere det som er ført, før en MVA-termin, før årsavslutning, eller etter at en agent har bokført.
argument-hint: "[periode, f.eks. 2026-Q2] [omfang: alt | bank | kjøp]"
---

# Gjennomgå

Dette er kontrollen, ikke bokføringen. Du endrer ingenting. Du sammenligner
det som står i Dobbel med det dokumentene faktisk sier, og gir hvert
objekt en status. Konklusjoner du ikke kan begrunne fra en kilde, er
`ukjent`, ikke `ok`.

Les `/dobbel:dobbel` først hvis du ikke har gjort det i denne samtalen.

## Uavhengighet

- Bruk kun lesende verktøy: `review_period_evidence`, `get_file`,
  `get_voucher`, `get_purchase`, `get_bank_reconciliation`,
  `get_vat_report`, rapportene.
- Ikke gjenbruk AI-forslaget fra `get_inbox_item` som fasit. Det er
  kandidaten du kontrollerer. Les originaldokumentet selv med `get_file`.
- Har en annen agent bokført i perioden, ikke les dens resonnement som
  kilde. Bilaget står på egne ben eller ikke.
- Ingen vesentlighetsgrense med mindre brukeren har gitt deg én. Uten
  terskel er et avvik et avvik.

Start rapporten med en isolasjonserklæring: hvilken periode og hvilket
scope, at du bare har brukt lesende verktøy, og at ingen forslag fra
første runde er lagt til grunn.

## 1. Avgrens og tell

Bestem scope: periode og omfang (alt, bank eller kjøp). Hent registeret
med **ett** kall:

- `review_period_evidence(companyId, from, to, scope?)` — hvert bilag,
  kjøp, hver banklinje og hvert innbokselement i perioden, med `files`
  bak hvert objekt, koblingene mellom dem, og `evidence` per bilag:
  `document`, `system_generated`, `bank_statement_only` eller `none`.
  Registeret inneholder ingen AI-forslag.

`total` er nevneren. Rapporten skal ha nøyaktig så mange rader. Er
`truncated` satt på en samling, snevre inn perioden eller `scope` og
kall igjen til alt er med. Rekker du ikke alle, si det, og merk
rapporten som `delvis gjennomgang` — aldri som ferdig.

## 2. Sjekk per objekt

For hvert **kjøp / bilag med dokument** (`evidence: document`):

- `get_file(fileId)` → les dokumentet (lenken, eller bildet inline med
  `includeContent: true`). Kan du ikke lese det, er objektet `ukjent`.
- `get_voucher` → linjene. Dato, leverandør, totalbeløp, MVA-beløp og
  sats stemmer med dokumentet
- konto er rimelig for det dokumentet viser
- MVA-kode stemmer med selskapets MVA-status og dokumentets sats
- ingen dublett (samme leverandør, beløp og dato et annet sted i
  registeret)

For bilag med `evidence: bank_statement_only`: banklinjen beviser
bevegelsen, ikke formålet. Er kontering og MVA-fradrag likevel satt, er
det `trenger deg` med mindre bilagsmalen (gebyr, renter, overføring)
forklarer det.

For bilag med `evidence: none`: `ledger_only`. Finnes det et
førsteklasses verktøy som burde vært brukt i stedet? Er beskrivelsen
forståelig? Uten dokument er det aldri `ok`.

For hver **banklinje** (`state`):

- `matched`: matchen gir mening — beløp og motpart stemmer med det den
  er koblet til (`matchedVoucherId` / `matchedPurchaseId` /
  `matchedInvoiceId`)
- `ignored_with_reason`: begrunnelsen forklarer faktisk linjen
- `awaiting_document`: står som åpen oppgave — `trenger deg`
- `unbooked`: ikke bokført — `feil` hvis perioden er ment lukket,
  ellers `advarsel`
- overføringer mellom egne kontoer (`isInternalTransfer`) er ikke ført
  som kostnad eller inntekt

For hvert **innbokselement**: er det behandlet (`voucherId`), avvist
med grunn, eller ligger det fortsatt der mens perioden skal lukkes?

Per felt noterer du proveniens: `bekreftet mot dokument`,
`kun i regnskapet`, `støttet av banklinje`, `forklart av bruker`.

## 3. Status

Hvert objekt får én status:

- `ok` — alle sjekker bekreftet mot kilde
- `advarsel` — stemmer, men noe bør ses på (uvanlig konto, svak
  beskrivelse, sen dato)
- `feil` — avvik mellom dokument og regnskap, eller regelbrudd
- `trenger deg` — kan bare avgjøres av brukeren (formål, privat/næring,
  manglende dokument)
- `ukjent` — kilden mangler eller er uleselig

## 4. Aggregater som støtte, ikke erstatning

`get_bank_reconciliation`, `get_vat_report` og `get_trial_balance` viser om
totalene henger sammen. De beviser ikke at hvert bilag er riktig. Bruk dem
til å finne hvor du skal se nøyere, og ta dem med i rapporten.

## Rapport

Skriv for et menneske, på brukerens språk:

1. Scope, nevner, og om gjennomgangen er komplett eller delvis.
2. Tabell med én rad per objekt: id, dato, beløp, status, funn.
3. Funnene som krever handling, gruppert: `feil` først, så
   `trenger deg`, så `ukjent`. For hver: hva kilden sier, hva Dobbel
   sier, og anbefalt rettelse (`post_correction_voucher`, nytt kjøp,
   `request_missing_document` …).
4. Aggregater: bank ties / ties ikke, MVA-grunnlag rimelig / ikke.

Foreslå rettelser. Utfør dem ikke i denne skillen. Vil brukeren rette,
gjør det som egen oppgave med `/dobbel:bokfor-innboks` eller
`/dobbel:avstem-og-lukk`, der godkjenningsreglene gjelder.
