La gestione della [[cache]] è sicuramente uno dei problemi più grossi nello sviluppo applicativo.
Lo state può essere raggruppato in due contenitori:
1. [[UI state]]: isModalOpen, isItemHighlighted, etc.
2. [[Server cache]]: User data, tweets, contacts, etc.
La principale causa di complessità e problemi viene quando si cerca di mischiare questi due contenitori. Quando ciò avviene, lo UI state che non dovrebbe essere global viene reso tale poichè il Server cache state è tipicamente global, così naturalmente ci porta a rendere tutto global.

# Differenze tra UI State e Server Cache State

## UI State

- Riguarda lo **stato locale** e temporaneo dell'interfaccia utente.
- Esempi:
    - Una modale aperta/chiusa.
    - Un elemento evidenziato.
    - Una barra laterale visibile.
- Deve essere locale a un componente o una parte specifica dell'app.

## Server Cache State

- Riguarda i **dati recuperati da un server** e solitamente condivisi tra più componenti.
- Esempi:
    - Informazioni sull'utente.
    - Elenchi di post, messaggi o notifiche.
- Di solito è **globale**, perché i dati devono essere accessibili in vari punti dell'app.

---

# Il Problema: Confondere i Due Stati

Se si unisce lo stato dell'interfaccia utente con lo stato della cache del server (ad esempio mettendoli entrambi in uno stato globale come Redux o Context API), emergono i seguenti problemi:

1. **Overhead inutile**:
    - Lo **UI state** di un componente potrebbe attivare aggiornamenti non necessari in altri componenti che usano lo stato globale, creando rallentamenti.
2. **Difficoltà di debugging**:
    - È più difficile capire quale parte del codice aggiorna un particolare pezzo di stato, specialmente se questo è condiviso globalmente.
3. **Architettura rigida**:
	- Lo **UI state**, che è locale per natura, diventa più difficile da gestire in maniera isolata.
1. **Scalabilità compromessa**:
    - Aggiungere nuove funzionalità o stati diventa complicato, perché tutto è intrecciato in un unico store.

---

# La Soluzione: Separare gli Stati
Un approccio migliore è distinguere chiaramente i due tipi di stato:
## Gestione del UI State
- Utilizza **stato locale** o **gestori locali**:
    - `useState`, `useReducer`.
    - Soluzioni come **Zustand** o altre librerie leggere se serve qualcosa di più sofisticato.

## Gestione del **Server Cache State**
- Usa librerie ottimizzate per cache e sincronizzazione con il server:
    - **React Query** (TanStack Query): Gestisce la cache, i refetch e la sincronizzazione in modo automatico.
    - Apollo Client (per GraphQL).

---

# Esempio Pratico

## Scenario:
Vuoi mostrare una lista di utenti e, quando clicchi su un utente, aprire una modale con i dettagli.
## Cosa NON Fare:
Mettere tutto lo stato in Redux/Context:

```ts
const globalState = {
  isModalOpen: false, // UI state
  selectedUserId: null, // UI state
  users: [] // Server cache state
};
```

Questo rende la gestione dello stato inefficiente e il codice meno leggibile.

---
## Cosa Fare:

Separare i due tipi di stato:

```ts
function App() {
  const { data: users } = useQuery('fetchUsers', fetchUsers); // Server cache state

  const [isModalOpen, setIsModalOpen] = useState(false); // UI state
  const [selectedUser, setSelectedUser] = useState(null); // UI state

  const openModal = (user) => {
    setSelectedUser(user);
    setIsModalOpen(true);
  };

  return (
    <div>
      <UserList users={users} onUserClick={openModal} />
      {isModalOpen && <UserModal user={selectedUser} onClose={() => setIsModalOpen(false)} />}
    </div>
  );
}
```

---

# Conclusione

Separando lo **UI state** e lo **server cache state**, ottieni:

- **Maggiore leggibilità**: È chiaro dove ogni pezzo di stato viene utilizzato.
- **Performance migliorata**: Riduci gli aggiornamenti globali inutili.
- **Facilità di debugging**: Sai esattamente dove cercare se qualcosa va storto.

Usare strumenti come React Query o Zustand, accoppiati con una chiara suddivisione delle responsabilità, aiuta a mantenere un'app React più snella e scalabile.

