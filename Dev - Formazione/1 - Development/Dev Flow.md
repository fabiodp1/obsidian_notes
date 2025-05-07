Ecco il **flow completo e dettagliato**, con i comandi specifici, e le istruzioni per Terminale, VS Code, Fork e GitFlow.

---

# Inizio di una Nuova Feature

## Creazione del Feature Branch
### Terminale (Con GitFlow)

```bash
git flow feature start FEATURE_NAME  # Crea un branch da develop e effettua il checkout automatico
```

### VS Code

1. Installa l'estensione **GitFlow** (es. "GitHub Pull Requests and Issues" o "GitFlow").  
2. Apri la palette comandi (Ctrl+Shift+P / Cmd+Shift+P).  
3. Cerca "GitFlow: Start Feature" e inserisci il nome della feature.

### Fork (App)

1. Fai clic destro sul branch `develop` nel grafico dei branch.  
2. Seleziona **"New Branch"**.  
3. Assegna il nome `feature/FEATURE_NAME` e conferma.

---
# Sviluppo Giornaliero

## 1. Commit delle Modifiche

### Terminale

```bash
git add .                          # Aggiungi tutte le modifiche
git commit -m "Descrizione commit" # Crea un commit
```

### VS Code

1. Apri la scheda **Source Control** (Ctrl+Shift+G / Cmd+Shift+G).  
2. Seleziona i file da includere nel commit.  
3. Inserisci il messaggio di commit nella barra superiore.  
4. Clicca sull'icona ✔️ per confermare il commit.

### Fork (App)

1. Vai alla sezione **"Changes"**.
2. Seleziona i file da commitare.  
3. Inserisci il messaggio di commit nel campo in basso.  
4. Clicca su **"Commit"**.

---
## 2. Fetch del Branch `develop` Remoto

### Terminale

```bash
git fetch origin develop  # Scarica l'ultima versione di develop dal remote
```

### VS Code

1. Clicca sui tre puntini (⋯) nella scheda **Source Control**.  
2. Seleziona **"Fetch"** dal menu a tendina.  
3. Scegli **"origin"** come remote.

### Fork (App)

1. Clicca sul pulsante **"Fetch"** nella barra superiore.  
2. Seleziona **"Fetch All"** per aggiornare tutti i branch remoti.

---
## 3. Rebase del Feature Branch su `origin/develop`

### Terminale

```bash
git checkout feature/FEATURE_NAME  # Assicurati di essere sul tuo feature branch
git rebase origin/develop          # Allinea il branch alla versione remota di develop
```

### VS Code

1. Clicca sui tre puntini (⋯) nella scheda **Source Control**.  
2. Seleziona **"Rebase onto..."** dal menu.  
3. Scegli `origin/develop` come base.

### Fork (App)

1. Fai clic destro su `origin/develop` nel grafico dei branch.  
2. Seleziona **"Rebase [feature/FEATURE_NAME] onto origin/develop"**.

### 3.1. Risoluzione Conflitti (Se Necessaria)

#### Terminale

1. Modifica manualmente i file in conflitto.  
2. Dopo aver risolto i conflitti:  

   ```bash
   git add <file-risolto>       # Aggiungi ogni file risolto
   git rebase --continue        # Prosegui il rebase
   ```

4. Se vuoi annullare il rebase:  

   ```bash
   git rebase --abort
   ```

#### VS Code

1. Apri i file in conflitto dalla scheda **Source Control**.  
2. Usa i pulsanti:  
   - **"Accept Current Change"** (mantieni le tue modifiche).  
   - **"Accept Incoming Change"** (accetta le modifiche da `develop`).  
3. Clicca su **"Mark as Resolved"** dopo aver risolto ogni conflitto.  
4. Clicca su **"Rebase Continue"** nella notifica in basso a destra.

#### Fork (App)

1. Apri il file in conflitto nell'editor integrato.  
2. Usa lo strumento di merge a 3 vie per confrontare:  
   - La tua versione (a sinistra).  
   - La versione comune (al centro).  
   - La versione da `develop` (a destra).  
