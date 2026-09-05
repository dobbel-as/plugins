---
name: bokfor-innboks
description: Registrer kjøp i Dobbel fra kvitteringer og inngående fakturaer — fra filer brukeren peker på, eller fra dokumenter som allerede ligger i innboksen. Bruk når brukeren vil bokføre kvitteringer, fakturaer eller «det som ligger i innboksen».
argument-hint: "[mappe eller filer] eller «innboksen»"
---

# Bokfør innboksen

Hvert kjøp i Dobbel er konsekvens av ett dokument. Du laster opp, leser,
bekrefter mot originalen, og lager et utkast. Mennesket godkjenner i
appen, med mindre selskapet står på «Gjøre».

Les `/dobbel:dobbel` først hvis du ikke har gjort det i denne samtalen.
Kall `get_context` og noter `autonomy_level`.

## Avgrens scopet

Si høyt hva som er i scope før du begynner: hvilke filer, eller hvilke
innbokselementer (`list_inbox_items` med status ubehandlet). Tell dem.
Rapporten til slutt skal ha like mange rader.

## Per dokument

### Fra fil

1. `prepare_inbox_upload` → signert engangs-URL og `fileId`.
2. Last opp filens bytes med HTTP PUT til URL-en. Dette skjer utenom
   samtalen (curl eller tilsvarende). Ingenting er registrert i Dobbel
   ennå.
3. `register_inbox_upload(fileId)` → innbokselement. AI-lesingen starter i
   bakgrunnen. Kallet er idempotent: samme `fileId` gir samme element.
4. Fortsett som «fra innboksen».

### Fra innboksen

5. `get_inbox_item` → forslag til type (kontantkjøp eller inngående
   faktura), dato, beløp, MVA, leverandør og linjer.
6. **Bekreft mot dokumentet.** Forslaget er en kandidat. Sjekk selv, mot
   bildet eller PDF-en, at totalbeløp, dato, leverandør og MVA-sats
   stemmer. Er et felt uleselig eller i strid med dokumentet, la det stå
   uavklart og spør brukeren. Gjett aldri.
7. Er leverandøren ny: `lookup_brreg` på org.nr eller navn, deretter
   `create_supplier`. Finnes den: `list_suppliers` og bruk id-en.
8. Velg konto. `suggest_account` gir forslag; sjekk `recommendedTemplates`
   først. MVA-kode fra `list_vat_codes` — selskaper uten MVA-registrering
   får aldri fradrag.
9. `create_purchase(companyId, fileId, type, …)` → kjøpsutkast med
   debet/kredit-linjer.
10. På `DRAFT`: stopp her. Si at utkastet venter på godkjenning i Dobbel.
    På `FULL`: `approve_voucher` posterer. Bekreft likevel i chatten når
    beløpet er stort eller konteringen usikker.

## Vanlige feller

- **Dubletter.** Samme faktura kan ligge både i innboksen (e-post) og som
  fil. Sjekk `list_inbox_items` og `list_purchases` på leverandør, beløp og
  dato før du oppretter.
- **Kvittering med flere MVA-satser** (mat og drikke, transport): én linje
  per sats.
- **Utenlandsk leverandør uten norsk MVA:** omvendt avgiftsplikt. Ikke
  gjett kode — spør, eller pek på `preview_reverse_charge_vat`.
- **Privat utlegg** som skal refunderes eieren er fortsatt et kjøp med
  dokument; motkonto er mellomregning, ikke bank.
- **Dokumentet er ikke et kjøp** (kontrakt, tilbud, avtale): ikke opprett
  kjøp. Si hva det er, og at det kan kobles til lån eller investering med
  `set_loan_agreement` / `set_investment_agreement` hvis relevant.

## Rapport

Én rad per dokument i scope:

| Dokument | Leverandør | Dato | Beløp | MVA | Konto | Status |
|---|---|---|---|---|---|---|

Status er `utkast opprettet`, `postert`, `dublett — hoppet over`,
`uavklart — trenger deg` (med hva som mangler) eller `ikke et kjøp`.
Ingen rad skal mangle.