# React query
[[react-query]] è un set di [[hook]] React che permette di fare query, cache, e cambiare dati sul server in maniera flessibile  per supportare molti casi di utilizzo e ottimizzazioni.
[query/docs/src/pages/docs/api.md at v2 · TanStack/query](https://github.com/TanStack/query/blob/v2/docs/src/pages/docs/api.md#usequery) query
[query/docs/src/pages/docs/api.md at v2 · TanStack/query](https://github.com/TanStack/query/blob/v2/docs/src/pages/docs/api.md#usemutation) mutation

Normalmente per dichiarare uno state cache ne dichiariamo prima la query utilizzando [[useQuery]]:

```ts
function useBook(bookId, user) {
  const {data} = useQuery({
    queryKey: ['book', {bookId}],
    queryFn: () =>
      client(`books/${bookId}`, {token: user.token}).then(data => data.book),
  })
  return data ?? loadingBook
}

function useListItems(user) {
  const {data: listItems} = useQuery({
    queryKey: 'list-items',
    queryFn: () =>
      client(`list-items`, {token: user.token}).then(data => data.listItems),
  })
  return listItems ?? []
}
```

## useQuery
[[useQuery]] viene utilizzato per gestire la lettura dei dati dal server, e la gestione della cache.
La [[qureyKey]] viene utilizzata come chiave per la cache, ogni volta che i dati per quella chiave verranno richiesti, `react-query` controllerà se sono presenti, se lo sono farà un controllo per capire se sono validi, se non sono presenti o non sono validi, lancerà la query per ottenerli dal server, aggiornando la cache, in caso contrario li restituirà.

La `queryKey` può essere una stringa o una lista (i valori al suo interno verranno usati per creare una chiave univoca per l'elemento, usata ad esempio per dare un nome alla query di un elemento specifico, es. un book)

## useMutation
[[useMutation]] è un hook che semplifica la gestione delle mutazioni dei dati, ovvero operazioni che modificano lo stato del server (ad esempio POST, PUT, DELETE). Funziona in modo diverso rispetto a `useQuery`, che viene utilizzato per leggere dati dal server.

Quando esegui una mutazione con `useMutation`:
- Non cerca dati nella cache né aggiorna automaticamente i dati come `useQuery`.
- Serve per eseguire un'azione sul server (es. invio di un nuovo elemento a un database con un'API REST).

```ts
function useUpdateListItem(user, options) {
  return useMutation(
    updates =>
      client(`list-items/${updates.id}`, {
        method: 'PUT',
        data: updates,
        token: user.token,
    }),
}
```

`useMutation` ritornerà non solo la funzione per effettuare la mutazione, ma anche degli helper per capire lo stato della mutazione (isLoading, isError).

```ts
const [
  mutate,
  { status, isIdle, isLoading, isSuccess, isError, data, error, reset },
] = useMutation(mutationFn, {
  onMutate,
  onSuccess,
  onError,
  onSettled,
  throwOnError,
  useErrorBoundary,
})
```

Inoltre oltre alla funzione da lanciare per la mutazione, è possibile anche definire dei comportamenti (es. throwOnError, useErroBoundary) o delle azioni al trigger di determinati eventi (onError, onSettled ecc.)

```ts
// Normalmente utilizzato per essere sicuri che al cambiamento di un elemento della cache, un altro (es. list che lo comprende) venga aggiornato
{
  onSettled: () => queryCache.invalidateQueries('list-items'),
}
```

## [[ReactQueryConfigProvider]]
React query in caso ci servisse standardizzare certi comportamenti di `react-query` in ogni suo utilizzo, mette a disposizione un componente Provider `ReactQueryConfigProvider` che si occuperà di fare il provide della nostra config

```ts
// Possiamo definire comportamenti sia per le query che per le mutation
// Qui decidiamo come vogliamo che funzionino i retry e gestione degli errori
const queryConfig = {
  queries: {
    useErrorBoundary: true,
    refetchOnWindowFocus: false,
    retry(failureCount, error) {
      if (error.status === 404) return false
      else if (failureCount < 2) return true
      else return false
    },
  },
}

export const rootRef = {}
loadDevTools(() => {
  const root = createRoot(document.getElementById('root'))
  root.render(
    <ReactQueryConfigProvider config={queryConfig}>
      <App />
    </ReactQueryConfigProvider>,
  )
  rootRef.current = root
})
```

## [[prefetchQuery]]
Ci sono casi in cui è necessario aggiornare dei dati al leave di una pagina, in quel caso non è necessario usare `useQuery` ma potrebbe essere più appropriato fare un remove della query precedente e fare un prefetch al leave.
In questa maniera sono sicuro che al ritorno in quella pagina (che ad esempio mostra una lista di oggetti), lo stale fra i vecchi dati mostrati e la loro sostituzione con quelli aggiornati, non crei cambiamenti improvvisi di quello che vedo.

### Differenza con useQuery
#### 1. [[useQuery]]
- **Scopo**: È un hook React utilizzato per gestire il fetching, il caching, il re-fetching e lo stato del caricamento dei dati.
- **Funzionamento**: Viene eseguito automaticamente **quando il componente si monta** o quando cambiano i suoi `queryKey`.
- **Ritorno**: Fornisce lo stato della query (`data`, `error`, `isLoading`, ecc.), permettendoti di integrarlo direttamente nell'interfaccia utente.
#### 2. [[prefetchQuery]]
- **Scopo**: Serve per caricare o aggiornare i dati nella cache **prima** che siano richiesti da un componente. Non restituisce lo stato della query; invece, riempie la cache in modo che i dati siano già pronti quando vengono richiesti con `useQuery`.
- **Funzionamento**: Deve essere chiamato manualmente e **non è legato al ciclo di vita di un componente**.
- **Ritorno**: Restituisce una `Promise` che risolve quando il prefetching è completo.
### Perché usare `prefetchQuery` invece di `useQuery`?

1. **Caricamento dei dati fuori dal ciclo di rendering:**
    - Con `prefetchQuery`, puoi caricare i dati in anticipo, anche prima che un componente venga montato.
    - Esempio: immagina una navigazione tra pagine. Se prefetchi i dati della prossima pagina, la transizione sarà istantanea perché i dati saranno già nella cache.
1. **Separazione delle responsabilità:**
    - `prefetchQuery` può essere chiamato ovunque, ad esempio in una funzione che si occupa di preparare i dati (come in un `useEffect` globale o in un listener di eventi).
    - `useQuery`, invece, è legato al ciclo di vita di un componente.
2. **Evita rendering non necessari:**
    - `useQuery` forza un rendering del componente mentre attende i dati (mostrando ad esempio uno stato di caricamento).
    - Con `prefetchQuery`, i dati saranno già pronti nella cache quando il componente si monta.

### Per farla breve..
- Usa **`useQuery`** se i dati devono essere recuperati in modo reattivo, direttamente dal ciclo di vita del componente.
- Usa **`prefetchQuery`** per preparare i dati in anticipo, migliorando l'esperienza utente in scenari come la navigazione o azioni future.

## [[optimistic UI]] and recovery
Spesso il nostro applicativo compie azioni asincrone il cui risultato porta ad un cambiamento nella UI (es. clicco su un pulsante che scatena una chiamata, al success il pulsante cambia di colore).
Nel 90% dei casi, non dovremmo avere errori, per cui possiamo ottimisticamente dire che il pulsante dovrà cambiare di aspetto o che dovrà comparire un campo extra. Per questo potremmo decidere di adottare questa strategia:
1. Compio comunque il side-effect come se l'azione fosse andata a buon fine
2. Dovessero esserci dei problemi riporto la situazione allo stato iniziale (prima del side-effect)
Questo approccio si chiama **optimistic UI**.

React query ci permette di farlo dandoci la possibilità di intervenire in determinati eventi della mutation (onError, onMutate, onSuccess ecc.).

```ts
const defaultMutationOptions = {
  onError: (err, variables, recover) =>
    typeof recover === 'function' ? recover() : null,
  onSettled: () => queryCache.invalidateQueries('list-items'),
}

  

function useUpdateListItem(user, ...options) {
  return useMutation(
    updates =>
      client(`list-items/${updates.id}`, {
        method: 'PUT',
        data: updates,
        token: user.token,
      }),
    {
      onMutate(newItem) {
        const previousItems = queryCache.getQueryData('list-items')
  
        // This optimistically updates the list
        queryCache.setQueryData('list-items', old => {
          return old.map(item => {
            return item.id === newItem.id ? {...item, ...newItem} : item
          })
        })

        // this is returned so that it can be used by onError and onSettled in case of errors
        return () => queryCache.setQueryData('list-items', previousItems)
      },
      ...defaultMutationOptions,
      ...options,
    },
  )
}
```

>Questa tecnica può essere usata anche a prescindere da [react-query](react-query). Ad es. se abbiamo un metodo che prima fa il set di uno stato e poi lo invia al server, avremo che se la chiamata server fallisce, comunque lo stato sarà stato cambiato.
>Possiamo quindi fare nel `catch` della chiamata un `setState(oldState)` senza usare la `lambda` in modo da avere lo state pre-rirendering, e quindi resettare.

