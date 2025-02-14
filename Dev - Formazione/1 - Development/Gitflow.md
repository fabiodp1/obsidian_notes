[Gitflow Workflow | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
# WHAT
[[Gitflow]] è un modello di workflow legacy di Git che involve l'utilizzo di branch di feature e più branch primarie.

# HOW
Invece di un singolo [[main]] branch, questo flow utilizza 2 branch per registrare la sotria del progetto.
Il `main` contiene la release ufficiale, e il branch [[develop]] che serve come branch di integrazione per features. Inoltre è conveniente taggare tutti i commit al main con il numero di versione.

Il primo passo è di complementare il default `main` con il branch `develop`.
Quando si utilizza la libreria `git-flow`, lanciando `git flow init` su una repo esistente, verrà creato il branch `develop`.

```terminal
$ git flow init 

Initialized empty Git repository in ~/project/.git/ No branches exist yet. Base branches must be created now. Branch name for production releases: [main] Branch name for "next release" development: [develop] 

How to name your supporting branch prefixes? 
Feature branches? [feature/] 
Release branches? [release/] 
Hotfix branches? [hotfix/] 
Support branches? [support/] 
Version tag prefix? [] 

$ git branch * develop  main
```

## Feature branches

### Creare un feature branch

Ogni nuova feature dovrebbe avere il proprio branch, che può essere pushata nel repo centrale.
Ma invece di sbranciare da `main`, i branch [[feature]] nascono da `develop`, che rappresenta il loro branch padre.
Quando una feature è completa, questa viene mergiata di nuovo in `develop`.

>Le feature non dovrebbero MAI interaggire direttamente con `main`

![[Pasted image 20241125173118.png]]

```
// senza git-flow extension
git checkout develop 
git checkout -b <feature_branch>

// con extension
git flow feature start <feature_branch>
```

### Terminare un feature branch

Quando abbiamo terminato con gli sviluppi della feature, il prossimo passo è fare il merge del feature branch in `develop`:

```
// senza git-flow extension
git checkout develop 
git merge <feature_branch>

// con extension
git flow feature finish <feature_branch>
```

## Release branches

![[Pasted image 20241125173826.png]]

Una volta che `develop` ha abbastanza feature per un rilascio, viene fatto il fork di un [[release]] branch dal branch `develop`.

### Creare un [[release branch]]

>Creando questo branch inizia il prossimo ciclo di release, così non possono più essere aggiunte nuove feature, solo bug-fixes, generazione di documentazione e altre task finalizzate al rilascio.

Una volta pronto, il branch `release` viene mergiato nel `main` e taggato con un numero di versione. Inoltre dovrebbe essere mergiato di nuovo in `develop`, che potrebbe essere andato avanti nel metre che è iniziato il ciclo di release.

Utilizzando un branch dedicato per preparare le release, rende possibile la rifinitura della release corrente, mentre il resto del team può continuare a lavorare alle feature per il prossimo release.

```
// senza git-flow extension
git checkout develop 
git checkout -b release/0.1.0

// con extension
$ git flow release start 0.1.0 
Switched to a new branch 'release/0.1.0'
```

### Terminare un release branch

Una volta che la release è pronta, il branch verrà mergiato in `main` e `develop`, dopo di che il branch release verrà cancellato.
E' importante che venga fatto il re-merge in `develop` poichè potrebbero essere state fatte delle modifiche importanti nel branch `release` e devono essere accessibili alle nuove feature.

```
// senza git-flow extension
git checkout main 
git merge release/0.1.0

// con extension
git flow release finish '0.1.0'
```

## Hotfix branches

![[Pasted image 20241126091904.png]]

I branch `hotfix` o di mantenimento vengono usati per fare una patch veloce della release in produzione. Sono simili a branch `release` e `feature` eccetto il fatto che si basano sul `main` invece di `develop`.
E' l'unico branch che dovrebbe fare il fork direttamente da `main`. Una volta che il fix è completo, va mergiato sia in main che in develop (o il branch `release` corrente), e `main` andrebbe taggato con un numero di versione aggiornato.

Avere un linea dedicata per i bug fix, permette al team di risolvere problemi senza interrompere il resto del workflow o evita di dover aspettare il prossimo ciclo di release.

### Creare un hotfix branch

```
// senza git-flow extension
git checkout main 
git checkout -b hotfix_branch

// con extension
$ git flow hotfix start hotfix_branch
```

### Terminare un hotfix branch

```
// senza git-flow extension
git checkout main 
git merge hotfix_branch 
git checkout develop 
git merge hotfix_branch 
git branch -D hotfix_branch

// con extension
git flow hotfix finish hotfix_branch
```