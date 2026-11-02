# Guida pratica a Git e GitHub su Windows

Questa guida spiega come installare e configurare **Git** e **GitHub CLI (`gh`)**, creare repository pubblici e privati e gestire versioni, aggiornamenti e ripristini dal terminale **PowerShell** (anche quello integrato in Zed o VS Code).

> **Differenza:** Git salva e versiona il codice sul computer; GitHub ospita i repository online. GitHub CLI (`gh`) serve per creare e gestire repository GitHub direttamente dal terminale.

## 1. Installare Git

Aprire **PowerShell**:

```powershell
winget install --id Git.Git -e
```

Chiudere e riaprire PowerShell e verificare:

```powershell
git --version
```

In alternativa: [sito ufficiale Git](https://git-scm.com/downloads/win).

## 2. Configurare Git (uno o più account GitHub)

Git permette di utilizzare più account GitHub sullo stesso PC. È importante distinguere:

- **Identità Git:** nome ed email associati ai commit.
- **Account GitHub:** account autenticato per creare repository e pubblicare codice.

### 2.1 Configurazione globale (account principale)

Impostare nome ed email predefiniti per tutti i repository:

```powershell
git config --global user.name "ElioErrico"
git config --global user.email "EMAIL_PERSONALE"
git config --global init.defaultBranch main
```

Queste impostazioni vengono utilizzate nei repository che non possiedono una configurazione locale specifica.

### 2.2 Configurazione locale (secondo account)

Quando si lavora su un progetto associato all'altro account GitHub:

```powershell
cd "C:\Progetti\ProgettoLavoro"

git config --local user.name "USERNAME_LAVORO"
git config --local user.email "EMAIL_LAVORO"
```

Le impostazioni locali si applicano **solamente al repository corrente** e prevalgono su quelle globali.

Se il repository è appena stato creato e non è ancora inizializzato, eseguire prima `git init`.

### 2.3 Verificare l'identità Git

```powershell
git config --global --list
git config --list --show-origin
git config user.name
git config user.email
```

Per sapere da quale configurazione provengono nome ed email:

```powershell
git config --show-origin --get user.name
git config --show-origin --get user.email
```

Il nome e l'email sono metadati dei commit, non credenziali. Usare un'email verificata nel corrispondente account GitHub oppure il relativo indirizzo `noreply`.

### 2.4 Verificare Git Credential Manager

Su Git per Windows è generalmente già disponibile Git Credential Manager:

```powershell
git config --show-origin --get-all credential.helper
```

Se appare `manager`, il gestore è configurato.

**Non utilizzare `git config` per inserire password o token GitHub.**

---

## 3. Installare GitHub CLI e gestire più account

GitHub CLI (`gh`) permette di autenticare più account GitHub contemporaneamente e di passare rapidamente da uno all'altro.

### 3.1 Installare GitHub CLI

```powershell
winget install --id GitHub.cli -e
```

Riaprire PowerShell e verificare:

```powershell
gh --version
```

### 3.2 Creare un secondo account GitHub

Se il secondo account non esiste ancora:

1. Aprire https://github.com/signup
2. Registrarsi con un indirizzo email differente.
3. Scegliere un nuovo username e una password.
4. Verificare l'indirizzo email.
5. Configurare preferibilmente l'autenticazione a due fattori (2FA).

La creazione dell'account avviene sul sito GitHub, non attraverso `git config`.

Se entrambi gli account esistono già, saltare questo passaggio.

### 3.3 Autenticare il primo account

Aprire PowerShell:

```powershell
gh auth login --web --git-protocol https
```

Seguire le istruzioni e autorizzare il primo account dal browser.

Quando richiesto, consentire l'utilizzo delle credenziali anche per Git.

Verificare:

```powershell
gh auth status
```

### 3.4 Aggiungere il secondo account

Eseguire nuovamente:

```powershell
gh auth login --web --git-protocol https
```

Nel browser accedere con il **secondo account GitHub** e completare l'autorizzazione.

Se il browser apre automaticamente il primo account, cambiare account dal selettore del sito GitHub prima di autorizzare la CLI.

Verificare che entrambi siano presenti:

```powershell
gh auth status
```

Esempio:

```text
github.com
  ✓ Logged in as ElioErrico
  ✓ Logged in as AccountLavoro
```

I due account rimangono memorizzati. Non è necessario disconnettersi ogni volta.

### 3.5 Passare da un account all'altro (SWITCH)

Per selezionare l'account personale:

```powershell
gh auth switch --user ElioErrico
```

Per selezionare l'account lavorativo:

```powershell
gh auth switch --user AccountLavoro
```

In alternativa, utilizzare la selezione interattiva:

```powershell
gh auth switch
```

Verificare l'account attivo:

```powershell
gh auth status --active
```

**Importante:** `gh auth switch` modifica l'account autenticato per GitHub CLI. Non modifica automaticamente `git config user.name` e `git config user.email`.

### 3.6 Configurare l'autenticazione Git tramite GitHub CLI

Per utilizzare le credenziali gestite da GitHub CLI anche con `git push`, `git pull` e `git clone`:

```powershell
gh auth setup-git
```

Questo configura Git affinché usi GitHub CLI come gestore delle credenziali per gli host GitHub autenticati.

Con questa configurazione, per i repository HTTPS di GitHub.com le operazioni Git utilizzano normalmente le credenziali dell'account attivo in `gh`.

**Attenzione:** cambiando account con `gh auth switch`, cambia anche l'account utilizzato dall'helper GitHub CLI per l'autenticazione HTTPS. Pertanto, prima di un `push`, assicurarsi di aver selezionato l'account che ha accesso al repository.

Se invece si continua a utilizzare Git Credential Manager come helper effettivo, lo switch di `gh` potrebbe non modificare le credenziali usate da Git. In quel caso verificare la configurazione degli helper e le credenziali memorizzate.

### 3.7 Procedura quotidiana per cambiare account

**Lavorare con l'account personale:**

```powershell
gh auth switch --user ElioErrico
gh auth status --active

cd "C:\Progetti\ProgettoPersonale"
git config user.email
git remote -v
```

**Lavorare con l'account lavorativo:**

```powershell
gh auth switch --user AccountLavoro
gh auth status --active

cd "C:\Progetti\ProgettoLavoro"
git config user.email
git remote -v
```

Se nome ed email sono già configurati localmente, non occorre reimpostarli ogni volta.

---

## 5. Creare un nuovo repository PUBBLICO su GitHub

**Scenario:** il codice è già in una cartella sul PC, ma il repository non esiste ancora su GitHub.

### 5.1 Selezionare l'account GitHub

Prima di creare il repository, scegliere l'account proprietario.

**Account personale:**

```powershell
gh auth switch --user ElioErrico
```

**Account lavorativo:**

```powershell
gh auth switch --user AccountLavoro
```

Verificare:

```powershell
gh auth status --active
```

### 5.2 Entrare nella cartella del progetto

```powershell
cd "C:\Progetti\MioProgetto"
```

Se la cartella non è ancora un repository Git:

```powershell
git init
```

### 5.3 Configurare l'identità corretta per il progetto

Per il progetto personale:

```powershell
git config --local user.name "ElioErrico"
git config --local user.email "EMAIL_PERSONALE"
```

Oppure, per il progetto lavorativo:

```powershell
git config --local user.name "AccountLavoro"
git config --local user.email "EMAIL_LAVORO"
```

Verificare:

```powershell
git config user.name
git config user.email
```

Questa configurazione rimane associata al repository anche dopo aver cambiato account GitHub CLI.

### 5.4 Creare il primo commit

```powershell
git branch -M main
git add .
git status
git commit -m "Initial commit"
```

Prima di `git add .`, verificare di aver creato correttamente `.gitignore` e di non includere credenziali.

### 5.5 Creare il repository online e pubblicarlo

Per creare un repository pubblico nell'account attivo:

```powershell
gh repo create MioProgetto --public --source=. --remote=origin --push
```

In alternativa, per indicare esplicitamente il proprietario:

```powershell
gh repo create ElioErrico/MioProgetto --public --source=. --remote=origin --push
```

Oppure:

```powershell
gh repo create AccountLavoro/MioProgetto --public --source=. --remote=origin --push
```

L'account attivo deve avere i permessi necessari per creare il repository nel proprietario specificato.

Se il nome del proprietario non coincide con l'account attivo, verificare attentamente i permessi prima di procedere.

### 5.6 Verificare il repository remoto

```powershell
git remote -v
gh repo view --web
```

Controllare che il repository appartenga all'account desiderato.

---

## Promemoria rapido: gestione di due account

| Operazione | Comando |
|---|---|
| Aggiungere un account | `gh auth login --web --git-protocol https` |
| Elencare gli account | `gh auth status` |
| Cambiare account | `gh auth switch` |
| Selezionare account personale | `gh auth switch --user ElioErrico` |
| Selezionare account lavoro | `gh auth switch --user AccountLavoro` |
| Account attivo | `gh auth status --active` |
| Configurare Git con GitHub CLI | `gh auth setup-git` |
| Impostare email del progetto | `git config --local user.email "EMAIL"` |
| Verificare autore commit | `git config user.name` |
| Verificare email commit | `git config user.email` |
| Verificare repository remoto | `git remote -v` |
| Rimuovere un account dalla CLI | `gh auth logout --user USERNAME` |

**Regola generale:** ogni repository mantiene le proprie impostazioni locali di nome ed email, mentre `gh auth switch` seleziona l'account con cui eseguire le operazioni GitHub.

Questo consente di utilizzare account personale e lavorativo senza modificare continuamente l'identità dei commit.

## 6. Creare un nuovo repository PRIVATO su GitHub

Stessa procedura, con `--private` invece di `--public`:

```powershell
cd "C:\Progetti\MioProgetto"
git init
git branch -M main
git add .
git status
git commit -m "Initial commit"
gh repo create MioProgetto --private --source=. --remote=origin --push
```

> **Scegliere UNA sola** delle procedure dei punti 5 e 6 per la stessa nuova cartella. Se il repository esiste già su GitHub, non usare `gh repo create` per ricrearlo.


## 7. Clonare sul PC un repository già presente su GitHub

```powershell
cd "C:\Progetti"
git clone https://github.com/TUO_USERNAME/MioProgetto.git
cd MioProgetto
```

Per repository privati, l'account autenticato deve avere i permessi di accesso. Il comando di clone è lo stesso.

## 8. Pubblicare le modifiche di un repository esistente

Nella cartella del progetto:

```powershell
git status                 # Controlla cosa è cambiato
git diff                   # Mostra differenze non ancora messe in stage
git add .                  # Prepara le modifiche
git diff --cached          # Verifica cosa entrerà nel commit
git commit -m "Correzione bug"
git pull --rebase          # Integra eventuali nuovi commit remoti
git push                   # Pubblica i commit
```

Se `git pull --rebase` segnala conflitti, risolverli prima di eseguire `git push`. Nel caso di repository appena creato senza upstream configurato, eseguire `git push -u origin main` la prima volta.

**Da ricordare:** `add` prepara, `commit` salva nella cronologia **locale**, `push` pubblica su GitHub.

## 9. Riallineare il repository locale a GitHub

Per sostituire lo stato del branch locale corrente con quello di `main` su GitHub, scaricando prima i riferimenti aggiornati:

```powershell
git fetch origin && git reset --hard origin/main && git clean -fd
```

Oppure se hai errore powershell

```powershell
git fetch origin
if ($LASTEXITCODE -ne 0) { throw "git fetch non riuscito" }

git reset --hard origin/main
if ($LASTEXITCODE -ne 0) { throw "git reset non riuscito" }

git clean -fd
```

**ATTENZIONE: comando distruttivo.** Prima fare un backup e verificare lo stato con `git status`. Il comando:

- `git fetch origin` aggiorna i riferimenti locali ai branch remoti, senza modificare i file di lavoro.
- `git reset --hard origin/main` sposta il branch **attualmente selezionato** al commit di `origin/main` e scarta le modifiche locali ai file tracciati e gli eventuali commit locali non presenti su GitHub. Non esegue `git switch main`.
- `git clean -fd` elimina i file e le cartelle non tracciati da Git. Non elimina i file ignorati; per vedere prima cosa verrebbe rimosso, eseguire `git clean -nd`.

La sequenza presuppone che esistano il remote `origin` e il branch remoto `main`. In PowerShell 7 o successivo, `&&` esegue ciascun comando solo se quello precedente è riuscito. In Windows PowerShell 5.1, eseguire le tre righe separatamente e proseguire solo se ciascuna termina senza errori.

Questo non è un normale aggiornamento: per scaricare e integrare le novità senza cancellare il lavoro locale, usare `git pull` come descritto nel capitolo seguente.

## 10. Scaricare gli aggiornamenti da GitHub

Se la cartella non contiene modifiche locali da conservare:

```powershell
git pull
```

Per vedere le novità **senza integrarle subito**:

```powershell
git fetch origin
git log HEAD..origin/main --oneline
```

Per mettere temporaneamente da parte anche i nuovi file non tracciati:

```powershell
git stash push -u -m "Lavoro temporaneo"
git pull
git stash pop
```

L'applicazione dello stash può generare conflitti.

## 11. Ripristinare il codice dopo modifiche sbagliate

### A. Eliminare modifiche locali NON ancora registrate in un commit

Per ripristinare i file tracciati all'ultimo commit:

```powershell
git restore .
```

Se hai già usato `git add .` e vuoi annullare anche le modifiche in stage:

```powershell
git restore --source=HEAD --staged --worktree .
```

I file nuovi non tracciati **non vengono eliminati**.

### B. Tornare esattamente alla versione su GitHub (`main`)

**ATTENZIONE: operazione distruttiva.** Perde le modifiche locali ai file tracciati e sposta il branch, abbandonando gli eventuali commit locali non presenti sul remoto.

Prima fare un backup della cartella, poi, se stai lavorando su `main`:

```powershell
git fetch origin
git switch main
git reset --hard origin/main
```

Questa procedura **non elimina** file locali non tracciati. Se desideri eliminarli, verifica prima l'anteprima:

```powershell
git clean -nd    # Mostra cosa verrebbe eliminato
git clean -fd    # Elimina i file non tracciati: irreversibile
```

Non usare `git clean -fdx` senza conoscere le conseguenze: elimina anche file ignorati, per esempio `.env` e ambienti virtuali.

### C. Scaricare una copia nuova senza toccare la cartella danneggiata

```powershell
cd "C:\Progetti"
git clone https://github.com/TUO_USERNAME/MioProgetto.git MioProgetto_PULITO
```

Questa è spesso l'opzione più sicura.

### D. Annullare un commit già pubblicato senza riscrivere la cronologia

```powershell
git revert HEAD
git push
```

## 12. Eliminare i `venv` già tracciati senza cancellarli dal PC

Aggiungere prima `venv/`, `.venv/` e gli altri percorsi corretti nel `.gitignore`. Poi, **se la cartella si chiama proprio `venv` o `.venv`**:

```powershell
git rm -r --cached --ignore-unmatch -- venv .venv
git add .gitignore
git status
git commit -m "Stop tracking virtual environments"
git push
```

Se gli ambienti hanno nomi diversi, ripetere `git rm -r --cached --ignore-unmatch -- NOME_CARTELLA` per ciascuna cartella già tracciata.

`--cached` rimuove i file dall'indice Git, **non dal disco**. I file restano però nei commit precedenti: ripulire la cronologia richiede una procedura separata e coordinamento con eventuali collaboratori.

## 13. Branch: lavorare su nuove funzionalità

```powershell
git switch -c feature/nuova-funzione  # Crea e apre un branch
# ... modifica il codice ...
git add .
git commit -m "Implementa nuova funzione"
git push -u origin feature/nuova-funzione
```

Per integrare localmente il branch in `main`:

```powershell
git switch main
git pull
git merge feature/nuova-funzione
git push
```

In alternativa si può aprire una **Pull Request** su GitHub, particolarmente utile quando collaborano più persone.

## 14. Rinominare il repository su GitHub

Su GitHub aprire il repository → **Settings** → **General** → **Repository name** → **Rename**.

Aggiornare il collegamento locale:

```powershell
git remote set-url origin https://github.com/TUO_USERNAME/NUOVO_NOME.git
git remote -v
```

La cartella sul PC può mantenere il vecchio nome; rinominarla è facoltativo.

## 15. Tabella rapida dei comandi

| Operazione | Comando |
|---|---|
| Versione installata | `git --version` |
| Configurazione | `git config --global --list` |
| Stato del progetto | `git status` |
| Differenze non preparate | `git diff` |
| File preparati per il commit | `git diff --cached` |
| Aggiungere modifiche | `git add .` |
| Creare un commit | `git commit -m "Messaggio"` |
| Pubblicare commit | `git push` |
| Scaricare e integrare commit | `git pull` |
| Scaricare novità senza integrarle | `git fetch origin` |
| Clonare progetto | `git clone URL` |
| Ultimi commit | `git log --oneline -10` |
| Branch attuale | `git branch --show-current` |
| Elenco branch | `git branch -a` |
| Nuovo branch | `git switch -c nome-branch` |
| Passare a un branch | `git switch nome-branch` |
| Mostrare i remoti | `git remote -v` |
| Controllare l'accesso `gh` | `gh auth status` |
| Creare repository pubblico | `gh repo create NOME --public --source=. --remote=origin --push` |
| Creare repository privato | `gh repo create NOME --private --source=. --remote=origin --push` |
| Cronologia movimenti di HEAD | `git reflog` |
| Mettere da parte modifiche | `git stash push -u` |
| Riapplicare lo stash | `git stash pop` |

## 16. Problemi frequenti

- **`fatal: not a git repository`**: aprire il terminale nella cartella corretta oppure inizializzare Git se è un progetto nuovo.
- **`nothing to commit`**: non ci sono nuove modifiche pronte per il commit.
- **`remote origin already exists`**: il remote è già presente; controllarlo con `git remote -v` e, se necessario, modificarlo con `git remote set-url origin URL`.
- **`repository not found`**: controllare URL, esistenza del repository e autorizzazioni dell'account.
- **`non-fast-forward`** durante il push: recuperare i commit remoti (`git pull --rebase`) e risolvere eventuali conflitti prima di riprovare.
- **Problemi di autenticazione**: controllare `gh auth status` e la configurazione di Git Credential Manager.
- **Il `venv` è finito su GitHub**: aggiungere la cartella a `.gitignore`, rimuoverla dall'indice con `git rm --cached` e fare un nuovo commit.

## 17. Buone pratiche

1. Controllare sempre `git status` e, quando necessario, `git diff --cached` prima di un commit.
2. Fare commit piccoli e con descrizioni precise.
3. Preparare `.gitignore` **prima** del primo caricamento.
4. Non versionare ambienti virtuali, cache, token, password e chiavi private.
5. Creare un branch o un commit di sicurezza prima di far modificare ampie parti di codice a un agente AI.
6. Non usare `git push --force`, `git reset --hard` o `git clean -fd` senza aver verificato l'impatto sui file.
7. Non considerare GitHub un backup completo: i file ignorati o non committati non vi sono salvati.

## Documentazione ufficiale

- [Git – Download per Windows](https://git-scm.com/downloads/win)
- [Libro Pro Git](https://git-scm.com/book/en/v2)
- [GitHub CLI – Manuale](https://cli.github.com/manual/)
- [GitHub CLI – `gh repo create`](https://cli.github.com/manual/gh_repo_create)
- [GitHub – Documentazione Git](https://docs.github.com/en/get-started/using-git)
