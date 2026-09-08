# Watchtower runtime template

Mal for ett privat konfigurasjons- og state-repo til en selvstendig
[Watchtower-fork](https://github.com/marmarmar-code/watchtower).
Malen inneholder ingen programkode eller secrets.

## Start her

Følg **[den samlede installasjonsveiledningen](https://github.com/marmarmar-code/watchtower/blob/main/INSTALL.md)**.
Den beskriver repoer, oppsettsveiviser, nøkler, kanal, baseline og automatisk kjøring.
Bruk veiledningen i den versjonen av kodeforken du faktisk tar i bruk.

1. Velg **Use this template** og opprett ditt eget **Private** repo.
2. Fork Watchtower-koden. `link-github` i steg 4 setter runtime-koblingen.
3. Klon begge repoene til separate kataloger. Fra kodekatalogen kjører du:

```bash
python -m watchtower setup --runtime ../watchtower-runtime
```

Veiviseren i Watchtower 0.6 lager et oppsett fra `general`, `finance`, `health`,
`digital`, `property` eller `retail`.
Den kan bevare den deaktiverte malen som `config/watchtower.before-setup.toml` før
ny konfigurasjon skrives. Den avviser aktive oppsett og runtimes som allerede har state.

4. Etter at konfigurasjonen er lagret i det private repoet, kobler du repoene med
   `link-github` fra kodekatalogen. Installer GitHub CLI og logg inn først:

```bash
python -m watchtower link-github --code-repo DIN_EIER/DIN_KODEFORK --runtime-repo DIN_EIER/DITT_RUNTIME_REPO
python -m watchtower link-github --code-repo DIN_EIER/DIN_KODEFORK --runtime-repo DIN_EIER/DITT_RUNTIME_REPO --apply
```

Kontrollen er uten endringer til `--apply` brukes. Kommandoen lager egen nøkkel og
setter runtime-variabler. Eksisterende koblinger overskrives ikke. Følg deretter
installasjonsveiledningen for kanaltilgang og kontroll av installasjonen.

Du kan også redigere `config/watchtower.toml` manuelt. Alle eksempelkilder er
fortsatt deaktivert. Erstatt aktuelle `REPLACE_ME`-verdier og aktiver bare de
kildene du vil følge. En aktiv kilde med plassholdere blir avvist.

## Innhold og ansvar

| Sted | Innhold |
| --- | --- |
| `config/watchtower.toml` | Privat konfigurasjon, virksomhetsliste og filtre |
| `state/` | Baseline, deduplisering, status, leveringskø og privat varselhistorikk |
| Actions Secrets i kodeforken | Deploy-nøkkel, webhook og eventuelle API-nøkler |

Malen leses bare ved opprettelse. Det skjer ingen automatisk oppdatering av denne
runtimen fra malen eller upstream. Behold én runtime per installasjon.

Avtal en driftsansvarlig, en redaksjonell ansvarlig og en stedfortreder lokalt.
Det følger ingen sentral drift, SLA eller plikt for upstream til å feilsøke oppsettet.
Avklar bruksrett med rettighetshaver mens Watchtower mangler formell programvarelisens.

## Videre veiledning

- [Kilder og filtrering](https://github.com/marmarmar-code/watchtower/blob/main/README.md)
- [Gjenbruk av virksomhetslister](https://github.com/marmarmar-code/watchtower/blob/main/ENTITIES.md)
- [Drift, dekningsstatus og varselhistorikk](https://github.com/marmarmar-code/watchtower/blob/main/OPERATIONS.md)
- [Oppgraderinger og kompatibilitet](https://github.com/marmarmar-code/watchtower/blob/main/UPGRADING.md)

Konfigurasjonsformatet er 1. Eldre oppsett uten eksplisitt versjonsfelt kan fortsatt
leses. Nye `entity_refs` krever Watchtower 0.5; legg dem ikke til før kodeforken er oppgradert.

0.5-malen inneholder også et deaktivert eksempel for `finanstilsynet_registry`.
Fullfør en eventuell ventende leveringskø før retur til en eldre kodeversjon.

## Utvid oppsettet med hendelser og tall

Watchtower 0.6 har ti ferdige oppskrifter for blant annet styringsrente, valuta,
SSB-tall, alvorlige farevarsler, Riksrevisjonens rapporter og Nkom. Fra kodekatalogen:

```bash
python -m watchtower list-recipes
python -m watchtower add-source --runtime ../watchtower-runtime --recipe nb_policy_rate
python -m watchtower add-source --runtime ../watchtower-runtime --recipe nb_policy_rate --apply
python -m watchtower preview --config ../watchtower-runtime/config/watchtower.toml --state-dir ../watchtower-runtime/state --source nb_policy_rate
```

Første `add-source` viser forslaget; `--apply` legger det til lokalt. Commit og push
fra din private runtime etter kontroll. Eksisterende konfigurasjon og state bevares.
`preview` sender ingenting og skriver ikke state. Første ordinære henting av en
ny kilde etablerer stille baseline.

Malen har deaktiverte eksempler for egne JSON-utvalg, nettsidetekst og SSB-tall.
Disse krever kodeversjon 0.6. CSV og dokumentlister er også tilgjengelig via
oppskrifter og egen konfigurasjon. Se
[ferdige kildeoppsett](https://github.com/marmarmar-code/watchtower/blob/main/RECIPES.md) og
[endringsregler](https://github.com/marmarmar-code/watchtower/blob/main/EVENT_MONITORING.md).
