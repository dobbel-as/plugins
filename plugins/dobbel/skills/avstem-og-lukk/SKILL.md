---
name: avstem-og-lukk
description: Bokfør banktransaksjonene i Dobbel, forklar det som gjenstår, og lukk bankavstemmingen for perioden når portene er åpne. Bruk når brukeren vil avstemme banken, bokføre kontoutskriften, eller gjøre måneden ferdig.
argument-hint: "[bankkonto] [periode, f.eks. 2026-08]"
---

# Avstem og lukk

Poenget med en avstemming er at hver bevegelse i banken enten er bokført
eller forklart. Ikke null gjenstående for enhver pris — etterprøvbar
forklaring. En banklinje beviser at penger flyttet seg, ikke hvorfor.

Les `/dobbel:dobbel` først hvis du ikke har gjort det i denne samtalen.
Kall `get_context` og noter `autonomy_level` og om `bank:reconcile` er
gitt.

## 1. Hent grunnlaget

`list_bank_accounts` → velg konto. `get_bank_reconciliation(companyId,
bankAccountId, periode)` gir:

- inngående og utgående saldo på begge sider (regnskap og bank)
- differansen dekomponert i årsaker
- `lines.booked`, `lines.suggested`, `lines.unbooked`, `lines.explained`
- `closeReadiness.blockers` — det som må bort før perioden kan lukkes

`lines.unbooked` er jobben. Tell dem; rapporten skal ha like mange rader.

## 2. Per ubokført linje, i denne rekkefølgen

1. **Gjør den opp et bokført kjøp?** `list_purchases` på beløp og
   leverandør → `commit_bank_match(purchaseId)`.
2. **Gjør den opp en kundefaktura?** `list_invoices` på beløp og KID →
   `commit_bank_match(invoiceId)`.
3. **Finnes bilaget allerede** (lønn, MVA-oppgjør, manuelt bilag)?
   `search_vouchers` → `commit_bank_match(voucherId)`.
4. **Bankgebyr, renter, overføring mellom egne kontoer?**
   `commit_bank_match(counterAccountNumber)`, eller `create_voucher` med
   bilagsmal (`list_voucher_templates`). Gebyr hører på 7770, renter på
   8050/8150.
5. **Mangler dokumentasjonen?** `request_missing_document`. Linjen sperres
   for bokføring til dokumentet finnes, og teller som forklart i
   avstemmingen. Dette er riktig valg når du *vet* det er et kjøp men ikke
   har bilaget. Gjett aldri kostnadskonto for å få det til å gå opp.
6. **Skal aldri i regnskapet** (privat utlegg gjort opp utenom, feil
   konto)? `ignore_bank_transaction` med begrunnelse.
7. **Uklart?** Stopp og spør. Legg linjen i rapporten som `uavklart`.

`lines.suggested` er forslag som venter på bekreftelse. Bekreft dem med
brukeren før `commit_bank_match`, ikke automatisk.

## 3. Forklar det som gjenstår

Reelle poster som ikke kan matches ennå (sjekk i posten, betaling som
ligger i banken men ikke i regnskapet) forklares med
`explain_reconciliation_difference`. Oppgi nøyaktig én av
`bankTransactionId` eller `voucherId`. Forklaringen blir del av
avstemmingsrapporten etter bokføringsloven § 11.

## 4. Lukk perioden

Kall `get_bank_reconciliation` på nytt. Er `closeReadiness.blockers` tom,
og har du `bank:reconcile` og nivå `FULL`, kan `close_bank_reconciliation`
kalles. Lukkingen er en godkjenning: den låser perioden for videre
bilagsføring mot kontoen, og bare en admin kan gjenåpne. Bekreft derfor
med brukeren i chatten før du kaller den, selv på `FULL`.

Mangler du scope eller nivå: si at perioden er klar til lukking, og at
brukeren lukker den under Bank i Dobbel.

## Rapport

| Dato | Beløp | Tekst | Handling | Referanse |
|---|---|---|---|---|

Handling er `bokført mot kjøp`, `bokført mot faktura`, `koblet til bilag`,
`bokført mot konto NNNN`, `venter på bilag`, `ignorert (begrunnelse)`,
`forklart` eller `uavklart — trenger deg`. Avslutt med saldo begge sider,
gjenstående differanse og om perioden ble lukket.
