# Stripe Connect for operatører

Svar på issue #5: «Hvordan bør Stripe Connect settes opp for operatører?»
Undersøkt 2026-10-04 mot docs.stripe.com og stripe.com/no. Priser og API-detaljer endrer seg, så sjekk kildene før dere bygger.

## Anbefaling

1. **Én tilkoblet konto (connected account) per operatør, opprettet med Accounts v2-API-et.** Kontotypene Standard, Express og Custom er nå merket som utgåtte (deprecated) og gjelder bare plattformer som bruker dem fra før. Nye plattformer skal bruke Accounts v2. Der v2 mangler noe, brukes v1 med controller properties. [1][2]
2. **Direct charges.** Leieavtalen er mellom sluttkunden og operatøren, så operatøren bør være merchant of record. Stripe anbefaler direct charges for SaaS-plattformer der de tilkoblede kontoene handler direkte med sine egne kunder. [3][4]
3. **Stripe Billing Subscriptions opprettet på operatørens konto** (med `Stripe-Account`-headeren), og ikke egen planlegging. Da får vi Smart Retries, e-poster ved feilet betaling og 3DS-håndtering ferdig levert. Det koster 0,7 % av Billing-volumet. [4][5][6]
4. **Plattformgebyr via `application_fee_percent`** på abonnementet. For faste kronebeløp settes `application_fee_amount` på hver faktura fra en `invoice.created`-webhook. [4]
5. **Kontooppsett for MVP (anbefalt, men ikke endelig): `dashboard: "full"`, `fees_collector: "stripe"`, `losses_collector: "stripe"`.** Med dette betaler operatøren Stripe-gebyrene selv, Stripe bærer risikoen for negativ saldo, og plattformen slipper Connect-gebyrer. Stripe gjør også KYC. Alternativet er Express-dashboard med `fees_collector: "application"`. Det gir mer kontroll og vår egen merkevare, men da betaler plattformen alle Stripe-gebyrer pluss Connect-gebyrer. Se avveiningen under. [7][8][9]
6. **Purring (dunning) styres av Minilagr.** Stripe står for nye trekkforsøk og e-poster. Minilagr lytter på `invoice.payment_failed` og `customer.subscription.updated` (`past_due`/`unpaid`) og sperrer adgangskoden etter egne regler. [5][6]
7. **Bare kort i MVP** fungerer godt med Billing: Smart Retries gjelder kort, og automatiske nye forsøk for lokale betalingsmetoder er av som standard. [5]

## Kontooppsett

### Kontotyper vs. konfigurasjoner

- Siden om kontotyper sier: «The information on this page applies only to platforms that already use legacy connected account types (Standard, Express, or Custom accounts).» Stripe anbefaler controller properties eller Accounts v2 i stedet. [1]
- I Accounts v2 får en `Account` *konfigurasjoner*. `merchant` lar kontoen ta imot betalinger (`card_payments`, `stripe_balance.payouts`). `customer` lar plattformen fakturere kontoen. `recipient` lar kontoen motta overføringer, og trengs for indirect charges. [2]
- Med v2 kan samme `Account` både ta imot leie (merchant) og betale Minilagrs eget abonnement (customer), uten et eget `Customer`-objekt. Det passer hvis plattformeieren senere skal fakturere operatører via Stripe. [10]
- v1 må fortsatt brukes for OAuth, recipient service agreement, treasury og card issuing. Ingen av disse er aktuelle for oss. [2]

### Ansvar og dashboard (låses når kontoen opprettes)

| Felt (v2) | Verdier | Betydning |
| --- | --- | --- |
| `defaults.responsibilities.fees_collector` | `stripe` / `application` | Hvem som betaler Stripe-gebyrer for direct charges. Kan ikke endres senere. [7] |
| `defaults.responsibilities.losses_collector` | `stripe` / `application` | Hvem som bærer negativ saldo. Settes den til `application`, må også `fees_collector` være `application`. [7] |
| `dashboard` | `full` / `express` / `none` | Hvilket Stripe-dashboard operatøren får tilgang til. [7] |
| `requirements_collector` | (beregnes) | Plattformen samler inn KYC bare med `losses_collector=application` og `dashboard=none`. Ellers gjør Stripe det. [7] |

