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

Veiviseren i Watchtower 0.5 lager et oppsett fra `general`, `finance` eller `health`.
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
