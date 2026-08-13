---
name: readme-check
description: Manutenzione periodica del profilo GitHub matteobaccan/matteobaccan — cerca errori di battitura, rigenera la tabella "Some of my projects" con i repo da almeno 5 star, aggiorna la lista contributi open source e controlla che linguaggi e tecnologie siano allineati ai progetti. Usala quando l'utente chiede di ricontrollare, aggiornare o rinfrescare il README del profilo, la lista progetti, le star, i contributi o le tecnologie.
---

# Check periodico del README del profilo

Quattro attività indipendenti. Falle tutte salvo diversa indicazione, e riporta i risultati separatamente.

## Struttura del README

L'ordine delle sezioni è stato deciso apposta: **prima il codice, poi il materiale didattico, in fondo i widget**. Non rimetterlo in ordine alfabetico o cronologico.

```
intro + social
## My open source contributions   <- il segnale più forte, sta in alto
## Some of my projects
## Skills                         (### Languages used / explored / technologies)
## My books
## My courses
## My slides                      <- adiacente ai corsi: stessa natura
## My Talks
## My articles                    (### Daily.DEV, lista dentro <details>)
## Stats and badges               (### snake / trophy / hacktoberfest / userstats)
blocchi commentati
```

Vincoli da non rompere:

- I marker `<!-- daily.dev BOOKMARKS:START -->` / `:END` sono usati da `gautamkrishnar/blog-post-workflow` per riscrivere la lista articoli. Devono restare, con la lista **direttamente** tra i due. Il `<details>` va **fuori** dai marker, altrimenti la action lo cancella al primo giro.
- Dopo `<summary>` ci vuole una riga vuota, altrimenti la lista markdown non renderizza dentro il `<details>`.
- Il file contiene 13 blocchi commentati (widget disattivati). Se sposti sezioni, controlla che aperture e chiusure restino pari:
  `grep -o '<!--' README.md | wc -l` e `grep -o -- '-->' README.md | wc -l` devono coincidere.

Se riorganizzi ancora, verifica di non aver perso contenuto confrontando le righe ordinate:

```bash
cp README.md /tmp/before.md   # PRIMA di toccare
# ...modifiche...
diff <(grep -v '^\s*$' /tmp/before.md | sort) <(grep -v '^\s*$' README.md | sort)
```

Devono comparire **solo aggiunte** intenzionali, mai rimozioni.

## 1. Errori di battitura

