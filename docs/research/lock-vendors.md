# Låsleverandører for ubetjente minilagre i Norge og Norden

Undersøkelse for issue #3: «Hvilke låsleverandører finnes for ubetjente minilagre i Norge?»
Undersøkt 2026-10-04. Kildene er leverandørsider, API-dokumentasjon og integrasjonssider hos minilager-SaaS. Påstander jeg ikke fant i en primærkilde er merket **Ikke verifisert**.

## Kort svar

- **To leverandører er bevist i drift i Norge i dag:** **Salto** (Salto KS / JustIN Mobile) hos Second Space, og **Nokē** (Janus International) hos Green Storage (tidligere City Self-Storage / OK Minilager). Begge kan styre både port/inngang og den enkelte bod, og begge har API.
- **Nokē, PTI og OpenTech er det de store SaaS-produktene integrerer med først** (Storeganise, Stora, SiteLink). **Salto KS** og **sedisto** er de mest relevante europeiske alternativene. sedisto er laget for minilager og bruker vanlig maskinvare fra markedet.
- **Alle seriøse alternativer virker uten nett i en periode:** Låsene lagrer tilgangsregler lokalt, og telefonen åpner over Bluetooth. Men en *ny* sperring (for eksempel ved purring) kommer ikke frem til låsen før gatewayen er på nett igjen. Det må domenet ta høyde for.
- **Priser er nesten aldri offentlige.** Nokē, PTI, OpenTech, BearBox og sedisto selges bare etter tilbud. De eneste listeprisene jeg fant: Salto KS-abonnement fra 225,08 € per år (inkl. mva., minste nivå) og Neo-sylinder fra 587,97 €, begge hos en forhandler, og igloohome API til 2–5 USD per aktiv lås per måned.
- **Ordet «adgangskode» i GLOSSARY passer ikke rett inn i alle systemer.** Salto KS lager PIN-en selv (vi kan ikke velge den). Nokē bruker app, «quick-click» og brikker på boden, og PIN-tastatur på porten. Minilagr bør derfor modellere en adgangskode som noe leverandøren utsteder, ikke noe vi velger.

## Sammenligning

