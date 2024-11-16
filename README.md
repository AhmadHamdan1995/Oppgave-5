# Oppgavespesfikke leveranser

\# | Oppgave | Leveranse 1 | Leveranse 2 | Leveranse 3
--- | --- | --- | --- | ---
1 | AWS Lambda og GitHub Actions | Lambda Endponit https://ga0wooh0i9.execute-api.eu-west-1.amazonaws.com/Prod/hello | GitHub Actions: https://github.com/AhmadHamdan1995/-sam-lambda/actions/runs/11844422654/| -
2 | Infrastruktur med Terraform of SQS | SQS Queue URL https://sqs.eu-west-1.amazonaws.com/244530008913/image-processing-queue | GitHub Actions workflow (master) https://github.com/AhmadHamdan1995/SQS/actions/runs/11861720236/ | GitHub Actions workflow (non-master) https://github.com/AhmadHamdan1995/SQS/actions/runs/11861827040/
3 | Javaklient og Docker | - | - | - 
4 | Metrics og overvåkning | SQS Queue URL https://sqs.eu-west-1.amazonaws.com/244530008913/image-processing-queue | GitHub Actions workflow (master) https://github.com/AhmadHamdan1995/SQS/actions/runs/11865699575 | GitHub Actions workflow (non-master) https://github.com/AhmadHamdan1995/SQS/actions/runs/11865722046
5 | Drøft; Serverless, Function as a service vs Container-teknologi |  Diskusjon og evaluering


## Oppgave 5: Serverløs, Function-as-a-Service vs. Containerteknologi

### 1. Automatisering og Kontinuerlig Levering (CI/CD)

#### Mikrotjenester

- Med mikrotjenester er hver tjeneste vanligvis en containerisert applikasjon, og trenger sin egen CI/CD-pipeline. Dette betyr at vi har mer komplekse pipelines å administrere, spesielt hvis vi bruker noe som GitHub Actions integrert med AWS-tjenester.
- Distribusjoner kan ta lengre tid siden containere er tyngre enn individuelle serverløse funksjoner. På den annen side gir mikrotjenester oss mer kontroll over distribusjonsprosessen.
- Terraform er flott her for infrastruktur som kode, som gjør det mulig for oss å sette opp alle våre mikrotjenestemiljøer konsekvent.

#### Serverløs (FaaS)

- Serverløse funksjoner er mer granulære, så i stedet for å distribuere en hel tjeneste, distribuerer vi flere mindre funksjoner. Dette kan øke distribusjonshastigheten siden Lambda-funksjoner distribueres raskere enn containere.
- Dette betyr imidlertid også at vi må administrere flere distribusjonspipelines, som kan bli kaotisk uten riktig automatisering. Vi må ha automatiseringsskript og arbeidsflyter (som å bruke GitHub Actions) som kan distribuere funksjoner individuelt, noe som kan øke kompleksiteten i versjonskontroll.

**Konklusjon**: Mikrotjenester er bedre for kontrollerte distribusjoner, mens serverløs er raskere, men krever nøye automatisering for å unngå pipeline-spredning.

### 2. Observabilitet

#### Mikrotjenester

- Overvåking av mikrotjenester innebærer vanligvis å spore containere, tjenester og deres interaksjoner. Verktøy som AWS CloudWatch, sammen med tilpassede metrikk, brukes til å overvåke helse og ytelse.
- Logging er sentralisert (ved hjelp av verktøy som CloudWatch Logs), noe som gjør det lettere å spore problemer siden mikrotjenester er langvarige, så logger er mer konsistente.
- Feilsøking er enklere med mikrotjenester siden vi har mer kontroll over miljøet vårt og kan få tilgang til applikasjonslogger og metrikk direkte fra kjørende containere.

#### Serverløs (FaaS)

- Serverløse funksjoner er i sin natur statsløse og kortvarige, noe som gjør observabilitet vanskeligere. Logger er spredt over mange små funksjoner, noe som betyr at vi må bruke verktøy som AWS X-Ray eller CloudWatch Logs Insights for å sette sammen spor fra flere kilder.
- Utfordringen ligger i å spore ulike steder og logger, spesielt hvis funksjoner blir kalt asynkront (som gjennom SQS). Feilsøking kan bli vanskelig når vi har en kjede av Lambda-funksjoner som kaller hverandre.
- Men med riktig observabilitetsoppsett (som å bruke CloudWatch-alarm), kan vi sette opp automatiserte svar på problemer, noe som hjelper med kontinuerlige tilbakemeldingssløyfer.

**Konklusjon**: Mikrotjenester er lettere å overvåke og feilsøke, mens serverløs trenger avanserte verktøy for distribuert sporing og overvåking på grunn av sin hendelsesdrevne natur.

### 3. Skalerbarhet og Kostnadskontroll

#### Mikrotjenester

- Mikrotjenester skalerer basert på ressursene vi tildeler containere (som CPU og minne), noe som kan være mer forutsigbart, men også dyrere hvis det ikke er optimalisert. Auto-skalering kan brukes, men må konfigureres riktig.
- Kostnader kan raskt øke på grunn av alltid-på-infrastruktur, spesielt hvis vi ikke utnytter ressursene fullt ut.

#### Serverløs (FaaS)

- Serverløse funksjoner som Lambda skalerer automatisk basert på innkommende hendelser, noe som er ideelt for uforutsigbare arbeidsbelastninger. Vi betaler kun for det vi bruker (per kjøring), noe som generelt er mer kostnadseffektivt for bursty eller lav gjennomstrømningsapplikasjoner.
- Imidlertid, hvis applikasjonen må håndtere konstant tung trafikk, kan serverløs bli dyrere på grunn av pay-per-request-modellen.
- SQS kan bidra til å avkoble og kontrollere flyten av hendelser, redusere kostnader ved å buffer og håndtere topper effektivt.

**Konklusjon**: Serverløs er kostnadseffektiv og skalerer godt for bursty arbeidsbelastninger, mens mikrotjenester er bedre for forutsigbare, høyvolumsapplikasjoner med stabile skaleringsbehov.

### 4. Eierskap og Ansvar

#### Mikrotjenester

- Mikrotjenester legger mer ansvar på DevOps-teamet for å vedlikeholde infrastruktur, håndtere distribusjoner, skalering og patching. Det er en tydelig grense mellom forskjellige tjenester, noe som gjør det lettere å tildele eierskap.
- Teamene må administrere sine egne pipelines, overvåkingsoppsett og infrastrukturkonfigurasjoner (ved hjelp av IaC-verktøy som Terraform).

#### Serverløs (FaaS)

- Med serverløs skifter mye av infrastrukturanvaret til AWS (f.eks. ingen serveradministrasjon, innebygd auto-skalering). Dette reduserer den operative byrden på DevOps-teamet.
- Imidlertid må teamet fortsatt administrere funksjonsspesifikke konfigurasjoner, distribusjonsautomatisering og sørge for at kostnadsoptimaliseringsstrategier (som å unngå cold start-problemer) er på plass.
- Serverløs fremmer også en delt ansvarlighetsmodell, der utviklere kan ta mer eierskap over distribusjon og administrasjon av funksjoner siden de er enklere å bygge og distribuere.

**Konklusjon**: Mikrotjenester krever mer praktisk administrasjon fra DevOps-teamet, mens serverløs reduserer infrastrukturadministrasjon, men krever nøye planlegging rundt kostnad og funksjonseierskap.
