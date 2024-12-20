[How to Create a Git Repository | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/setting-up-a-repository)
# Inizializzare un progetto

- [[git init]] : per creare un nuovo repository da 0

```console
git init

OR

git init <directoryPath>
```

- [[git clone]] : per clonare in locale un repository esistente

```console
git clone <repo url>
```

# Salvare le modifiche

In realtà il termine "salvare" in Git non è molto appropriato, il suo corrispettivo sarebbe "committing". Un commit sarebbe l'equivalente di un salvataggio.

- [[git add]] : aggiunge una modifica della working directory alla [[staging area]]. Dice a Git che vogliamo includere degli aggiornamenti a un certo file nel prossimo commit. Non modifica effettivamente la repository, i cambiamenti non vengono veramente applicati fino a quando non viene fatto [[git commit]] .
- [[git status]] : viene utilizzato per vedere la situazione della working directory e la [[staging area]]
- [[git push]] : serve ad inviare al remote repository i cambiamenti committati, permettendo ai collaboratori di accedervi.

Bisogna pensare alla [[staging area]] come ad un buffer che si trova fra la working directory e la project history.
Invece di committare tutti i cambiamentii fatti dall'ultimo commit, lo [[stage]] permette di raggruppare i relativi cambiamenti in snapshot altamente focalizzati, prima che questi vengano committati nel project history.
Ciò vuol dire che posso fare qualsiasi tipo di modifica a file non in relazione fra loro, per poi tornare indietro e dividerli in raggruppamenti logici di commit, aggiungendo cambiamenti in relazione fra loro allo stage e committandoli pezzo per pezzo.

```terminal
git add <file> // verranno messi in stage per il prossimo commit tutti i                             cambiamenti al file indicato

git add <directory> // stessa cosa ma con la directory

git add -p // avvierà una sessione di staging interattivo
```

- [[git commit]] : utilizzato per salvare lo snapshot delle modifiche presente in [[stage]]

```terminal
git commit -m "feat: [#69] commit message"
```

# Comandi per collaborazione e workflow

- [[git fetch]]: serve per fare il download dei commit, file e ref da un repository remoto verso il proprio repo locale. la differenza fra i due è che mentre `git pull` farà un fetch e poi merge, `git fetch` è la versione 'safe', farà solo il fetch dei dati, lasciando all'utente la gestione del merge.
  E' una sorta di update, ma solo per sincronizzare la situazione del locale con quella del remoto.

```
git fetch <romote>
git fetch <remote> <branch>
git fetch --all // fetch di tutto quello che c'è a remoto
```

- [[git pull]] : prima di tutto lancia `git fetch`, dopo `git merge` per fare il merge del contenuto remoto in un nuovo [[merge commit]] locale.

```
git pull <remote> // equivale a `git fetch ＜remote＞` seguito da `git merge                           origin/＜current-branch＞`
```

- [[git branch]] : viene utilizzato per la gestione dei branch. Permette di crearne di nuovi branch, rimuoverli, mostrarne una lista o rinominarli.

```
git branch // una lista di tutti i branch nel repository

git branch <branch> // crea un nuovo branch

git branch -d <branch> // cancella il branch indicato in modo SAFE

git branch -D <branch> // FORZA la cancellazione del branch, anche se ha                                     cambiamenti non mergiati

git branch -m <branch> // rinomina il branch corrente

git branch -a // lista di tutti i branch remoti

```

- [[git checkout]] : permette di navigare fra i branch. Aggiornerà i file nella working directory per essere sincronizzati con la versione presente in quel branch, e dice a Git di registrare i nuovi commit su quel branch.
	-  E' legato a [[git branch]], infatti per creare un nuovo branch, è possibile creare il nuovo branch utilizzando [[git branch]] e poi facendo [[git checkout]] su quel branch, oppure esiste una shortcut per creare il branch e fare direttamente il checkout in esso:
	
```
git checkout -b <new-branch>
```

- [[git push]] : utilizzato per caricare il contenuto del repository locale su un repository remoto. Trasferisce i commit del repo locale ad uno remoto.
  Potenzialmente può sovrascrivere i cambiamenti già presenti a remoto, per questo bisogna stare attenti quandi si pusha.
  
```
git push <remote> <branch>
```

- [[git merge]] : è un modo per rimettere assieme una history che ha subito un fork. Permette di prendere le linee indipendenti di sviluppo create da `git branch` e integrarle in un singolo branch.
  `git merge` combinerà multiple sequenze di commit in un'unica storia unificata. Prima di lanciare un `git merge` è importante effettuare degli step di preparazione
	  1. Eseguire un [[git status]] per accertarsi che [[HEAD]] stia puntando al corretto branch che riceverò il merge. Se necessario fare un [[git checkout]] sul branch.
	  2. Assicurarsi che il branch che riceverà il merge e il branch che verrà mergiato siano aggiornati. Eseguire un [[git fetch]] per fare tirare giù tutti i commit remoti. Una volta che il fetch è completo, assicurarsi che il branch che riceverà il merge sia anche questo aggiornato eseguendo un [[git pull]].
	  3. Effettuare un [[git merge]] seguito dal nome del branch che sarà mergiato in quello corrente.
## [[pull request]]

Rende più facile la collaborazione fra sviluppatori.
Una pull request notifica il team del completamento una una feature. Infatti una volta che il feature branch è pronto, lo sviluppatore fa una pull request, in questa maniera chi di dovere sa che dovrà revisionare il codice e farne il merge nel main branch.

In realtà è più di questo, è un vero e proprio forum in cui si può discutere le feature proposte. Se ci sono problemi con i cambiamenti fatti, i colleghi possono postare feedback nella pull request o persino apportare modifiche alla funzionalità inviando follow-up commit. Tutta questa attività viene registrata nella pull request.