| Leverandør | Opprinnelse | API for å lage, endre og sperre tilgang | Port, bod eller begge | Uten nett | Pris (offentlig) | SaaS som integrerer | Bevist i Norge |
|---|---|---|---|---|---|---|---|
| **Nokē Smart Entry** (Janus) | USA | Ja, Nokē Core API: offline-nøkler, quick-click-koder, brikker, logg [1] | Begge: Nokē ONE/Ion på boddør, i tillegg til inngang [2] | Ja: offline-nøkler, quick-click og brikker; kontroller lagrer 300 hendelser [1][2] | Nei, kun tilbud [3] | Storeganise, Stora, SiteLink, Space Manager, Storman, Kinnovis, 6Storage m.fl. [4] | Ja, Green Storage / City Self-Storage [5] |
| **Salto KS** | Spania | Ja, KS Connect API: brukere, tilgangsgrupper, PIN, digitale nøkler, offline-nøkler [6] | Begge, via tilgangsgrupper som samler inngang og bod [7] | Ja: PIN, digital nøkkel og brikke virker når låsen mister kontakt med IQ-gatewayen [8] | Abonnement fra 225,08 €/år (S, 15 brukere, Lite); Neo-sylinder fra 587,97 € (forhandler, inkl. mva.) [9][10] | Storeganise (add-on) [7], Sharefox [11] | Ja, Second Space (JustIN Mobile by Salto) [12] |
| **sedisto** | Tyskland | Integrasjon med Stora og Storeganise; ingen offentlig API-dokumentasjon funnet [13][14] | Begge: port, dører, heis og bod [15] | Ja: BLE fra telefon, gateway mellomlagrer rettigheter, LTE-reserve, failover [15][14] | Nei | Stora, Storeganise [13][14] | Ikke verifisert (oppgir Norge som støttet land [13]) |
| **PTI StorLogix Cloud** | USA | Ja via PMS-integrasjon: kode opprettes ved utleie, sperres ved mislighold, kan endres, tidsvindu og tastatur kan velges [16] | Begge: tastatur, port og døralarmer, ProEdge Bluetooth-lås på bod [17] | Ikke verifisert | Nei | SiteLink [16], Storeganise [18], Stora [19] | Ikke verifisert |
| **OpenTech INSOMNIAC CIA** | USA | Ja, «open API» [20] | Port og inngang (tastatur); bod ikke verifisert | Ikke verifisert | Nei | SiteLink, Storeganise, Stora [21][18][22] | Ikke verifisert |
| **BearBox** | Storbritannia/Spania | Integrasjon med Stora og Space Manager; SiteLink «coming soon» [23][24] | Begge: port (kode, QR, skiltgjenkjenning) og BearLock på bod [23][19] | Ikke verifisert | Nei | Stora, Space Manager, Storeganise, SiteLink (snart) [23][18][24] | Ikke verifisert |
| **Paxton Net2** | Storbritannia | Via Stora-integrasjon: tildele og fjerne tilgang, automatisk sperring [25] | Dører og inngang (kortlesere); bod ikke verifisert | Ikke verifisert | Nei («lisensfri programvare» [11b]) | Stora, Storeganise, Sharefox [25][18][11] | Ikke verifisert |
| **igloohome / iglooworks** | Singapore | Ja: PIN og mobilnøkler, utløp, webhooks [26] | Bod (hengelås, sylinder og mer); ikke port | Ja: algoPIN genereres uten kontakt med låsen [26] | API 2 USD (igloohome) / 5 USD (iglooworks) per aktiv lås per måned [26] | Sharefox («Igloo») [11] | Ikke verifisert |
| **RCO R-CARD M5** | Sverige | M5 Admin API og M5 User API, krever avtale med RCO [27] | Dører og inngang (klassisk adgangskontroll) | Ikke verifisert | Nei | Sharefox [11]; mobil via Accessy, Unloc, Inlet [27] | Ikke verifisert for minilager |
| **Unloc** | Norge (Oslo) | Ja: samlet API over flere låsmerker (lager ikke låser selv) [28] | Avhenger av låsen under | Ikke verifisert | Nei | Ingen minilager-SaaS funnet | Ikke verifisert for minilager |

Tallene i hakeparentes viser til kildelisten nederst.

## Detaljer

### Nokē Smart Entry (Janus International)

- **Produkter:** Nokē ONE (batteridrevet lås på boddøren), Nokē Ion (kablet lavspenning i dørkarmen), Nokē Retrofit for eksisterende dører og Nokē hengelås [2].
- **Tilgang:** mobilapp (kan få operatørens merkevare), PIN-tastatur og «quick-click» [2].
- **Nett:** Låsene kobles til skyen gjennom et mesh-nett. Bluetooth 4.0 med 128-bit AES-CCM [2].
- **API (Nokē Core API):** Bearer-token. Endepunktene er `/unlock`, `/keys` (utstede og trekke tilbake offline-nøkler), `/qc/issue|revoke` (opptil 100 quick-click-koder per lås), `/fobs/issue|revoke|sync` og `/activity` [1].
- **Uten nett:** En offline-nøkkel og en åpne-kommando lagres på telefonen og kan brukes senere uten nett. **Begrensning:** quick-click-koder legges til eller fjernes bare ved en *online* opplåsing [1]. En sperring kan altså bli forsinket.
- **Pris:** Ikke offentlig. Janus skriver at prisen avhenger av antall boder og forholdene på stedet, og ber deg kontakte en representant [3].
- **I Norge:** Green Storage (City Self-Storage) bruker appen «Green Storage Access by Nokē» til både inngang og bod [5].

### Salto KS

