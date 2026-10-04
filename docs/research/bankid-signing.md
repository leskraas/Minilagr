# BankID-signering: Idura (tidl. Criipto) eller Signicat?

Research for [issue #4](https://github.com/leskraas/Minilagr/issues/4). Kildene ble lest 4. oktober 2026. Alle priser er uten mva. Påstander jeg ikke kunne bekrefte i en primærkilde er merket **(ikke verifisert)**.

## Kort svar

**Idura er det beste valget for Minilagr.** Idura er det nye navnet på Criipto. Navnet ble byttet 13. november 2025, og API-ene er de samme som før ([Idura](https://idura.eu/blog/criipto-is-now-idura)). Idura har publiserte priser med selvbetjening og gratis testing. Signerte dokumenter leveres som PAdES-LTA, og signering med norsk BankID gir QES som standard. Idura har også en oppdatert Node-SDK for signering. Siden 2024 eies Idura av Stø (BankID BankAxept), altså selskapet som utsteder BankID ([Idura](https://idura.eu/blog/bankid-bankaxept-acquires-criipto)).

Signicat gjør det samme teknisk sett, og har i tillegg arkiv, iframe-flyt og flere protokoller. Ulempen er at prisene er skjult bak avtale eller dashboard, at onboarding går via kontrakt og bank, og at Signicat ikke har noen offisiell Node-SDK.

Begge løsningene kan brukes til innlogging på sluttkundens selvbetjeningsside med samme leverandørkonto som signeringen. Hos Idura er innlogging (Verify) og signering (Signatures) to separate abonnementer.

Å koble seg direkte på BankID OIDC er ikke et reelt alternativ. Stø selger bare via partnere eller forhandlere, og et forhandleravtale koster NOK 100 000 i etablering og NOK 8 300 per måned ([Stø](https://stoe.no/en/services/id/pricing), [BankID dev](https://developer.bankid.no/bankid-oidc-provider/resources/provisioning/)).

## Sammenligning

| | **Idura** (tidl. Criipto) | **Signicat** | **Direkte mot BankID (Stø)** |
|---|---|---|---|
| Pris per signatur | Abonnement fra €139/mnd for 200 signaturer, pluss BankID QES-gebyr på €0,42 per signatur ([pris](https://idura.eu/pricing/signatures)) | Ikke publisert. Pris består av etablering, abonnement og transaksjoner ([pris](https://www.signicat.com/pricing)) | NOK 4,46 per signering, betalt av forhandler ([Stø](https://stoe.no/en/services/id/pricing)) |
| Pris per innlogging | Abonnement fra €67/mnd for 1 000 innlogginger, pluss €0,089 (biometri) eller €0,126 (BankID High) ([pris](https://idura.eu/pricing/verify)) | Ikke publisert | NOK 0,95 (biometri) eller NOK 1,34 (High). Fra 1.10.2026 får nye kunder fast pris per bruker for autentisering ([Stø](https://stoe.no/en/services/id/pricing)) |
| PDF-signering | Ja. GraphQL-API, flere signatarer ([docs](https://docs.idura.app/signatures/)) | Ja. Sign API v2 (REST), flere signatarer ([docs](https://developer.signicat.com/identity-methods/nbid/integration-guide/sign-nbid/)) | Ja. CSC- og WYSIWYS-API ([BankID dev](https://developer.bankid.no/bankid-esign-provider/getting-started/)) |
| Signaturnivå med norsk BankID | QES som standard ([Idura](https://idura.eu/blog/norwegian-bankid-qes)) | QES med `PKISIGNING`. Autentiseringsbasert signering gir *ikke* QES ([docs](https://developer.signicat.com/docs/electronic-signing/sign-api-v2/signing-methods/)) | QES (Stø er kvalifisert tillitstjenesteleverandør, ifølge [Idura](https://idura.eu/blog/norwegian-bankid-qes)) |
| Format | PAdES-LTA ([docs](https://docs.idura.app/signatures/)) | DSS-signert PAdES for PKI. XAdES pakket i PAdES for autentiseringsbasert signering ([docs](https://developer.signicat.com/docs/electronic-signing/sign-api-v2/signing-methods/)) | PAdES ([BankID dev](https://developer.bankid.no/bankid-oidc-provider/api/signing/signdoc-pades/)) |
| Lagring hos leverandør | Kryptert i Azure i EU. Slettes ved `closeSignatureOrder`, eller opptil 7 dager senere ([docs](https://docs.idura.app/signatures/getting-started/document-lifecycle/)) | Til `dueDate` (maks 45 dager), deretter 7 dager karens og 30 dager i backup ([docs](https://developer.signicat.com/docs/electronic-signing/sign-api-v2/features/document-retention-and-deletion/)) | Minilagr lagrer selv |
| Node/Next.js | `@criipto/signatures` (Node-SDK), `@criipto/verify-express` og `@criipto/auth-js` ([npm](https://www.npmjs.com/package/@criipto/signatures)). Ingen egen Next.js-guide | Ingen offisiell Node-SDK funnet. Standard OIDC og REST | Auth.js har en ferdig `bankid-no`-provider ([Auth.js](https://authjs.dev/getting-started/providers/bankid-no)) |
| Testmiljø | Gratis, uten tidsbegrensning ([pris](https://idura.eu/pricing/signatures)) | Gratis sandkasse ([pris](https://www.signicat.com/pricing)) | Preprod-miljø, men krever partner |
| Innlogging på selvbetjeningssiden | Ja, med Idura Verify (OIDC) | Ja, med eID Hub (OIDC, SAML, REST) | Ja, med OIDC |
| Oppstart i produksjon | Bestilles i dashboardet. Krever norsk selskap. Ca. 10–13 virkedager ([docs](https://docs.idura.app/verify/e-ids/norwegian-bankid/)) | Avtale med Signicat og brukerstedssertifikat via egen bank ([docs](https://developer.signicat.com/identity-methods/nbid/integration-guide/prerequisites/)) | Bare via partner ([BankID dev](https://developer.bankid.no/bankid-oidc-provider/resources/provisioning/)) |

## Detaljer

### Navn og eierskap

- Criipto byttet navn til Idura 13. november 2025. Ifølge Idura er produkter, API-er, avtaler og priser uendret ([Idura](https://idura.eu/blog/criipto-is-now-idura)). Dokumentasjonen ligger nå på `docs.idura.app`, men npm-pakkene heter fortsatt `@criipto/*` ([npm](https://registry.npmjs.org/@criipto/signatures)).
- BankID BankAxept (nå Stø) kjøpte Criipto 26. september 2024. Selskapet drives videre som et uavhengig selskap ([Idura](https://idura.eu/blog/bankid-bankaxept-acquires-criipto)).
- Fra 1. mai 2026 er Stø AS eneste utsteder av norsk BankID. BankID Server og SEID-SDO-formatet er faset ut til fordel for OIDC og PAdES ([Signicat](https://www.signicat.com/about/norwegian-bankid-sto-changes-and-their-effects-on-signicat-solutions)). Både Idura og Signicat oppgir at de støtter den nye modellen.

### Priser

**Idura Signatures** ([kilde](https://idura.eu/pricing/signatures)):
- Small: €139/mnd for 200 signaturer.
- Medium: €209/mnd for 500 signaturer.
- Large: €348/mnd for 1 000 signaturer.
- Enterprise: fra 1 500 signaturer, pris etter avtale.
- I tillegg kommer eID-gebyret "NO BankID QES Signing" på €0,42 per signatur.
- Årlig betaling gir 10 % rabatt. Ved selvbetjening med kort er det ingen onboarding-avgift. Komplekse prosjekter kan få en engangsavgift.
- Regneeksempel: Med Small fullt utnyttet blir det ca. €0,70 + €0,42 ≈ €1,12 per signatur. Prisen for signaturer utover det som er inkludert vises ikke på siden **(ikke verifisert)**.

**Idura Verify** ([kilde](https://idura.eu/pricing/verify)):
- Small: €67/mnd for 1 000 innlogginger.
- Medium: €220/mnd for 5 000 innlogginger.
- Large: €400/mnd for 10 000 innlogginger.
- I tillegg kommer BankID-gebyret: €0,089 per innlogging med biometri, €0,126 med BankID High.
- Verify og Signatures er to separate abonnementer.

**Signicat** ([kilde](https://www.signicat.com/pricing)):
- Signicat sier selv at prisen er "a combination of setup, subscription and transaction fees". Detaljerte priser vises først i dashboardet når man har opprettet en gratis konto, eller fås fra salg.
- Tredjepartssider nevner planer fra €49/mnd **(ikke verifisert i primærkilde)**.

**Stø (BankID-utsteder)** ([kilde](https://stoe.no/en/services/id/pricing)):
- Priser fra 1. januar 2026, fakturert til forhandler:
  - Autentisering med BankID High: NOK 1,34.
  - Autentisering med biometri: NOK 0,95.
  - Signering: NOK 4,46.
  - Alle prisene inkluderer fødselsnummer.
- Fra 1. oktober 2026 får nye kunder fast pris per bruker for autentisering. Eksisterende kunder går over 1. januar 2027. Signering prises fortsatt per transaksjon.
- Selve prisen per bruker er ikke publisert **(ikke verifisert)**. Det er heller ikke klart hvordan modellen påvirker Iduras og Signicats priser til sine kunder **(ikke verifisert)**.

### PDF-signering og signert dokument

**Idura:**
- Flyten er tre kall:
  1. `createSignatureOrder` laster opp PDF-ene.
  2. `addSignatory` gir en signeringslenke per signatar.
  3. `closeSignatureOrder` returnerer de signerte PDF-ene ([Node-SDK](https://docs.idura.app/signatures/integrations/nodejs/)).
- Bevis fra eID-en bygges inn i PDF-en og forsegles med Iduras EUTL-sertifikat. Ved lukking får dokumentet et tidsstempel for langtidsgyldighet ([livssyklus](https://docs.idura.app/signatures/getting-started/document-lifecycle/)). Resultatet er PAdES-LTA ([docs](https://docs.idura.app/signatures/)).
- Dokumentene slettes når ordren lukkes, med mindre `retainDocumentsForDays` (1–7) er satt. Minilagr må derfor lagre den signerte leieavtalen selv.
- Webhooks finnes, for eksempel for "signatar har signert" ([docs](https://docs.idura.app/signatures/)).

**Signicat Sign API v2:**
- Flyten er: last opp dokument, lag dokumentsamling, opprett signeringsøkt, send brukeren til signering og hent resultatet ([NBID-guide](https://developer.signicat.com/identity-methods/nbid/integration-guide/sign-nbid/)).
- For QES må `signingFlow: PKISIGNING` og `vendor: NBID` settes. Det gir en DSS-signert PAdES.
- Begrensninger:
  - PKI-signering krever LoA high, altså ikke biometri.
  - Bare norsk språk støttes.
  - Signataren kan bare valideres på fødselsnummer.
- Flere signatarer kan signere i vilkårlig rekkefølge. Man kan ikke blande PKI-signering og autentiseringsbasert signering i samme dokumentsamling.
- Autentiseringsbasert signering gir XAdES per dokument per signatar, som kan pakkes til PAdES. Dette er *ikke* QES ([signeringsmetoder](https://developer.signicat.com/docs/electronic-signing/sign-api-v2/signing-methods/)).
- Dokumenter lever til `dueDate` (standard 30 dager, maks 45), og blir deretter slettet ([retention](https://developer.signicat.com/docs/electronic-signing/sign-api-v2/features/document-retention-and-deletion/)). Signicat har et eget arkivprodukt, men det har jeg ikke undersøkt.
- Signicat støtter også virksomhetssignering ("merchant signing", AES) med et eget BankID B2B-oppsett.

Jeg har ikke funnet ut om Idura bruker biometri eller krever BankID High for QES-signering **(ikke verifisert)**.

### Utvikleropplevelse

**Idura:**
- Signering bruker GraphQL. Det finnes en offisiell Node-SDK, `@criipto/signatures` (v1.35.1, publisert juni 2026) ([npm](https://registry.npmjs.org/@criipto/signatures)).
- Innlogging bruker standard OIDC. Det finnes `@criipto/auth-js`, `@criipto/verify-express` (Passport) og `@criipto/verify-react` ([integrasjoner](https://docs.idura.app/verify/integrations/)).
- Det finnes ingen egen Next.js-guide. Siden Verify er standard OIDC, bør Auth.js med en generisk OIDC-provider fungere **(ikke testet)**.
- Testbrukere lages i BankIDs preprod-RA ([docs](https://docs.idura.app/verify/e-ids/norwegian-bankid/)).

**Signicat:**
- Tilbyr REST (Sign API v2, Authentication REST API), OIDC (inkludert CIBA og iframe) og SAML ([NBID](https://developer.signicat.com/identity-methods/nbid/), [forberedelser](https://developer.signicat.com/identity-methods/nbid/integration-guide/prerequisites/)).
- Jeg fant ingen offisielle npm-pakker under `@signicat`. Pakken `signicat-oidc-client` er laget av en tredjepart og ble sist oppdatert for omtrent 6 år siden ([npm](https://www.npmjs.com/package/signicat-oidc-client)).
- Gratis sandkasse med forhåndslagde testbrukere.

### Samme løsning for innlogging på selvbetjeningssiden

Begge leverandørene tilbyr OIDC-innlogging med norsk BankID i samme konto som signeringen:
- Idura Verify. Fødselsnummer hentes med `ssn`-scope etter samtykke ([docs](https://docs.idura.app/verify/e-ids/norwegian-bankid/)).
- Signicat eID Hub.

Signicat krever dokumentert lovhjemmel (for eksempel hvitvaskingsloven) for å utlevere fødselsnummer ([Signicat](https://developer.signicat.com/identity-methods/nbid/integration-guide/prerequisites/)). Idura-dokumentasjonen nevner ikke et slikt krav **(ikke verifisert)**.

Hvis det er nok å kjenne igjen sluttkunden, er biometrisk innlogging den billigste varianten. Det er en egen avgjørelse om Minilagr trenger fødselsnummer for innlogging.

## Åpne spørsmål for Minilagr

1. **Hvem er brukersted i BankID?** BankID-appen viser navnet på brukerstedet. Signicat ber om ett visningsnavn per avtale ([Signicat](https://developer.signicat.com/identity-methods/nbid/integration-guide/prerequisites/)). Det er ikke avklart om hver operatør kan vises med eget navn under Minilagrs avtale, eller om sluttkunden ser "Minilagr" **(ikke verifisert, spør leverandørene)**.
2. **Trenger leieavtalen QES?** Begge leverandørene tilbyr QES med norsk BankID. Jeg har ikke vurdert om lavere nivå (AES) er juridisk tilstrekkelig for en leieavtale for bod **(ikke vurdert)**.
3. **Lagring av signert avtale.** Ingen av leverandørene er et arkiv. Minilagr må lagre PAdES-filen selv per operatør, uten at operatørenes data blandes.

## Kilder

- Idura: [navnebytte](https://idura.eu/blog/criipto-is-now-idura), [oppkjøp](https://idura.eu/blog/bankid-bankaxept-acquires-criipto), [QES med norsk BankID](https://idura.eu/blog/norwegian-bankid-qes), [pris Signatures](https://idura.eu/pricing/signatures), [pris Verify](https://idura.eu/pricing/verify), [Signatures-docs](https://docs.idura.app/signatures/), [dokumentlivssyklus](https://docs.idura.app/signatures/getting-started/document-lifecycle/), [Node-SDK](https://docs.idura.app/signatures/integrations/nodejs/), [norsk BankID i Verify](https://docs.idura.app/verify/e-ids/norwegian-bankid/), [integrasjoner](https://docs.idura.app/verify/integrations/)
- Signicat: [pris](https://www.signicat.com/pricing), [Stø-endringer](https://www.signicat.com/about/norwegian-bankid-sto-changes-and-their-effects-on-signicat-solutions), [NBID-oversikt](https://developer.signicat.com/identity-methods/nbid/), [forberedelser](https://developer.signicat.com/identity-methods/nbid/integration-guide/prerequisites/), [Sign API for NBID](https://developer.signicat.com/identity-methods/nbid/integration-guide/sign-nbid/), [signeringsmetoder](https://developer.signicat.com/docs/electronic-signing/sign-api-v2/signing-methods/), [dokumentlagring](https://developer.signicat.com/docs/electronic-signing/sign-api-v2/features/document-retention-and-deletion/)
- Stø / BankID: [priser](https://stoe.no/en/services/id/pricing), [kom i gang](https://stoe.no/en/services/id), [OIDC-provisjonering](https://developer.bankid.no/bankid-oidc-provider/resources/provisioning/), [eSign-API](https://developer.bankid.no/bankid-esign-provider/getting-started/), [PAdES-signering](https://developer.bankid.no/bankid-oidc-provider/api/signing/signdoc-pades/)
- Øvrig: [Auth.js BankID Norge-provider](https://authjs.dev/getting-started/providers/bankid-no), npm: [@criipto/signatures](https://www.npmjs.com/package/@criipto/signatures), [signicat-oidc-client](https://www.npmjs.com/package/signicat-oidc-client)
