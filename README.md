# Watchtower runtime template

Privat konfigurasjons- og state-repository for [Watchtower](https://github.com/marmarmar-code/watchtower).

Dette repositoryet inneholder ingen programkode. Det skal heller aldri inneholde webhook-adresser, API-nøkler, tilgangstokener eller private nøkler.

## Ansvarsdeling

En installasjon består av to repositoryer:

```text
<organisasjon>/watchtower           offentlig fork med programkode og workflow
<organisasjon>/watchtower-runtime   privat konfigurasjon og state
```

Hver installasjon eier og drifter sin egen fork, runtime, konfigurasjon og secrets. Det følger ingen sentral driftsgaranti eller plikt til å utvikle særtilpasninger. Generelle forbedringer kan foreslås som pull requests til upstream-repositoryet.

## Kom raskt i gang

1. Opprett et privat repository fra malen med navnet `watchtower-runtime`.
2. Fork Watchtower til samme konto eller organisasjon, og aktiver Actions.
3. Følg stegene under for deploy key, varsling og kilder.

Malen er laget for redaksjoner som vil følge offentlige kilder uten å legge hemmeligheter eller privat state i den offentlige koden. Du trenger bare å fylle inn kildene du faktisk vil følge.

## 1. Opprett privat runtime

Velg **Use this template** og opprett:

```text
<organisasjon>/watchtower-runtime
```

Sett repositoryet til **Private**. Ikke bruk vanlig fork av denne malen; en template gir en ren historikk.

Standardnavnet `watchtower-runtime` gjør at Watchtower finner runtime automatisk når begge repositoryene har samme eier.

Brukes et annet navn eller en annen eier, opprett denne Actions-variabelen i Watchtower-forken:

```text
WATCHTOWER_RUNTIME_REPOSITORY=<eier>/<runtime-repository>
```

En annen runtime-branch kan eventuelt angis med:

```text
WATCHTOWER_RUNTIME_REF=<branch>
```

Det er ikke nødvendig å redigere workflow-filen.

## 2. Fork Watchtower

Fork:

```text
marmarmar-code/watchtower
```

til kontoen eller organisasjonen som skal kjøre installasjonen.

Åpne deretter fanen **Actions** i forken og aktiver workflows dersom GitHub ber om det. Planlagte workflows i nye forker kan være deaktivert til dette er gjort.

## 3. Opprett avgrenset deploy key

Lag et eget SSH-nøkkelpar på en maskin du kontrollerer:

```bash
ssh-keygen -t ed25519 \
  -C "watchtower-runtime" \
  -f watchtower-runtime-key \
  -N ""
```

Dette lager:

```text
watchtower-runtime-key.pub   offentlig nøkkel
watchtower-runtime-key       privat nøkkel
```

I det private runtime-repositoryet:

1. Gå til **Settings → Deploy keys → Add deploy key**.
2. Lim inn innholdet fra `watchtower-runtime-key.pub`.
3. Aktiver **Allow write access**. Watchtower må kunne oppdatere `state/`.

I den offentlige Watchtower-forken:

1. Gå til **Settings → Secrets and variables → Actions → Secrets**.
2. Opprett secret `RUNTIME_DEPLOY_KEY`.
3. Lim inn hele innholdet fra den private filen `watchtower-runtime-key`.
4. Slett nøkkelfilene lokalt når oppsettet er verifisert, eller oppbevar dem sikkert dersom de skal kunne roteres senere.

Ikke gjenbruk denne deploy key-en til andre repositoryer.

## 4. Konfigurer varsling

Microsoft Teams er standard i denne malen:

```toml
[notifications]
provider = "teams"
```

Opprett en Teams Workflow med webhook-trigger for ønsket kanal. Legg webhook-adressen i Watchtower-forken som GitHub Actions-secret:

```text
TEAMS_WEBHOOK_URL
```

For Slack, endre runtime-konfigurasjonen til:

```toml
[notifications]
provider = "slack"
```

og opprett:

```text
SLACK_WEBHOOK_URL
```

Webhook-adressen skal aldri limes inn i `config/watchtower.toml`.

## 5. Konfigurer kilder

Rediger:

```text
config/watchtower.toml
```

Alle eksempelkilder er deaktivert. For hver kilde som skal brukes:

1. Erstatt alle relevante `REPLACE_ME`-verdier.
2. Kontroller kilde-spesifikke innstillinger.
3. Sett `enabled = true`.

Minst én kilde må være aktiv før `dry-run` eller ordinær `run`. Watchtower avviser en overvåkingskjøring uten aktive kilder, slik at et uferdig oppsett ikke ser vellykket ut.

En aktiv kilde med gjenværende `REPLACE_ME` blir avvist av valideringen. En aktiv kilde må også ha positive filterregler, eller eksplisitt:

```toml
[source.filter]
match_all = true
```

`match_all` bør bare brukes når adapteren allerede er begrenset av en konkret liste, slik som BRREGs `companies`.

### Kildetyper

```text
regjeringen
stortinget
konkurransetilsynet
euronext
doffin
hoyesterett
brreg
rss
ssb
```

Doffin krever Actions-secret:

```text
DOFFIN_API_KEY
```

BRREG krever ingen API-nøkkel. Legg organisasjonsnumrene i `companies` og velg hendelser gjennom `events`.

RSS-profiler gjør flere offisielle feeder tilgjengelige uten at redaksjonen må finne og vedlikeholde URL-ene selv. Watchtower leveres med profiler for Politiloggen, Finanstilsynet, Mattilsynet og Norges Banks pressemeldinger. Kommandoen `python -m watchtower list-rss-profiles` viser profilnavnene som kan brukes i konfigurasjonen.

SSB krever ingen API-nøkkel. Legg femsifrede tabellnumre i `tables`. Watchtower henter bare den lille tabellbeskrivelsen og varsler om nye perioder eller strukturendringer; den laster ikke ned selve statistikkdataene.

## 6. Verifiser oppsettet

Kjør workflowen manuelt fra **Actions → Watchtower → Run workflow**.

### A. `test-notification`

Denne sender et representativt testvarsel med tittel, endringsdetalj, metadata og lenkeknapp gjennom provideren som er valgt i runtime.

Testen er først godkjent når:

- meldingen faktisk vises i valgt kanal;
- lenken kan åpnes;
- ved Teams viser den tilhørende Workflow-kjøringen `Succeeded`.

En grønn GitHub-jobb alene er ikke tilstrekkelig dersom meldingen ikke vises.

### B. `dry-run`

Denne validerer runtime og henter aktive kilder uten å sende ordinære varsler eller skrive ny state.

Godkjenn bare kjøringen dersom alle aktive kilder fullfører uten feil.

### C. `run`

Første ordinære kjøring etablerer en stille baseline. Det skal normalt ikke komme kildevarsler. Workflowen skal committe JSON-state til `state/` i det private repositoryet.

Kontroller:

- at workflowen er grønn;
- at `state/` inneholder filer for de aktive kildene;
- at committen er skrevet av `watchtower[bot]`;
- at neste uendrede kjøring ikke sender varsler.

## 7. Normal drift

Workflowen våkner hvert femte minutt. Den sjekker likevel hver kilde bare når kildeintervallet er utløpt (standard er 60 minutter). Dette gir én felles tidsplan, mens hyppigere eller sjeldnere kilder kan velge sitt eget `interval_minutes` (minimum 5).

State må forbli privat og versjonert. Ikke slett state for en aktiv kilde uten å forstå at neste ordinære kjøring da oppretter en ny stille baseline.

Hver kjøring viser en samlet, anonymisert kildestatus i oppsummeringen på GitHub. Den røper ikke private kilde-ID-er eller filterverdier. Detaljene for den enkelte kilde forblir i den private runtime-en.

## Sikkerhetsregler

- Produksjonsruntime skal være privat.
- Secrets skal bare ligge i GitHub Actions Secrets.
- Ikke commit `.env`, private nøkler, sertifikater eller webhook-adresser.
- Ikke kopier state eller konfigurasjon fra en annen installasjon.
- Kontroller repositoryets synlighet før reelle overvåkingsverdier legges inn.
- Roter straks en credential som ved en feil er publisert; sletting av filen alene er ikke tilstrekkelig.

## Oppdatering av fork

Hver fork velger selv når den synkroniseres med upstream. Hold egne kodeendringer små og isolerte for å redusere konflikter.

Produksjonsworkflowen sjekker med vilje ut kode fra `main`, slik at en utestet branch ikke får tilgang til produksjonssecrets og privat runtime. Oppgrader derfor slik:

1. Synkroniser upstream-endringen i en branch.
2. La branch-CI fullføre og gjennomgå diffen.
3. Merge først når CI er grønn.
4. Kjør `dry-run` manuelt fra `main` umiddelbart etter merge.
5. Kjør deretter `run` og kontroller state og varsling.

Ved en større oppgradering kan den planlagte workflowen deaktiveres midlertidig mens kontrollene gjennomføres.

## Anbefalt beskyttelse

Beskytt `main` i Watchtower-forken med:

- pull request før merge;
- grønn CI som krav;
- blokkering av force push;
- blokkering av sletting av branch.

Dette hindrer at en utestet kodeendring får tilgang til produksjonssecrets og privat runtime ved neste planlagte kjøring.

## Lisensstatus

Det er foreløpig ikke lagt inn en programvarelisens for Watchtower. Den offentlige koden og denne malen gir derfor ikke i seg selv generell tillatelse til bruk, endring eller videre distribusjon.

En ny installasjon må ha uttrykkelig tillatelse fra rettighetshaveren fram til rettighetshaver og lisens er formelt avklart.