- **Modell:** Brukere legges i tilgangsgrupper. En gruppe samler låser, for eksempel inngangsdøren pluss én bod [7].
- **API (KS Connect API):** OAuth2 (`password`-flyt for server-til-server med en site_admin-systembruker). Dekker brukere, tilgangsgrupper, digitale nøkler, låser, PIN-koder, offline-nøkler og hendelseslogg [6]. Klient-ID og hemmelighet får du fra din lokale Salto-avdeling [29]. Om dette krever en egen partneravtale eller koster noe er **ikke verifisert**.
- **PIN:** Ifølge Seam sin integrasjonsdokumentasjon lager Salto KS PIN-koden selv. Du kan ikke angi en egendefinert kode [30] (sekundærkilde, ikke bekreftet hos Salto).
- **Sperring:** I Storeganise-integrasjonen flagges boden ved forfall, og det utløser en overlock i Salto. PIN-en er fortsatt gyldig, men åpner ikke. Boden åpnes automatisk igjen når kunden har betalt [7].
- **Uten nett:** Hvis IQ-gatewayen mister internett men fortsatt når låsene, håndhever den reglene selv. Hvis låsen ikke når IQ, faller den tilbake på lokalt lagrede data. PIN har offline-tilgang automatisk; digital nøkkel og brikke må slås på. Endringer krever at IQ er på nett [8].
- **Pris:** Abonnementet prises etter antall brukere og nivå (Lite eller Pro) [31]. Hos forhandleren digital-key-world: årsvoucher fra 225,08 € inkl. mva. (minste nivå, opptil 15 brukere), med nivåer opp til 2 000 brukere [9]. XS4 Neo-sylinder fra 587,97 € inkl. mva. (uten tastatur, bare RFID/NFC/BLE) [10]. Prisen per bod og prisen for låser med PIN-tastatur er **ikke verifisert**.
- **I Norge:** Second Space bruker JustIN Mobile (by Salto) til bygget. Noen steder brukes appen også til boden; andre steder har boden en vanlig kodelås [12].

### sedisto

- Laget for minilager. Programvare, Bluetooth-modul og bygningsgateway på «vanlig maskinvare fra markedet», slik at operatøren ikke låses til én leverandør [15][32].
- **Tilgang:** app (BLE), PIN-tastatur, NFC-kort og QR, til port, dører, heis og bod [15].
- **Uten nett:** Gatewayen mellomlagrer rettigheter og har LTE-reserve og failover. Storeganise beskriver løsningen som «Online AND offline» med redundans «even without internet, without power» [15][14].
- **Integrasjoner:** Stora (Norge er oppført som støttet land) og Storeganise (innflytting, utflytting, overlock, PIN og app) [13][14].
- **API og pris:** ikke offentlig.

### PTI Security Systems (StorLogix Cloud)

- Kontroller (CloudController eller FalconXT), AP1-tastaturer, døralarmer og ProEdge Bluetooth-låser på bod, med StorID-app [17].
- SiteLink-integrasjonen: Kunden får tilgangskode automatisk ved utleie og blir sperret ved mislighold. Koden kan endres, og tidsvindu og hvilke tastaturer som gjelder kan styres [16].
- Oppførsel uten nett og pris: **ikke verifisert**.

### OpenTech Alliance (INSOMNIAC CIA)

- Skybasert adgangskontroll for minilager uten PC på stedet, med «open API» [20]. Har gått inn i det europeiske markedet [21]. Integrert med SiteLink, Storeganise og Stora [21][18][22].
- Bod-låser, oppførsel uten nett og pris: **ikke verifisert**.

### BearBox

- Port (kode, QR, skiltgjenkjenning) og BearLock på bod, som åpner bare for betalende kunder og låser automatisk ved forfall [23][19]. Integrert med Stora og Space Manager [23]. SiteLink-integrasjonen er merket «coming soon» [24].
- Stora omtaler BearBox som utbredt i Norden [19] (sekundært, Stora er partner). Oppførsel uten nett og pris: **ikke verifisert**.

### Andre som er nevnt

