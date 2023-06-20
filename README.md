# flex-distroless-bump

Dette repoet inneholder script og innstruksjoner distroless-versjon i Dockerilfilene til Java- og NodeJs-applikasjoner
tilhørende Team Flex i én og samme operasjon.

Prosessen med å bumpe all Java apper i `~/Development` foregår på følgende måte:

Start med å gjøre scriptet `flex-apps` kjørbart med `chmod u+x flex-apps`.

Finn deretter _latest_ distroless-versjon
for [java17-debian11](https://console.cloud.google.com/gcr/images/distroless/global/java17-debian11)
eller [nodejs18-debian11](https://console.cloud.google.com/gcr/images/distroless/global/nodejs18-debian11)
-applikasjonene som skal oppdateres og lagre den i en miljøvariabel:

```sh
DISTROLESS_SHA=672df6324b5e36527b201135c37c3ed14579b2eb9485a4f4e9ab526d466f671c
```

Bruk `flex-apps` til å finne alle java- eller node-applikasjoner i en mappe og lagre de i en miljøvariabel:

```sh
# Gå til mappen som inneholder alle appene. For eksempel ~/Development:
cd ~/Development

# Finn alle java-applikasjoner og lagre de i en miljøvariabel:
FLEX_APPS=$(flex-distroless-bump/flex-apps java)
```

Bruk deretter koden under til å iterere over alle appene og utføre en kommando i hver av appene:

```sh
# Pull master og lag en ny branch i hver av appene. Har du pågående arbeid vil du kanskje stashe endringer først:
while IFS= read -r dir; do (cd $dir; git co master; git pull; git switch -c dev-bump-distroless) done <<< "$FLEX_APPS"

# Bump distroless-versjon i hver av appene og verifiser med 'git diff':
while IFS= read -r dir; do (cd $dir; sed -i "s/\(^FROM.*:\).*$/\1$DISTROLESS_SHA/g" Dockerfile; git diff) done <<< "$FLEX_APPS"

# Commit og push endringene i hver av appene:
while IFS= read -r dir; do (cd $dir; git commit -am "Bump-distroless"; git push) done <<< "$FLEX_APPS"

# Opprett pull request i hver av appene (krever at GitHub CLI - 'gh' installert):
while IFS= read -r dir; do (cd $dir; gh pr create --title "Bump distroless" --body "Bump distroless SHA to $DISTROLESS_SHA") done <<< "$FLEX_APPS"

# Merge PR når testing er unnagjort (krever at GitHub CLI - 'gh' installert):
while IFS= read -r dir; do (cd $dir; gh pr merge -s) done <<< "$FLEX_APPS"

## Sjekk ut master og slett branchen i hver av appene:
while IFS= read -r dir; do (cd $dir; git co master; git branch -D dev-bump-distroless) done <<< "$FLEX_APPS"
```

While-funksjonaliteten til Kommandoen over kan flyttes inn i et script, men det er ikke trivielt å støtte alle
kommandoer som må kjøres som input til det scriptet.