v1 controller properties tilsvarer disse: `controller.fees.payer`, `controller.losses.payments`, `controller.requirement_collection` og `controller.stripe_dashboard.type`. [7]

Stripes anbefalinger for direct charges: dashboard Full, Embedded eller Express, negativ saldo hos Stripe, og gebyrinnkreving hos enten plattformen eller Stripe. [8]

### Avveining: full + stripe vs. express + application

| | A: `full` + `fees_collector=stripe` | B: `express` + `fees_collector=application` |
| --- | --- | --- |
| Stripe-gebyrer (kort, Billing 0,7 %, tvister, 3DS, Radar) | Operatøren betaler direkte [9] | Plattformen betaler og må hente det inn via application fee [9] |
| Connect-gebyrer | Ingen («We don't charge any Connect fees») [9] | 15 kr per månedlig aktiv konto + 0,25 % + 5 kr per utbetaling [11] |
| Hvem som styrer abonnementer | Operatøren kan selv styre kundenes abonnementer i dashboardet [4] | Plattformen må styre alle abonnementer [4] |
| Merkevare | Operatørens Stripe-konto | Express-dashboardet kan få vår merkevare [7] |
| Risiko | Operatøren først, deretter Stripe (`losses_collector=stripe`) [7][9] | Samme hvis `losses_collector=stripe` |

Hvorfor A for MVP: lavest plattformkostnad og minst ansvar. Ulempen er at operatøren får et fullt Stripe-dashboard og kan endre eller kansellere abonnementer utenom Minilagr. Det må vi tåle, eller fange opp via webhooks (`customer.subscription.updated`/`deleted`). Dette er et produktvalg for plattformeieren og bør tas med i en ADR.

## Charge type

| | Direct charge | Destination charge |
| --- | --- | --- |
| Hvor belastningen ligger | Operatørens konto | Plattformen |
| Merchant of record | Operatøren | Plattformen (eller operatøren med `on_behalf_of`) |
| Refusjoner og tvister | Trekkes fra operatørens saldo | Trekkes fra plattformens saldo |
| Stripe-gebyrer | Velges med `fees_collector` | Alltid plattformen |
| Anbefalt for | SaaS, full dashboard | Markedsplasser, Express/Custom |

Kilder: [3][4][12]. Stripe sier: «Direct charges are recommended for connected accounts with access to the full Stripe Dashboard» og «Destination charges are recommended for connected accounts with access to the Express Dashboard or connected accounts without access to a Stripe-hosted dashboard». [4] Direct charges anbefales ikke for de gamle v1 Express- og Custom-kontoene. [3]

Med destination charges havner refusjoner og tvister hos plattformen, og plattformen må prøve å hente pengene tilbake fra operatøren med transfer reversals. [3][8] Det passer ikke når operatøren er utleier.

## Månedlige trekk: Billing vs. egen planlegging

**Billing på operatørens konto (anbefalt):**

```bash
curl https://api.stripe.com/v1/subscriptions \
  -u "$STRIPE_SECRET_KEY:" \
  -H "Stripe-Account: acct_OPERATOR" \
  -d customer=cus_SLUTTKUNDE \
  -d "items[0][price]=price_BOD" \
  -d application_fee_percent=5 \
  -d "expand[0]=latest_invoice.confirmation_secret"
```

- `Customer` og `Price` må finnes på operatørens konto. [4]
- Hver periode lager abonnementet en faktura, og fakturaen lager en belastning. [4]
- Begrensning: bare kontoer med fullt dashboard kan selv styre kundenes abonnementer. For andre kontoer må plattformen gjøre det. [4]
- Kobles kontoen fra plattformen, fortsetter abonnementene på operatørens konto. En `application_fee_percent` blir også krevd inn videre, så den bør fjernes før frakobling. [4]
- Pris: 0,7 % av Billing-volumet (pay-as-you-go) eller fra 6 800 kr/mnd på ettårsavtale. [13]

**Egen planlegging (PaymentIntent med `off_session=true` hver måned):** Sparer Billing-gebyret på 0,7 %, men da må vi selv bygge nye trekkforsøk, kundee-poster, håndtering av `authentication_required` og fakturaer. Kortet må lagres med SetupIntent (`usage=off_session`) eller `setup_future_usage=off_session`. [14] Ikke anbefalt for MVP.

Regneeksempel (ikke en kilde, bare aritmetikk med prisene fra [13]): leie 1 500 kr med norsk kort gir kortgebyr 2,4 % + 2 kr = 38 kr, pluss Billing 0,7 % = 10,50 kr.

## Feilede betalinger og purring

- **Smart Retries:** Stripe velger tidspunkt for nye forsøk ved hjelp av AI. Standard er «8 tries within 2 weeks». Vinduet kan settes til 1, 2 eller 3 uker, 1 måned eller 2 måneder. Alternativt kan man lage en egen plan med opptil 3 nye forsøk. [5]
- **Ingen nye forsøk** ved hard decline (`lost_card`, `stolen_card`, `authentication_required` med flere), når betalingsmetode mangler, eller når Connect-kontoen er koblet fra. [5]
- **Etter siste forsøk** går abonnementet til `canceled`, `unpaid`, forblir `past_due` eller `paused`, avhengig av innstillingene. [5] Stripe sier om `unpaid`: «Revoke access to your product when the subscription is `unpaid`». [6]
- **E-poster:** Stripe kan sende e-post ved hver feilet kortbetaling, en lenke for å bekrefte 3DS, påminnelser, varsel om utløpende kort (1 måned før) og fornyelsesvarsel. E-postene har lenke til en Stripe-hostet side der kunden kan oppdatere kortet. [15] Fakturaer og e-poster kan lokaliseres til norsk via `preferred_locales` / `defaults.locales`. [16] I sandbox sendes ikke e-postene automatisk. [15]
- **Webhooks å lytte på** (Connect-endepunkt, `connect=true`; hendelser fra direct charges har feltet `account`): [6][17]
  - `invoice.paid`: betaling OK.
  - `invoice.payment_failed`: feilet. `attempt_count` viser antall forsøk, og `next_payment_attempt` viser neste.
  - `invoice.payment_action_required`: kunden må gjennom 3DS.
  - `customer.subscription.updated` / `.deleted`: overganger til `past_due`, `unpaid` eller `canceled`.
  - `invoice.upcoming`, `invoice.created`: sistnevnte brukes hvis plattformgebyret er et fast beløp.
  - `account.updated`: krav og status for operatørkontoen.
  - `payout.failed`: utbetaling til operatøren feilet.

## Plattformgebyr (application fee)

- `application_fee_percent` (0–100, maks to desimaler) trekkes én gang per periode av fakturabeløpet etter rabatter, før Stripe-gebyrer. [4]
- Et fast gebyr kan ikke settes som gjentakende gebyr på abonnementet. Bruk i stedet `application_fee_amount` på hver faktura, som overstyrer prosenten. [4]
- `application_fee_percent` gjelder ikke fakturaer laget utenom periodene, for eksempel prorasjoner. [4]
- Med `fees_collector=stripe` trekker Stripe sitt gebyr i tillegg, så application fee skal bare dekke vår andel. Med `fees_collector=application` må application fee dekke både vår andel og Stripes gebyr. [7][9]

## Norge og NOK

- **Tilgjengelighet:** Norge (NO) finnes i Stripes landliste [18] og i listene over land som støttes for Express- og Custom-kontoer [1]. Accounts v2-dokumentasjonen nevner ingen begrensning for Norge. [2]
- **Kortgebyrer (stripe.com/no/pricing):** 2,4 % + 2,00 kr for norske og EU-kort, og 3,25 % + 2,00 kr for britiske og internasjonale kort. Valutaveksling koster i tillegg 2 %. [13]
- **Connect-gebyrer:** Ingen hvis «Stripe handles pricing». Hvis «You handle pricing»: 15 kr per månedlig aktiv konto + 0,25 % + 5 kr per utbetaling. [11]
- **Utbetalinger:** Første utbetaling kommer etter 7 kalenderdager, deretter er standard 3 virkedager. Minste utbetaling er 20 NOK. [19]
- **Valuta:** Direct charges blir alltid gjort opp i operatørens land. [3]
- **SCA/3DS:** SCA gjelder når virksomheten er i EØS (Norge er med i EØS) og kundene er i EØS. [20][21] Ved gjentakende trekk autentiseres kortet én gang mens kunden er til stede. Senere trekk merkes som merchant-initiated transactions (MIT) og krever et mandat i vilkårene, men banken kan fortsatt kreve 3DS. [20] Billing håndterer SCA for abonnementer, og integrasjonen må tåle statusen `incomplete` og `requires_action`. [20][6]

## Ikke verifisert / åpne spørsmål

- **Hvem sine Billing-innstillinger som gjelder.** Det er uklart om innstillingene for Smart Retries og e-poster kommer fra operatørens konto eller fra plattformen for abonnementer med direct charges, og om plattformen kan sette dem via API for kontoer uten fullt dashboard. Dette fant jeg ikke i dokumentasjonen. Bør testes i sandbox eller avklares med Stripe.
- **Gebyrene for Accounts v2** i Norge er hentet fra den generelle Connect-prissiden [11]. Jeg har ikke sett en egen side om v2-priser.
- **Om Billings 0,7 % belastes operatøren** med `fees_collector=stripe` er utledet fra raden «Invoicing and Subscriptions: Connected Account» i [9]. Det er ikke bekreftet med en faktisk faktura.
- **MVA på boligleie/lagerleie og krav til norske fakturaer** (organisasjonsnummer, MVA-merking) er utenfor denne undersøkelsen.
- **Vipps/MobilePay og AvtaleGiro** er ikke undersøkt, siden MVP er bare kort.
- **At Norge er med i EØS** er allment kjent og ikke hentet fra Stripe. Stripe-siden sier bare «EEA». [20][21]

## Kilder

1. Connected account types (legacy): https://docs.stripe.com/connect/accounts
2. Connect and the Accounts v2 API: https://docs.stripe.com/connect/accounts-v2
3. How charges work in a Connect integration: https://docs.stripe.com/connect/charges
4. Create subscriptions with Stripe Billing (Connect): https://docs.stripe.com/connect/subscriptions
5. Automate payment retries (Smart Retries): https://docs.stripe.com/billing/revenue-recovery/smart-retries
6. How subscriptions work: https://docs.stripe.com/billing/subscriptions/overview
7. Configure the behavior of connected accounts (v2): https://docs.stripe.com/connect/accounts-v2/connected-account-configuration
8. Recommended Connect integrations and charge types: https://docs.stripe.com/connect/integration-recommendations
9. Fee behavior on connected accounts: https://docs.stripe.com/connect/direct-charges-fee-payer-behavior
10. SaaS platform configurations for Accounts v1 and v2: https://docs.stripe.com/connect/accounts-v2/saas-platform-payments-billing
11. Connect-priser Norge: https://stripe.com/no/connect/pricing
12. Create invoices with Connect: https://docs.stripe.com/connect/invoices
13. Priser Norge: https://stripe.com/no/pricing
14. Save a payment method without a payment (Setup Intents): https://docs.stripe.com/payments/save-and-reuse?payment-ui=elements
15. Automate customer emails: https://docs.stripe.com/billing/revenue-recovery/customer-emails
16. Customize invoices (språk): https://docs.stripe.com/invoicing/customize
17. Connect webhooks: https://docs.stripe.com/connect/webhooks
18. Stripe global availability: https://stripe.com/global
19. Payouts (settlement timing by country): https://docs.stripe.com/payouts
20. Strong Customer Authentication readiness: https://docs.stripe.com/strong-customer-authentication
21. EFTA, EØS-avtalen: https://www.efta.int/eea