3. Clicca su **"Resolve"** dopo aver risolto i conflitti.  
4. Seleziona **"Continue Rebase"** per completare l'operazione.

---

## 4. Push delle Modifiche

### Terminale

```bash
git push origin feature/FEATURE_NAME --force-with-lease  # Push sicuro
```

### VS Code

1. Clicca sui tre puntini (⋯) nella scheda **Source Control**.  
2. Seleziona **"Push (Force)"** dal menu.  
3. Conferma il nome del branch.

### Fork (App)

1. Clicca sul pulsante **"Push"** nella barra superiore.  
2. Abilita la checkbox **"Force Push"**.  
3. Conferma l'operazione.

---
# Flow Creazione Pull Request

## 1. Rebase finale su `develop`**

```bash
git fetch origin develop
git rebase origin/develop  # Allinea il branch alla versione più recente di develop
git push origin feature/FEATURE_NAME --force-with-lease
```

>**VEDI SOPRA** per altri metodi per fare il rebase.

## 2. Creazione della Pull Request

### Requisiti Obbligatori (Checklist Aziendale)**

1. ✅ Branch rebasato su `develop` (vedi Step 1).
2. ✅ Target branch corretto:
    - **Feature branch** → `develop`
    - **Release/hotfix branch** → `main`/`production`
3. ✅ Descrizione PR include:
    - Link all'issue tracker (es. Jira, Trello)
    - Link alla documentazione di specifica
    - Descrizione "umana" delle modifiche
    - Dati di esempio/screenshot per testare
4. ✅ Contrassegna come **WIP** se non pronto per il merge.
5. ✅ Assegna almeno 1 revisore (o notifica il team leader).

---

### VS Code (Estensione GitHub)

#### 1. Installa l'estensione (se non presente)

- Cerca **"GitHub Pull Requests and Issues"** nel marketplace e installala.

#### 2. Crea la PR

1. Apri la palette comandi (Ctrl+Shift+P).
2. Cerca **"Create Pull Request"**.
3. Segui la procedura guidata:
    - **Base branch**: `develop`
    - **Compare branch**: `feature/FEATURE_NAME`
    - **Title**: Includi il nome della feature e una descrizione breve.
    - **Description**: Inserisci:
    
```markdown
**Issue Tracker:** [LINK](https://issue-tracker.com/ISSUE-123)  
**Documentazione:** [Specifiche](https://docs.com/spec)  
**Modifiche:**  
    - Aggiunto sistema di autenticazione OAuth2  
    - Corretto bug nel calcolo degli sconti
```

1. - **Assignees**: Seleziona un revisore dall'elenco.
    - **Labels**: Aggiungi `WIP` se necessario.  
2. Clicca **"Create"**.

### Fork (App)

#### 1. Apri la sezione Pull Requests

1. Clicca sul pulsante **"Pull Requests"** nella barra laterale.
2. Seleziona **"New Pull Request"**.

#### 2. Compila i dettagli

- **Source Branch**: `feature/FEATURE_NAME`
- **Target Branch**: `develop`    
- **Title**: Usa il formato `[FEATURE] Descrizione breve`.
- **Description**:

```markdown
### Collegamenti  
- **Issue:** [ISSUE-456](https://issue-tracker.com/ISSUE-456)  
- **Specifiche:** [Documento Tecnico](https://docs.com/tech-spec)  

### Modifiche  
- Implementata cache Redis per le query frequenti  
- Aggiornato il pacchetto `security-lib` alla v2.1  

### Test  
1. Esegui `npm run test:auth`  
2. Verifica il log delle transazioni in `/var/logs`  
```

- **Options**:
    - ✅ **Draft** (per WIP)
    - **Reviewers**: Aggiungi almeno un revisore.
        
#### 3. Invia la PR

Clicca **"Create Pull Request"**.

---

## 3. Best Practice per la Descrizione della PR

#### Template Consigliato (Markdown)

```markdown
## Collegamenti
- **Issue Tracker:** [ISSUE-123](https://issue-tracker.com/ISSUE-123)
- **Documentazione:** [Specifiche Funzionali](https://docs.com/functional-spec)

## Modifiche
- Aggiunto endpoint API `/v2/payments`  
- Rimosso codice deprecato dal modulo `legacy-utils`  
- **Motivazione:** Migrazione a Stripe per conformità PCI-DSS  
```