- **Paxton Net2:** Stora-integrasjon med automatisk sperring av kunder som ikke betaler (krever Net2 v6.9 eller nyere) [25].
- **igloohome / iglooworks:** algoPIN lages med algoritme uten kontakt med låsen, altså fullt offline og uten gateway [26]. Ulempen er at det er uklart om en allerede utstedt PIN kan trekkes tilbake før den utløper uten å besøke låsen (**ikke verifisert**).
- **RCO R-CARD M5:** Svensk adgangskontroll. Krever integrasjonsavtale med RCO, og API-et kan ikke kjøpes direkte fra forhandler [27].
- **Unloc:** Norsk plattform som samler flere låsmerker bak ett API og samarbeider med Salto Systems Nordic [28][33]. Bruk i minilager er ikke funnet.
- **Sharefox** (norsk utleie-SaaS med minilagermodul) lister integrasjoner med ASSA ABLOY, Danalock, dormakaba, EVVA, Igloo, iLOQ, Metra, Livion, Lockifi, Nimly, Nuki, Paxton, RCO, Salto, Inlet og Smartlocks [11]. Det viser hva som finnes, men ikke hva som er utbredt.

## Hva de store SaaS-produktene integrerer med

| SaaS | Adgangsintegrasjoner (fra egen side) |
|---|---|
| **Storeganise** [18] | OpenTech, Nokē, SpiderDoor, PTI, Salto KS, Sensorberg, BearBox, sedisto, StorAxxS, LockVue, Entryfy, Tapkey, Paxton, keynexis |
| **Stora** [22][34][19][13][25] | Egne BearBox og mSpaceLock, Keep It Simple Storage, Nokē, sedisto, Paxton Net2, OpenTech, PTI, Sensorberg, SpiderDoor |
| **SiteLink (Storable)** [24][35] | Access Control by Storable, SpiderDoor, INSOMNIAC CIA, PTI, BearBox (snart), Sentinel, Stor-Guard, samt Nokē (fra Janus' liste) [4] |
| **Sharefox** [11] | Se listen over |

**Nokē** er den eneste leverandøren som er integrert i alle tre store produktene og i tillegg er i drift i Norge. **PTI** og **OpenTech** finnes i alle tre, men er rettet mot USA. Ingen norsk minilager i denne undersøkelsen bruker dem.

## Konsekvenser for Minilagr (forslag, ikke besluttet)

- Lag et **låsadapter per leverandør** med operasjoner som «gi tilgang», «sperr», «gjenåpne» og «trekk tilbake». Begynn med Nokē og Salto KS, siden begge er i drift i Norge.
- **Sperring er asynkron.** Den kan ta tid å nå frem uten nett, og noen systemer (Salto via overlock) sperrer uten å ugyldiggjøre koden. Purring bør registrere at sperringen er *bestilt* og at den er *bekreftet* som to separate hendelser.
- **Adgangskoden kan være generert av leverandøren** (Salto) eller være noe annet enn en sifferkode (Nokē-app, quick-click). Det bør vurderes mot definisjonen i GLOSSARY.

## Ikke verifisert

- Priser for Nokē, PTI, OpenTech, BearBox, sedisto, Paxton og Unloc (ikke offentlige).
- Salto KS: pris per bod, låser med PIN-tastatur, og om tilgang til API-et koster noe.
- At Salto KS ikke tillater egendefinert PIN (bare sekundærkilde [30]).
- Oppførsel uten nett for PTI, OpenTech, BearBox og Paxton.
- Om sedisto, BearBox, PTI eller OpenTech har kunder i Norge.
- Hvilken leverandør som står bak Pelican Self Storage sin app (Pelican Smart Access); siden oppgir det ikke [36].
- Hvilken låsleverandør Flexistore bruker; siden oppgir det ikke [37].

## Kilder

1. Nokē Core API: https://github.com/noke-inc/noke-core-api-documentation
2. Nokē-oversikt (Janus Europe): https://januseurope.com/noke/
3. Janus, «How Much Does Smart Entry Cost Per Unit?»: https://www.janusintl.com/news-media/blog/the-naked-truth
4. Nokē-integrasjoner (Janus Europe): https://januseurope.com/noke-integrations/
5. Green Storage Norge, om oss: https://www.greenstorage.com/no/om-oss/
6. Salto KS Connect API, integrasjonstyper: https://developer.saltosystems.com/ks/connect-api/integration-types/
7. Storeganise, Salto KS-add-on: https://help.storeganise.com/article/408-salto-access-control
8. Salto KS, offline-tilgang: https://support.saltosystems.com/ks/general/offline-access/
9. digital-key-world, Salto KS voucher Lite/Pro: https://www.digital-key-world.com/en/SALTO-KS-Voucher-Abonnement-Lite-Pro-Configurator/SAL-KS-VOUCHKSx
10. digital-key-world, Salto Neo-sylinder: https://www.digital-key-world.com/en/SALTO-KS/Locking-cylinder/Neo-Locking-cylinder/
11. Sharefox, integrasjoner: https://sharefox.no/tjenesten/integrasjoner/ (11b: https://sharefox.no/blogg/las-til-minilager-velg-riktig-losning/)
12. Second Space, spørsmål og svar: https://secondspace.no/sporsmal-og-svar/
13. Stora Marketplace, sedisto: https://marketplace.stora.co/applications/sedisto (tidligere https://stora.co/integrations/sedisto)
14. Storeganise, sedisto-partner: https://storeganise.com/partner/sedisto
15. sedisto, digital adgangskontroll: https://www.sedisto.com/digital-access-control
16. SiteLink, PTI-integrasjon: https://www.sitelink.com/marketplace/gate-access/pti-security-systems
17. PTI, StorLogix Cloud Platform: https://www.ptistoragesecurity.com.au/storlogixcloudplatform/
18. Storeganise, integrasjoner: https://storeganise.com/integrations
19. Stora-blogg, «Top 9 Smart Self Storage Access Control Systems»: https://stora.co/blog/automated-smart-entry-options-for-self-storage
20. OpenTech, Storeganise-integrasjon: https://opentechalliance.com/blog/opentech-integrates-insomniac-cia-access-control-with-storeganise-self-storage/
21. SiteLink, INSOMNIAC CIA: https://www.sitelink.com/marketplace/gate-access/insomniac-cia-access-control
22. OpenTech, Stora-integrasjon: https://opentechalliance.com/blog/opentech-alliance-announces-integration-with-stora-self-storage-management-software/
23. BearBox, ubetjent minilager: https://bearbox.eu/en/solutions/automated-or-unmanned-self-storage
24. SiteLink, BearBox: https://www.sitelink.com/marketplace/gate-access/bearbox
25. Paxton, Stora-integrasjon: https://www.paxton-access.com/integrations/stora
26. igloohome for utviklere: https://www.igloohome.co/developers
27. RCO, aktive integrasjoner og API-er: https://rco.se/integrationer/aktiva-integrationer-och-apier
28. Unloc API: https://www.unloc.app/api
29. Salto KS Connect API: https://developer.saltosystems.com/ks/connect-api/
30. Seam, Salto KS PIN-koder (sekundær): https://docs.seam.co/latest/device-and-system-integration-guides/salto-ks-access-control-system/programming-code-based-salto-ks-credentials
31. Salto, nytt abonnement for KS: https://saltosystems.com/en/blog/announcing-new-salto-ks-subscription-model/
32. REFIRE, intervju med sedisto: https://www.refire-online.com/guest-columns/sedisto-we-dont-make-smart-locks-we-make-locks-smart/
33. Salto Systems Nordic og Unloc (bare overskriften er lest): https://www.mynewsdesk.com/no/saltosystems/pressreleases/salto-systems-nordic-og-unloc-inngaar-samarbeid-bygger-aapen-infrastruktur-for-digitale-noekler-3079426
34. Stora Marketplace: https://marketplace.stora.co/
35. SiteLink, Gates & Access: https://www.sitelink.com/marketplace/gate-access
36. Pelican-appen: https://pelican.se/en/pelican-app/
37. Flexistore: https://www.flexistore.no/en/
