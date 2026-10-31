# Guida pratica Git e GitHub (Windows PowerShell)

Questa guida spiega come pubblicare le modifiche, tornare alla versione
online e consultare le versioni precedenti. Eseguire i comandi nella cartella
del progetto, da **PowerShell**. I blocchi sono sequenze di comandi: controllare
l'esito di ciascuno prima di passare al successivo. In Windows PowerShell 5.1
non usare `&&` come separatore dei comandi.

> **Stato iniziale di questa copia:** al momento della stesura, la cartella
> non contiene `.git` ne un `.gitignore`. Quindi `git status`, `git push` e
> `git reset` funzioneranno solo dopo aver inizializzato un repository qui
> oppure dopo aver clonato un repository esistente. Non usare `git init` in
> una copia che contiene gia `.git`.

## 1. Preparazione

Installare [Git per Windows](https://git-scm.com/download/win) e,
per i comandi `gh`, [GitHub CLI](https://cli.github.com/). Creare un
[account GitHub](https://github.com/signup) se necessario. Aprire una nuova
finestra PowerShell e verificare:

```powershell
git --version
gh --version
```

Configurare l'identita dei commit (sostituire i valori con i propri):

```powershell
git config --global user.name "Nome Cognome"
git config --global user.email "email-associata-a-github@example.com"
git config --global --get user.name
git config --global --get user.email
```

L'email e il nome identificano l'autore dei commit; **non** effettuano
l'accesso a GitHub. Per l'autenticazione via browser:

```powershell
gh auth login
gh auth status
```

Nel menu di `gh auth login` scegliere GitHub.com e il protocollo HTTPS;
seguire la procedura proposta nel browser. Una password GitHub non va
inserita come password per `git push` su HTTPS. Per i repository privati
serve l'autorizzazione dell'account che esegue il push.

Portarsi nella cartella del progetto (adattare il percorso sul proprio PC):

```powershell
Set-Location "C:\Users\elio.errico\Desktop\Python\2026_09_05_Refrigerant_circuit_energy_balance_docker"
```

## 2. Proteggere i file locali prima di pubblicare

Prima di `git add .`, creare **nella radice del progetto** un `.gitignore`
almeno con queste righe, e adattarlo agli altri file locali sensibili:

```gitignore
venv/
.venv/
env/
ENV/
.env
.env.*
!.env.example
.secrets/
backups/
__pycache__/
*.py[cod]
.pytest_cache/
```

`.gitignore` non esiste ancora in questa copia: non presumere che i venv
o `.env` siano gia protetti. `!.env.example` consente di pubblicare un
esempio **solo se privo di segreti**. Non aggiungere credenziali, backup,
database, dati personali o artefatti generati. In questo progetto verificare
**prima di pubblicare** anche la licenza e i diritti di redistribuzione di
`REFPROP/`, `REFPROP.zip`, `SierraStudio_dll_1_1_3_0/` e relativo ZIP.
Se non sono redistribuibili, escluderli e documentare come installarli
separatamente: senza questi asset il clone non sara operativo da solo.

Controllare cosa Git considera tracciato, ignorato o pronto per il commit:

```powershell
git status --short
git status --short --ignored
git ls-files .env venv .venv env ENV .secrets backups
git diff --cached --stat
```

`git ls-files` deve essere vuoto per i file locali sensibili. Se un file
e **gia tracciato**, aggiungerlo a `.gitignore` non lo protegge: occorre
valutare la rimozione dall'indice con `git rm --cached` (o `-r --cached`
per una directory) e cambiare le eventuali credenziali gia pubblicate.
Il file locale resta sul disco dopo `--cached`, ma i commit passati possono
ancora contenerlo. Non eseguire `git add .` finche la selezione non e chiara.

## 3. Creare o collegare un repository

### Progetto locale nuovo, senza `.git`

Preparare `.gitignore` come sopra, poi:

```powershell
git init -b main
git add .
git status
git diff --cached --stat
git commit -m "Prima versione del progetto"
```

Esaminare l'elenco dei file in `git status` **prima** del commit. Per una
prima pubblicazione con GitHub CLI scegliere **una sola** visibilita.
Sostituire `NOME_REPOSITORY` con un nome libero nel proprio account:

```powershell
gh repo create NOME_REPOSITORY --private --source . --remote origin --push
```

Oppure, per un repository visibile a tutti:

```powershell
gh repo create NOME_REPOSITORY --public --source . --remote origin --push
```

Per questo progetto non scegliere `--public` senza aver verificato asset
licenziati, credenziali e dati. Prima di creare il remoto controllare
`git remote -v`: se `origin` esiste gia, non ricrearlo.

### Repository GitHub gia esistente

Per lavorare da una **nuova cartella**, usare l'URL HTTPS mostrato su GitHub:

```powershell
git clone https://github.com/UTENTE/NOME_REPOSITORY.git
Set-Location .\NOME_REPOSITORY
git status
```

Il clone contiene i file versionati, **non** `.env`, `venv`, backup o altri
file ignorati. Ricreare gli ambienti e la configurazione locale seguendo
il [README del progetto](../README.md), senza sovrascrivere la copia
originale. Se la cartella di lavoro esiste gia ma non ha `.git`, non clonare
sopra di essa e non inizializzarla per poi forzare il push: tenere le due
copie separate e confrontarle prima di trasferire modifiche.

## A. Ho modificato il PC: commit e caricamento online

Da una cartella Git gia collegata a GitHub, controllare il branch e i file:

```powershell
git status
git branch --show-current
git remote -v
git diff --stat
```

Selezionare i file desiderati con `git add percorso\file` (ripetibile).
Usare `git add .` solo dopo aver verificato `.gitignore` e tutti i file
nuovi. Poi:

```powershell
git add percorso\file
git diff --cached --stat
git diff --cached
git commit -m "Descrizione chiara della modifica"
git pull --rebase origin main
git push origin main
git status
```

Sostituire `main` con il branch effettivo mostrato da
`git branch --show-current`. `git pull --rebase` puo fermarsi in caso di
conflitto: risolvere i file, eseguire `git add` sui file risolti e poi
`git rebase --continue`; per annullare il rebase, `git rebase --abort`.
Non forzare il push per risolvere un rifiuto. Se `git status` dice che non
c'e nulla da salvare, non occorre un commit. Per vedere i commit locali
ancora assenti online: `git log --oneline origin/main..HEAD`.

Se `git add` segnala che LF verra convertito in CRLF, e un avviso sulle
terminazioni di riga Windows, non un errore. Controllare il diff e proseguire
se il contenuto e quello voluto; non cambiare la configurazione globale di
Git solo per nascondere l'avviso.

## B. Scartare le modifiche locali e tornare alla versione online

**Attenzione:** questa operazione elimina modifiche a file tracciati,
file nuovi non ignorati e commit locali non pubblicati dal branch corrente.
Non altera i file gia pubblicati su GitHub. Conserva `venv` e `.env`
**solo se non sono tracciati e sono ignorati o esclusi dai comandi `clean`**.
Non cancella i volumi Docker: per dati di PostgreSQL e backup consultare
separatamente il manuale della piattaforma.

Verificare **prima** il branch (`main` nell'esempio), l'URL di `origin`,
i file tracciati sensibili e i commit che verrebbero abbandonati:

```powershell
git status
git branch --show-current
git remote -v
git ls-files .env venv .venv env ENV .secrets backups
git fetch origin
git ls-tree -r --name-only origin/main -- .env venv .venv env ENV .secrets backups
git log --oneline origin/main..HEAD
git diff --stat origin/main
git clean -nd -e venv/ -e .venv/ -e env/ -e ENV/ -e .env -e .env.* -e .secrets/ -e backups/
```

`git clean -nd` e **solo una simulazione**: leggere ogni riga `Would remove`.
Se si vuole conservare un altro file/cartella locale non tracciato,
aggiungere una esclusione `-e nome/` sia alla simulazione sia al comando
reale, oppure farne una copia fuori dal repository. Se `git ls-files`
o `git ls-tree` mostra `.env` o file del venv, **fermarsi**: reset e clean
non li proteggeranno. Controllare anche altre varianti locali, per esempio
`.env.production`, nel repository e sul remoto. Eseguire il reset solo quando
`fetch` e i controlli
sono riusciti e si accetta di perdere il lavoro locale:

```powershell
git reset --hard origin/main
git clean -fd -e venv/ -e .venv/ -e env/ -e ENV/ -e .env -e .env.* -e .secrets/ -e backups/
git status
```

Usare il branch remoto corrispondente al proprio branch, non dare per
scontato `main`. `git reset --hard` recupera i file tracciati eliminati e
annulla anche le modifiche gia preparate con `git add`; `git clean -fd`
elimina file e directory non tracciati **non ignorati**. `-e` aggiunge
esclusioni per i nomi elencati; **non usare `git clean -fdx`**, che include
anche i file ignorati. Un file ignorato ma gia tracciato non e protetto.
Se occorre conservare anche i commit locali, non usare questa procedura.

## C. Vedere e recuperare versioni precedenti

Visualizzare la cronologia (premere `q` per uscire dal visualizzatore):

```powershell
git fetch origin
git log --oneline --decorate --all -20
git show --stat CODICE_COMMIT
git --no-pager diff CODICE_COMMIT..HEAD --stat
```

Per esplorare una versione storica **senza toccare la cartella attuale**,
creare una cartella separata con un worktree (scegliere un percorso libero):

```powershell
git worktree add --detach "..\versione-storica" CODICE_COMMIT
```

La cartella storica contiene solo i file versionati; per rimuoverla quando
non serve piu, dalla cartella del progetto usare
`git worktree remove "..\versione-storica"` dopo aver verificato che non
contenga modifiche da conservare. Non copiare automaticamente `.env` o
credenziali nel worktree.

Per ottenere una copia indipendente della cronologia completa, clonare in
una cartella nuova e passare al commit desiderato:

```powershell
git clone https://github.com/UTENTE/NOME_REPOSITORY.git "..\progetto-storico"
Set-Location "..\progetto-storico"
git switch --detach CODICE_COMMIT
```

`detached HEAD` e normale per la sola consultazione. Per **sviluppare**
partendo da quel commit, nella copia storica usare
`git switch -c nome-nuovo-branch`. Per ripristinare un singolo file nella
cartella corrente (sovrascrive eventuali modifiche a quel file), controllare
prima con `git status`, quindi:

```powershell
git restore --source=CODICE_COMMIT -- percorso\file
git diff -- percorso\file
```

Per ritrovare un commit locale perso dopo un reset, consultare
`git reflog` **prima** che Git elimini gli oggetti non piu referenziati;
non e un sostituto affidabile di un backup.

## Promemoria

| Obiettivo | Comando di partenza |
| --- | --- |
| Controllare lo stato | `git status` |
| Pubblicare una modifica | `git add percorso\file`, `git commit -m "Messaggio"`, `git push origin main` |
| Anteprima dei file da eliminare | `git clean -nd` (aggiungere le esclusioni della sezione B) |
| Tornare alla versione online | Controlli e comandi della sezione B, **distruttivi** |
| Vedere versioni vecchie | `git log --oneline --all -20` |
| Uscire dalla schermata `(END)` | Premere `q` |

I comandi qui documentati **non sono stati eseguiti sul progetto**:
la guida non crea repository, remoti o commit per conto dell'utente.