---

## 3. Gestione dei WIP (Work in Progress)

- **Terminale**: Usa `--draft` con GitHub CLI: 

```bash
gh pr create --draft
```

- **VS Code**: Seleziona l'opzione **"Create as Draft"** durante la creazione.
- **Fork**: Spunta la checkbox **"Draft"** nel form di creazione PR.
    
---

## 4. Assegnazione Revisori

| Strumento     | Istruzioni                                                         |
| ------------- | ------------------------------------------------------------------ |
| **Terminale** | `gh pr edit --add-reviewer @user1,@user2`                          |
| **VS Code**   | Usa il menu a tendina "Assignees" durante la creazione della PR.   |
| **Fork**      | Clicca su **"Add Reviewers"** e seleziona gli utenti dal dropdown. |

---

## 5. Aggiornamento della PR

- Dopo ulteriori commit, esegui nuovamente il rebase e force push:

```bash
git fetch origin develop
git rebase origin/develop          # Allinea il branch alle ultime modifiche di develop
git push origin feature/FEATURE_NAME --force-with-lease
```
 
- La PR si aggiornerà automaticamente.
- **Cosa fa**: Aggiorna il branch remoto con le nuove modifiche, mantenendo tutti i commit intermedi.

>**Attenzione**: Non fare squash locale! I commit rimangono separati fino al merge finale.
    
---

## 6. Chiusura della PR (Post-Approval)

- **Terminale**:

```bash
git-flow feature finish FEATURE_NAME
```
    
- **VS Code**:
    
    1. Apri la PR nell'estensione GitHub.
    2. Clicca **"Merge"** → **"Squash and Merge"**.
        
- **Fork**:
    
    1. Tasto destro sul branch
    2. GitFlow/finish feature
    3. Opzioni `Delete branches` e `Rebase before merging`
    4. Finish feature

---
# Gestione dei Branch Condivisi (⚠️ Avvisi Critici)

## Regole per Branch Condivisi

1. **Prima di ogni push**:  

   ```bash
   git fetch origin
   git rebase origin/feature/FEATURE_NAME  # Allineati alle modifiche altrui
   ```

2. **Dopo un force push altrui**:  

   ```bash
   git reset --hard origin/feature/FEATURE_NAME  # Sincronizza la tua copia locale
   ```

3. **Comunica sempre** prima di un force push (es. su Slack/Teams).

---

# Comandi di Emergenza

### Annulla un Rebase Fallito

```bash
git rebase --abort
```

### Ripristina il Branch al Remoto

```bash
git fetch origin
git reset --hard origin/feature/FEATURE_NAME
```

---
# Comandi GitFlow Avanzati

## Completamento di una Feature

```bash
git flow feature finish FEATURE_NAME  # Merge su develop ed elimina il branch locale
```

## Avvio di una Release

```bash
git flow release start RELEASE_NAME
```

## Avvio di un Hotfix

```bash
git flow hotfix start HOTFIX_NAME
```

---

# Strumenti Consigliati

| Operazione               | Terminale                  | VS Code                                | Fork (App)                          |
|--------------------------|----------------------------|----------------------------------------|-------------------------------------|
| **Gestione Conflitti**   | Editor manuale             | Merge Editor integrato                 | Strumento di merge a 3 vie          |
| **Visualizzazione Log**  | `git log --oneline --graph`| Estensione GitLens                     | Grafico interattivo dei branch      |
| **Configurazione**       | `git config --global ...`  | Impostazioni → Git                     | Preferences → Git                   |

---

# Best Practice Finali

1. **Rebase frequente**: Esegui `git fetch + rebase` almeno una volta al giorno.  
2. **Commit atomici**: Suddividi le modifiche in commit logici e mirati.  
3. **`--force-with-lease` > `-f`**: Usa sempre l'opzione sicura per i push.  
4. **Branch personali**: Preferisci branch individuali per evitare conflitti.  