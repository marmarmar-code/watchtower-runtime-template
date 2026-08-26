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
```

Doffin krever Actions-secret:

```text
DOFFIN_API_KEY
```

BRREG krever ingen API-nøkkel. Legg organisasjonsnumrene i `companies` og velg hendelser gjennom `events`.

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

Standardworkflowen kjører hver time på minutt `:23`. Tidsplanen kan endres i den enkelte fork.

State må forbli privat og versjonert. Ikke slett state for en aktiv kilde uten å forstå at neste ordinære kjøring da oppretter en ny stille baseline.

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