Rileggi `README.md` per intero (è l'unico file con prosa; `.github/workflows/*.yml`, `.vscode/settings.json`, `.whitesource`, `renovate.json` contengono solo config).

Il testo è **misto italiano e inglese**: le sezioni descrittive sono in inglese, le descrizioni di corsi/slide in italiano. Valuta ogni frase nella sua lingua.

Cose che sono già emerse in passato, da ricontrollare:

- Elisioni italiane: `e esempi` → `ed esempi` (tutte le righe delle tabelle corsi usano "ed esempi", mantieni la coerenza)
- Accenti su nomi propri stranieri: `Josè` → `José`
- Nomi di prodotto: `CodeSpace` → `Codespaces`
- Spazi in coda a fine riga (`sed -i 's/[[:space:]]*$//' README.md`)

Refuso noto **non correggibile da qui**: il repo si chiama `ocalm-boilerplate`, dovrebbe essere `ocaml-boilerplate`. Serve un rename su GitHub, non una modifica al README. Ricordalo all'utente se non l'ha ancora fatto.

Regole:

- **Correggi** i refusi oggettivi (ortografia, accenti, elisioni, nomi di prodotto, grammatica palesemente rotta).
- **Segnala senza toccare** le scelte stilistiche dell'autore, anche quando sono discutibili. Esempio noto: riga "libro di riflessioni che ogni sviluppatore dovrebbe **porsi**" — in italiano "porsi" regge "domande", non "riflessioni", ma è prosa dell'autore.
- Se correggi una parola presente in `.vscode/settings.json` sotto `cSpell.words`, aggiorna anche quella voce.

Verifica che tutti i link ai repo esistano ancora:

```bash
grep -o 'https://github.com/matteobaccan/[A-Za-z0-9_-]*' README.md | sort -u | while read u; do
  r="${u##*/}"; echo "$r -> $(gh api "repos/matteobaccan/$r" -q '.full_name' 2>&1 | head -1)"
done
```

## 2. Tabella "Some of my projects"

Criterio: **tutti** i repo di `matteobaccan` con `stargazers_count >= 5`, **esclusi gli archiviati**, ordinati per star decrescenti.

```bash
gh api "users/matteobaccan/repos?per_page=100" --paginate \
  -q '.[] | select(.stargazers_count >= 5) | select(.archived | not) | "\(.stargazers_count)|\(.name)"' \
  | sort -t'|' -k1 -rn
```

Controlla anche cosa è stato escluso, così puoi dirlo all'utente:

```bash
gh api "users/matteobaccan/repos?per_page=100" --paginate \
  -q '.[] | select(.stargazers_count >= 5) | select(.archived) | "ARCHIVIATO: \(.stargazers_count) \(.name)"'
gh api "users/matteobaccan/repos?per_page=100" --paginate \
  -q '.[] | select(.stargazers_count == 4) | "VICINO ALLA SOGLIA: 4 \(.name)"'
```

### Formato riga

Cinque colonne. `$R` è il nome esatto del repo (case-sensitive: `owner`, `html2pop3`, `HarbourJwt`, `cheshire-cat-api-client-java` sono minuscoli/misti), `$LABEL` è l'etichetta visualizzata.

```
| [**$LABEL**](https://github.com/matteobaccan/$R) | [![GitHub stars](https://img.shields.io/github/stars/matteobaccan/$R?color=yellow&logo=github&style=flat)](https://github.com/matteobaccan/$R/stargazers) | [![GitHub issues](https://img.shields.io/github/issues/matteobaccan/$R?color=green&logo=github&style=flat)](https://github.com/matteobaccan/$R/issues) | [![GitHub PRs](https://img.shields.io/github/issues-pr/matteobaccan/$R?style=flat&logo=github)](https://github.com/matteobaccan/$R/pulls) | [![GitHub PRs](https://img.shields.io/github/issues-pr-closed/matteobaccan/$R?style=flat&color=critical&logo=github)](https://github.com/matteobaccan/$R/pulls?q=is%3Apr+is%3Aclosed) |
```

Intestazione:

```
| Project :octocat: | Stars :star: | Issues :bug: | Open PRs :bell: | Closed PRs :fire: |
|---|---|---|---|---|
```

### Etichette

**Riusa le etichette già presenti nel README** — sono scritte a mano e non derivabili dal nome del repo:

| Repo | Etichetta |
|---|---|
| `owner` | Owner |
| `html2pop3` | HTML2POP3 |
| `HarbourJwt` | HarbourJWT |
| `CorsoAIBook` | Guida Pratica all'uso delle AI |
| `CorsoAI` | Corso AI |
| `CorsoHTML` | Corso HTML |
| `cheshire-cat-api-client-java` | Cheshire Cat API Client Java |

Per un repo nuovo non in tabella, usa il nome così com'è (`MultiRipper`, `SockRedirector`, `PassiveCooker`) salvo che sia un acronimo che va maiuscolo.

### Dove NON vanno le star

Solo **Some of my projects** e **My open source contributions** hanno la colonna Stars.

`My books`, `My courses` e `My slides` restano a due colonne. La colonna era stata aggiunta e poi **rimossa su richiesta dell'utente**: non riproporla a ogni giro.

## 3. Contributi open source

La sezione `## My open source contributions` elenca i repo **di altri** con più di 1000 star in cui c'è almeno una PR mergiata.

```bash
# repo esterni con PR mergiate, con conteggio
gh api -X GET "search/issues" -f q="type:pr author:matteobaccan is:merged" -f per_page=100 --paginate \
  -q '.items[].repository_url' | sed 's|.*/repos/||' | grep -v "^matteobaccan/" | sort | uniq -c | sort -rn > /tmp/prs.txt

# filtro >1000 star
while read n repo; do
  s=$(gh api "repos/$repo" -q '.stargazers_count' 2>/dev/null)
  [ -n "$s" ] && echo "$s|$repo|$n"
done < /tmp/prs.txt | sort -t'|' -k1 -rn | awk -F'|' '$1>1000'
```

Controlla i redirect prima di scrivere (`gh api repos/X -q .full_name` deve restituire lo stesso nome), altrimenti finisci per listare un nome vecchio.

Formato riga, tre colonne — `$FULL` è `owner/repo`:

```
| [**$FULL**](https://github.com/$FULL) | [$N](https://github.com/$FULL/pulls?q=is%3Apr+author%3Amatteobaccan+is%3Amerged) | [![GitHub stars](https://img.shields.io/github/stars/$FULL?color=yellow&logo=github&style=flat)](https://github.com/$FULL/stargazers) |
```

**Ordinamento: per numero di PR decrescente**, a parità per star decrescenti. Non per star — è stato cambiato apposta.

Il conteggio PR è **statico**: va rigenerato a ogni giro, a differenza delle star che si aggiornano da sole. Segnala all'utente i repo che sono appena passati sopra 1000 star (nuovi ingressi) e quelli scesi sotto.

## 4. Linguaggi e tecnologie

Confronta le sezioni `Languages I have used`, `Languages I have explored` e `Some of the technologies I have worked with` con quello che i repo dicono davvero:

```bash
gh api "users/matteobaccan/repos?per_page=100" --paginate -q '.[] | select(.fork|not) | .language // empty' | sort | uniq -c | sort -rn
gh api "users/matteobaccan/repos?per_page=100" --paginate -q '.[] | select(.fork|not) | .topics[]?' | sort | uniq -c | sort -rn | head -40
```

Distinzione decisa dall'utente e da rispettare:

- **Languages I have used** → linguaggi con progetti veri dietro.
- **Languages I have explored** → linguaggi presenti solo nella serie `*-boilerplate` (repo template hello-world, quasi tutti a 0 star). Non promuoverli nella prima sezione solo perché esiste il repo.

Un linguaggio nuovo che compare **solo** in un nuovo `*-boilerplate` va nella seconda sezione.

### Loghi shields.io

Prima di aggiungere un badge, verifica che lo slug del logo esista davvero, altrimenti esce un badge senza icona:

```bash
for l in slug1 slug2; do
  svg=$(curl -s "https://img.shields.io/badge/-test-333333?style=flat&logo=$l")
  echo "$svg" | grep -q "<image" && echo "OK $l" || echo "NO $l"
done
```

Slug verificati come **inesistenti** (badge volutamente senza `logo=`): `cobol`, `mercury`, `brainfuck`, `jakt`, `picocli`, `web3j`, `css3`, `scss`, `w3c`, `gnucobol`.
Attenzione: il logo CSS è `css`, **non** `css3` (simple-icons l'ha rinominato). Per Web3j si usa `logo=ethereum`.

## Insidie verificate sul campo

- **Line ending**: `core.autocrlf=true` e il blob in repo è LF. Se riscrivi il file con Python usa `io.open(p,'w',encoding='utf-8',newline='\n')`, altrimenti sporchi il diff.
- **Quoting shell**: generando le righe con bash, un apostrofo dentro l'etichetta (`all'uso`) produce l'artefatto `all'\''uso` nel markdown. Dopo la generazione controlla sempre: `grep -n "\\\\''" README.md` deve dare zero risultati.
- **Python non vede `/tmp`**: `/tmp` di Git Bash non è raggiungibile dal Python di Windows. Usa la directory scratchpad della sessione per i file intermedi.

## Verifica finale

Prima di riportare all'utente:

```bash
# tutti i badge star rispondono 200
grep -o 'https://img.shields.io/github/stars/[^)]*' README.md | while read u; do
  printf "%s %s\n" "$(curl -s -o /dev/null -w '%{http_code}' "$u")" "${u##*stars/}"
done

# tutte le righe tabella hanno lo stesso numero di colonne (atteso: 2 e 5)
awk -F'|' '/^\|/{print NF-2}' README.md | sort -u

# nessun artefatto di quoting
grep -n "\\\\''" README.md

git diff --stat
```

## Al termine

Riporta in due blocchi: (a) refusi corretti in forma di tabella prima/dopo, più quelli segnalati e non toccati; (b) composizione della tabella progetti, con menzione esplicita di cosa è stato aggiunto, rimosso, escluso perché archiviato, e cosa sta a 4 star (candidato al prossimo giro).

**Non committare senza chiedere.** Lascia le modifiche nel working tree e proponi il commit.
