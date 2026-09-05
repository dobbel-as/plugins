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

- Bruk kun lesende verktøy: `get_*`, `list_*`, `search_vouchers`,
  `get_bank_reconciliation`, `get_vat_report`, rapportene.
- Ikke gjenbruk AI-forslaget fra `get_inbox_item` som fasit. Det er
  kandidaten du kontrollerer. Les originaldokumentet selv.
- Har en annen agent bokført i perioden, ikke les dens resonnement som
  kilde. Bilaget står på egne ben eller ikke.
- Ingen vesentlighetsgrense med mindre brukeren har gitt deg én. Uten
  terskel er et avvik et avvik.

## 1. Avgrens og tell

Bestem scope: periode og omfang (alt, bank eller kjøp). Hent registeret:

- `search_vouchers` for perioden → alle bilag
- `list_bank_transactions` for perioden → alle banklinjer
- `list_inbox_items` → dokumenter i perioden, også ubehandlede
- `list_purchases` for perioden

Tell. Nevneren er antall objekter i scope. Rapporten skal ha nøyaktig så
mange rader. Rekker du ikke alle, si det, og merk rapporten som
`delvis gjennomgang` — aldri som ferdig.

## 2. Sjekk per objekt

For hvert **kjøp / bilag med dokument**:

- dokument finnes og er lesbart (`ledger_only` hvis det mangler)
- dato, leverandør, totalbeløp, MVA-beløp og sats stemmer med dokumentet
- konto er rimelig for det dokumentet viser
- MVA-kode stemmer med selskapets MVA-status og dokumentets sats
- ingen dublett (samme leverandør, beløp og dato et annet sted)

For hver **banklinje**:

- bokført, venter på bilag, ignorert med begrunnelse, eller forklart
- matchen gir mening: beløp og motpart stemmer med det den er koblet til
- overføringer mellom egne kontoer er ikke ført som kostnad eller inntekt

For **bilag uten dokument** (fritt bilag): finnes det et førsteklasses
verktøy som burde vært brukt i stedet? Er beskrivelsen forståelig?

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
