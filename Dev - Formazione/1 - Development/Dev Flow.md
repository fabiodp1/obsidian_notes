Ecco il **flow completo e dettagliato**, con i comandi specifici, e le istruzioni per Terminale, VS Code, Fork e GitFlow.

---

# 0. Inizio di una Nuova Feature

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
# 1. Sviluppo Giornaliero

---
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

---

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
# 2. Preparazione della Pull Request (PR)

---

## 1. Squash dei Commit (Opzionale)

### Terminale (Squash Locale)

```bash
git rebase -i origin/develop  # Avvia un rebase interattivo
# Sostituisci "pick" con "squash" o "fixup" per i commit da unire
git push origin feature/FEATURE_NAME --force-with-lease
```

### GitHub (Squash durante il Merge)

1. Durante la creazione della PR su GitHub, seleziona **"Squash and Merge"** come metodo di merge.  
2. Combina tutti i commit in un unico messaggio.

### Fork (App)

1. Fai clic destro sul branch nel grafico da cui fare il rebase (es. development).  
2. Seleziona **"Interactive Rebase"**.  
3. Usa le opzioni **"Squash"** o **"Fixup"** per unire i commit.

---

### **Step 7: Creazione della Pull Request**

#### **Terminale (Con GitHub CLI)**

```bash
gh pr create --base develop --head feature/FEATURE_NAME --title "Titolo PR" --body "Descrizione"
```

#### **VS Code**
1. Installa l'estensione **"GitHub Pull Requests and Issues"**.  
2. Apri la palette comandi (Ctrl+Shift+P).  
3. Cerca **"Create Pull Request"** e segui la procedura guidata.

#### **Fork (App)**
1. Clicca sul pulsante **"Create Pull Request"** nella barra superiore.  
2. Compila titolo, descrizione e seleziona `develop` come branch di destinazione.

---
# 3. Gestione dei Branch Condivisi (⚠️ Avvisi Critici)

---

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

## Comandi di Emergenza

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
# 4. Comandi GitFlow Avanzati

---

### **Completamento di una Feature**
```bash
git flow feature finish FEATURE_NAME  # Merge su develop ed elimina il branch locale
```

### **Avvio di una Release**
```bash
git flow release start RELEASE_NAME
```

### **Avvio di un Hotfix**
```bash
git flow hotfix start HOTFIX_NAME
```

---

## **Strumenti Consigliati**
| Operazione               | Terminale                  | VS Code                                | Fork (App)                          |
|--------------------------|----------------------------|----------------------------------------|-------------------------------------|
| **Gestione Conflitti**   | Editor manuale             | Merge Editor integrato                 | Strumento di merge a 3 vie          |
| **Visualizzazione Log**  | `git log --oneline --graph`| Estensione GitLens                     | Grafico interattivo dei branch      |
| **Configurazione**       | `git config --global ...`  | Impostazioni → Git                     | Preferences → Git                   |

---

## **Best Practice Finali**
1. **Rebase frequente**: Esegui `git fetch + rebase` almeno una volta al giorno.  
2. **Commit atomici**: Suddividi le modifiche in commit logici e mirati.  
3. **`--force-with-lease` > `-f`**: Usa sempre l'opzione sicura per i push.  
4. **Branch personali**: Preferisci branch individuali per evitare conflitti.  