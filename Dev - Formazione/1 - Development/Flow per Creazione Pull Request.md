# 0. Preparazione del Branch per la PR

---
## 1. Rebase finale su `develop`

```bash
git fetch origin develop
git rebase origin/develop  # Allinea il branch alla versione più recente di develop
git push origin feature/FEATURE_NAME --force-with-lease
```

---

## **1. Creazione della Pull Request**
### **Requisiti Obbligatori (Checklist Aziendale)**
1. ✅ Branch rebasato su `develop` (vedi Step 0a).  
2. ✅ Target branch corretto:  
   - **Feature branch** → `develop`  
   - **Release/hotfix branch** → `main`/`production`  
3. ✅ Descrizione PR include:  
   - Link all'issue tracker (es. Jira, Trello)  
   - Link alla documentazione di specifica  
   - Descrizione "umana" delle modifiche  
   - Dati di esempio/screenshot per testare  
4. ✅ Contrassegna come **WIP** se non pronto per il merge.  
5. ✅ Assegna almeno 1 revisore (o notifica il team leader).

---

### **Metodo 1: Terminale (Git + GitHub CLI)**
#### **Step 1: Crea la PR**
```bash
gh pr create \
  --base develop \                     # Target branch (develop per le feature)
  --head feature/FEATURE_NAME \        # Branch sorgente
  --title "FEATURE_NAME: Descrizione" \  
  --body "**Issue Tracker:** [LINK-ISSUE]
          **Documentazione:** [LINK-DOC]
          **Modifiche:** 
          - Descrizione delle modifiche effettuate
          - Motivazione delle scelte tecniche
          **Test:** 
          ```python
          # Esempio di codice per riprodurre il comportamento
          ```
          ![Screenshot](https://example.com/screenshot.png)" \
  --reviewer @UtenteGitHub \           # Assegna revisore
  --draft                              # Segnala come WIP (opzionale)
```

#### **Step 2: Verifica la PR**
```bash
gh pr view --web  # Apri la PR nel browser per controllare i dettagli
```

