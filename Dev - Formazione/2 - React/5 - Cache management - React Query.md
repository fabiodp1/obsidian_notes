# Premessa sulla gestione dello state

La gestione della [[cache]] è sicuramente uno dei problemi più grossi nello sviluppo applicativo.
Lo state può essere raggruppato in due contenitori:

1. [[UI state]]: isModalOpen, isItemHighlighted, etc.
2. [[Server cache]]: User data, tweets, contacts, etc.

La principale causa di complessità e problemi viene quando si cerca di mischiare questi due contenitori. Quando ciò avviene, lo UI state che non dovrebbe essere global viene reso tale poichè il Server cache state è tipicamente global, così naturalmente ci porta a rendere tutto global.

## UI State e Server Cache State

### UI State

- Riguarda lo **stato locale** e temporaneo dell'interfaccia utente.
- Esempi:
    - Una modale aperta/chiusa.
    - Un elemento evidenziato.
    - Una barra laterale visibile.
- Deve essere locale a un componente o una parte specifica dell'app.

### Server Cache State

- Riguarda i **dati recuperati da un server** e solitamente condivisi tra più componenti.
- Esempi:
    - Informazioni sull'utente.
    - Elenchi di post, messaggi o notifiche.
- Di solito è **globale**, perché i dati devono essere accessibili in vari punti dell'app.

## Il Problema: Confondere i Due Stati

Se si unisce lo stato dell'interfaccia utente con lo stato della cache del server (ad esempio mettendoli entrambi in uno stato globale come Redux o Context API), emergono i seguenti problemi:

1. **Overhead inutile**:
    - Lo **UI state** di un componente potrebbe attivare aggiornamenti non necessari in altri componenti che usano lo stato globale, creando rallentamenti.
2. **Difficoltà di debugging**:
    - È più difficile capire quale parte del codice aggiorna un particolare pezzo di stato, specialmente se questo è condiviso globalmente.
3. **Architettura rigida**:
	- Lo **UI state**, che è locale per natura, diventa più difficile da gestire in maniera isolata.
1. **Scalabilità compromessa**:
    - Aggiungere nuove funzionalità o stati diventa complicato, perché tutto è intrecciato in un unico store.

## La Soluzione: Separare gli Stati

Un approccio migliore è distinguere chiaramente i due tipi di stato:
### Gestione del UI State

- Utilizza **stato locale** o **gestori locali**:
    - `useState`, `useReducer`.
    - Soluzioni come **Zustand** o altre librerie leggere se serve qualcosa di più sofisticato.

### Gestione del Server Cache State

- Usa librerie ottimizzate per cache e sincronizzazione con il server:
    - **React Query** (TanStack Query): Gestisce la cache, i re-fetch e la sincronizzazione in modo automatico.
    - Apollo Client (per GraphQL).

## Esempio Pratico

### Scenario:

Vuoi mostrare una lista di utenti e, quando clicchi su un utente, aprire una modale con i dettagli.
### Cosa NON Fare:

Mettere tutto lo stato in Redux/Context:

```ts
const globalState = {
  isModalOpen: false, // UI state
  selectedUserId: null, // UI state
  users: [] // Server cache state
};
```

Questo rende la gestione dello stato inefficiente e il codice meno leggibile.

### Cosa Fare:

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

## Conclusione

Separando lo **UI state** e lo **server cache state**, ottieni:

- **Maggiore leggibilità**: È chiaro dove ogni pezzo di stato viene utilizzato.
- **Performance migliorata**: Riduci gli aggiornamenti globali inutili.
- **Facilità di debugging**: Sai esattamente dove cercare se qualcosa va storto.

Usare strumenti come React Query o Zustand, accoppiati con una chiara suddivisione delle responsabilità, aiuta a mantenere un'app React più snella e scalabile.

---

# WHAT

[[react-query]] è un set di [[hook]] React che permette di fare query, cache, e cambiare dati sul server in maniera flessibile per supportare molti casi di utilizzo e ottimizzazioni.

>Aiuta nell'invio di richieste HTTP e a tenere la [UI](UI) in sync, semplificando di molto il codice e l'esperienza di sviluppo.

---

# WHY

È anche possibile continuare ad usare i normali HTTP client per fare il fetch dei dati e utilizzare lo state di React, magari creando dei custom hook per il retrieve dei dati da poter riutilizzare, ma comunque avremmo delle limitazioni e mancheremmo di qualche feature.

Per esempio una delle feature che potrebbe essere comoda, è di poter fare il re-fetch dei dati nel momento in cui ci spostiamo su un altro tab del browser per poi ritornare sulla nostra app, in modo da aggiornare i dati a sincronizzare la [[UI]].

Un'altra feature che vorremmo è di poter fare la cache dei dati fetchati, in modo da poterli tenere in memoria e riutilizzarli senza dover fare un altro fetch nel momento in cui ci spostiamo di pagina per poi ritornare.

>È possibile implementare queste feature senza `Tanstack Query`, ma dovremmo scrivere un bel po di codice e sarebbe un po tricky e prone ad errori.

---

# Fetch data

Una volta installato, possiamo creare i nostri metodi che si occuperanno del fetch dei dati, ma invece di usare `HTTP client` come `axios` o `fetch`, utilizzeremo gli [hook](hook) messi a disposizione da [Tanstack](Tanstack%20Query.md).

Normalmente per dichiarare uno `state cache` ne dichiariamo prima la query utilizzando `useQuery`, questa si occuperà di fare il fetch dei dati che ci servono nel componente, e di darci anche informazioni riguardo lo `state` della chiamata.

Per configurarla sarà necessario definire:

- `queryFn`: la funzione che farà veramente la richiesta al server.

>`React Query` non invia di per se richieste HTTP, saremo noi a doverne configurare la logica.
>Poi `React Query` si occuperà di gestire il dato, gli errori, il caching e molto altro.

- `queryKey`: usando `useQuery`, ogni query dovrà avere una key che `React Query` utilizzerà internamente per fare la `cache` del dato ricevuto dalla risposta. La key è un `Array` di valori che possono avere diversi `type`.

`useQuery` ritornerà un oggetto contenente diverse proprietà fra cui:

- `data`: il dato ottenuto dal server
- `isPending`: per avere lo stato della chiamata
- `isError`: per controllare se la chiamata è fallita
- `error`: per ottenere l'oggetto dell'errore
- `refetch`: per rilanciare la stessa query (per es. al click di un button)

```ts
import { useQuery } from '@tanstack/react-query';

export default function NewEvents() {
	const { data, isPending, isError, error, refetch } = useQuery({
		queryKey: ['events'],
		queryFn: fetchEvents
	});

	let content;

	if(isPending) {
		content = <LoadingIndicator />;
	}

	if(isError) {
		content = (
			<ErrorBlock title="An error occurred" message={error.info?.message || 'Failed to fetch events.'} />
		)
	}
}

//...
export async function fetchEvents() {
	const response = await fetch(...);

	if(!response.ok) {
		const error = new Error('...');
		error.code = response.status;
		error.info = await response.json();
		// Perchè Tanstack gestisca l'errore va rilanciato
		throw error;
	}

	const { events } = await response.json();

	return events;
}
```

>Utilizzare `React Query` in questa maniera non basta, darebbe errore perché per poterlo utilizzare bisogna fare il wrap dei componenti con il provider `QueryClientProvider` e il `QueryClient`:

```tsx
import { QueryClientProvider } from '@tanstack/react-query';

// Prima di tutto va instanziato il client
const queryClient = new QueryClient();

function App() {
	return (
		<QueryClientProvider client={queryClient}>
			<RouterProvider router={router} />
		</QueryClientProvider>
	)
}
```

## Config Query Behaviors - Cache & Stale Data

Una delle più grandi feature di `React Query`, è la possibilità di fare `caching` client-side. Infatti ad ogni chiamata server fa il `cache` della risposta. In questa maniera se cambiamo la pagina dell'applicativo e poi ritorniamo, se la cache non è stata invalidata rivedremo gli stessi dati immediatamente, senza bisogno di fare un'altra chiamata server, così migliorando di molto la [[UX]].

>Le immagini non vengono gestite da `React Query` o da React, vengono comunque richieste al server o potenzialmente sarà il browser a farne il `cache`.

Dietro le quinte **in realtà `React Query` fa comunque la chiamata**, ma solo per controllare se i dati in cache sono ancora validi o meno. Nel mentre è già stato restituito il dato della cache.
Ovviamente come sviluppatori vogliamo avere il controllo su questo tipo di comportamenti, per questo React Query mette a disposizione prop di configurazione come `staleTime` con cui possiamo indicare dopo quanto tempo deve essere fatta la chiamata di controllo nel momento in cui il dato è presente in cache, in modo da evitare chiamate non necessarie:

```tsx
//...
const {data, isPending...} = useQuery({
	queryKey: ['events'],
	queryFn: fetchEvents,
	staleTime: 5000 // default 0               <==
});
```

Un'altra proprietà per configurare i comportamenti della query è `gcTime` (garbage collection time), che definisce quanto vanno mantenuti i dati nella cache, passato quel tempo la cache verrà invalidata e al mount del componente verrà rifatta la chiamata:

```tsx
//...
const {data, isPending...} = useQuery({
	queryKey: ['events'],
	queryFn: fetchEvents,
	gcTime: 30000 // default 5min               <==
});
```

## Query dinamiche e Query Keys

Consideriamo il caso di voler filtrare gli eventi mostrati nella pagina tramite un comunissimo campo di ricerca. In questo caso dovremo modificare il nostro metodo per il fetch in modo che accetti dei query parameter da inviare al server per avere i dati filtrati:

```ts
export async function fetchEvents(searchTerm) {
	let url = 'http://.../events';

	if(searchTerms) {
		url += '?search=' + searchTerm;
	}

	const response = await fetch(url);

	//...
	return events;
}
```

Nel nostro componente che si occuperà di lanciare la query, utilizzeremo com `queryFn` una funzione che lancia la richiesta con il parametro di ricerca.

>Se impostassimo come `queryKey` solo "events", riutilizzeremmo la cache del componente che mostra tutti gli eventi, quindi quella query ritornerebbe l'ultima lista di eventi presente in cache e quindi bypassando il filtraggio.
>Per questo dovremo costruire la nuova `queryKey` usando anche il valore passato, in modo che rimangano distinte e ogni ricerca unica.
>In questa maniera React Query potrà fare la `cache` e riutilizzare dati differenti per chiavi differenti, basate sulla stessa query.

Il secondo parametro può essere costruito come vogliamo, può anche solo essere il valore della ricerca, e quindi una stringa.

Potremmo pensare quindi di configurare la query così, ma è sbagliato:

```tsx
export default function FindEventSection() {
  const searchElement = useRef();

  const {data, ...} = useQuery({
    queryKey: ["events", { search: searchElement.current.value }], //<== WRONG
	queryFn: () => fetchEvents(searchElement.current.value)
  }) 

  return (
    <form>
      <input ... ref={ searchElement }/>
	  <button>Search</button>
	</form>
  )
}
```

>Questo perché con questo approccio un cambiamento al parametro di ricerca non scatenerebbe il re-rendering del componente, e quindi una nuova query col nuovo parametro, bisognerà usare un normale `state`:

```tsx
export default function FindEventSection() {
	const searchElement = useRef();
	const [ searchTerm, setSearchTerm ] = useState('');

	const {data, ...} = useQuery({
		queryKey: ["events", { search: searchTerm }],
		queryFn: () => fetchEvents(searchTerm)
	});

	const handleSubmit = (event) => {
		event.preventDefault();
		setSearchTerm(searchElement.current.value)
	}

	return (
		<form onSubmit="handleSubmit">
			<input ... ref={ searchElement }/>
			<button>Search</button>
		</form>
	)
}
```

## Query Config Obj & Aborting Requests

`React Query` di default passa come argomento alla nostra `queryFn` un oggetto contenente info sulla `queryKey` usata, e una proprietà chiamata `signal`, questa viene usata per fare l'abort della chiamata, ad es. nel caso in cui dovessimo cambiare pagina prima che la chiamata venga conclusa.

Possiamo utilizzare questa proprietà programmaticamente utilizzando l'oggetto che viene passato da `React Query` alla nostra `queryFn`, per poi passarlo al metodo che si occupa della fetch, in modo che lo possa utilizzare per fare l'abort.

Per gestire i valori che vogliamo passare alla nostra `queryFn`, possiamo utilizzare l'oggetto della config per aggiungere la proprietà che vogliamo:

```tsx
// The name of the added prop (searchTerms) can be anything we want
export async function fetchEvents({ signal, searchTerm }) {
	let url = 'http://.../events';

	if(searchTerm) {
		url += '?search=' + searchTerm;
	}

	// Now it can be used by the browser
	const response = await fetch(url, { signal });
	//...

// COMPONENT
export default function FindEventSection() {
	const searchElement = useRef();
	const [ searchTerm, setSearchTerm ] = useState('');

	const {data, ...} = useQuery({
		queryKey: ["events", { search: searchTerm }],
		// We pass the object that contains the signal forwarded and our prop
		// Obviously this is needed just because we need to pass a custom prop
		// or we could have just passed the fetchEvents func
		queryFn: ({ signal }) => fetchEvents({ searchTerm, signal })
	});
```

## Abilitare e Disabilitare le query

Ci sono casi in cui non vorremmo che certe query venissero fatte automaticamente se non a determinate condizioni. Ad es. potremmo avere una query per avere una lista di oggetti e una query per la stessa lista ma filtrata (come visto sopra), essendo 2 query diverse, non possono essere gestite centralmente, quindi se una pagina al mount mostra una zona con la lista generale e una con la lista filtrata, le rispettive query verrebbero lanciate ed essendo che al mount non è ancora stato inserito un valore di ricerca, il risultato sarebbe due liste identiche. 
Per questo vorremmo *disabilitare* la query di filtraggio fino a quando non viene inserito un termine di ricerca.

>Potremmo gestirla all'interno della nostra `queryFn`, ma esiste un modo più efficiente, utilizzando la proprietà di *useQuery* `enabled`.

```tsx
export default function FindEventSection() {
	const searchElement = useRef();
	const [ searchTerm, setSearchTerm ] = useState('');

	const {data, ...} = useQuery({
		queryKey: ["events", { search: searchTerm }],
		queryFn: ({ signal }) => fetchEvents({ searchTerm, signal }),
		enabled: searchTerm !== '';                                   // <===
	});
	
//...
```

>C'è però un problema con il codice scritto sopra. Infatti noi non vorremmo che la query venisse disabilitata **OGNI** volta che il searchTerm è vuoto, ma solo all'inizio. Questo perché se no ogni volta che l'utente cancella il termine di ricerca e lancia la ricerca, la nostra lista scomparirà non venendo fatto il fetch, e non è quello che si vorrebbe, è preferibile che vengano invece mostrati tutti gli elementi.

Per questo è consigliabile inizializzare il campo di ricerca ad `undefined`, in questa maniera la query sarà disabilitata solo al `mount` e non quando viene valorizzata con una stringa vuota:

```tsx
export default function FindEventSection() {
	const searchElement = useRef();
	const [ searchTerm, setSearchTerm ] = useState();                // <===

	const {data, ...} = useQuery({
		queryKey: ["events", { search: searchTerm }],
		queryFn: ({ signal }) => fetchEvents({ searchTerm, signal });
		enabled: searchTerm !== undefined;                           // <===
	});
	
//...
```

>Di default `React Query` tratta una query disabilitata come `isPending => true`, perché rimane in attesa che venga abilitata. Questo può creare problemi se abbiamo della [[UI]] che risponde a quella proprietà.

Per questo in alternativa a `isPending` ci viene messa a disposizione la prop `isLoading`. La differenza è che `isLoading` non sarà `true` quando la query è disabilitata.

>Per gestire della [UI](UI) che mostra uno stato di caricamento, es. il `loading spinner`, è preferibile usare la prop di `React Query` `isLoading`:

```tsx
export default function NewEvents() {
	const { data, isLoading, isError, error, refetch } = useQuery({
		queryKey: ['events'],
		queryFn: fetchEvents
	});
	//...

	return (
		//...
		{ isLoading && <LoadingSpinner /> }
		//...
	)
```

## Utilizzare la queryKey come input della queryFn

Potremmo avere `queryKey` che hanno come secondo elemento un oggetto variabile, e potremmo voler utilizzare questo nella funzione che passiamo alla `queryFn` (es. per gestire un parametro url per il `max` degli item che vogliamo ricevere dal server).
Per fare ciò possiamo utilizzare uno dei parametri dell'oggetto passato automaticamente alla funzione `queryFn` per accedere a questo secondo elemento:

```tsx
//...
const { data, isPending, isError, error } = useQuery({
	queryKey: ['events', { max: 3 }],
	// [1] per avere il secondo elemento dell'array della queryKey
	queryFn: ({ signal, queryKey }) => fetchEvents({ signal, ...queryKey[1]}),
});

```

>In questa maniera è possibile evitare ripetizioni ed è possibile scrivere del codice più flessibile.

---

# Cambiare dati con le Mutation

`React Query` oltre a poter essere usato per ottenere dati, può essere usato anche per inviare e aggiornare dati. Per fare ciò in maniera ottimizzata (volendo sarebbe possibile farlo anche tramite *useQuery*) viene messo a disposizione l'[hook](hook) `useMutation`.

>Lo scopo delle mutazioni non è quello di gestire la `cache` ma solo di mandare richieste al server. Per questo non aggiornerà la `cache` con la risposta ecc.

Come per `useQuery` vuole un oggetto in cui settare la funzione da lanciare, in questo caso alla prop `mutationFn` e ritornerà un oggetto che fra le alte prop conterrà la funzione `mutate` che sarà quella da utilizzare per lanciare la funzione impostata in `mutationFn`. Infatti a differenza di `useQuery`, la funzione settata non viene lanciata in automatico al mount del componente:

```tsx
export default function NewEvent() {
	const { mutate, isPending, isError, error } = useMutation({
		mutationFn: createNewEvent
	});

	const handleSubmit = (formtData) => {
		// Ovviamente la forma del dato dipende dalla funzione e da quello che si aspetta il server
		mutate( { event: formtData } );
	}

	//...

	return (
		<EventForm onSubmit={ handleSubmit } >
			{ isPending && 'Submitting...' }
			{ !isPending && (
				<>
					<Link to="../">Cancel</Link>
					<button type="submit">Create</button>
				</>
			) }
		</EventForm>
		{ isError && <ErrorBlock ... message={ error.info?.message || "..." } /> }
	)
}

//...
export async function createNewEvent(eventData) {
	const response = await fetch('...', {
		method: 'POST',
		//...
	})
}
```

## Gestire una Mutation Success e Invalidare la Query

Normalmente, nel momento in cui inviamo dati al server, vorremmo anche reagire alla risposta ad es. per gestire la [[UI]].
Potremmo gestirlo all'interno del nostro handler:

```tsx
//...
funciton handleSubmit(formData) {
	mutate({ event: formData });
	navigate('/events');                // <== ?
}
```

In questo modo però non verrà aspettata la fine della chiamata lanciata con `mutate`, quindi a prescindere dal successo o fallimento della chiamata, farà comunque un redirect.

Per questo `React Query` ci mette a disposizione altre proprietà configurabili che ci permettono di gestire questi casi, come la prop `onSuccess`:

```tsx
export default function NewEvent() {
	const { mutate, isPending, isError, error } = useMutation({
		mutationFn: createNewEvent,
		onSuccess: () => {
			navigate('/events');
			//...
		},
	});
//...
```

>Sapendo che al success abbiamo modificato i dati lato server, e quindi che la situazione è cambiata rispetto ai dati che abbiamo in pancia lato client, vorremmo anche fare in modo che venga rilanciata la query per il fetch dei dati, in modo da avere quelli aggiornati.
>Per fare ciò basta invalidare la query e quindi la `cache` che se detiene quel dato, in modo da riscatenare il fetch.

Per fare ciò avremo bisogno dell'istanza creata del `QueryClient`, di cui possiamo tranquillamente fare l'`export/import` e del suo metodo `invalidateQueries`, che può essere settato anche per invalidare una specifica `query`:

```tsx
import { queryClient } form '...';

export default function NewEvent() {
	const { mutate, isPending, isError, error } = useMutation({
		mutationFn: createNewEvent,
		onSuccess: () => {
			// La queryKey deve avere lo stesso formato (type) della dichiarazione
			// Senza 'exact' invaliderebbe tutte le query che si basano su 'events'
			// quindi invaliderebbe anche ['events', {search: searchTerm}]
			queryClient.invalidateQueries({ queryKey: ['events'], exact: true });
			navigate('/events');
		},
	});
//...
```

>Nella fattispecie, in questo esempio in realtà avrebbe senso invalidare anche la query contenente `searchTerm`, poiché se no se il nuovo elemento salvato rientra fra i dati filtrati da mostrare, non comparirebbe perché la relativa query non avrebbe dati aggiornati.

C'è però da considerare il caso in cui ad es. ci troviamo nella pagina di un 'evento' e ne gestiamo il delete tramite `mutation` come abbiamo fatto per il `create`.

>Nel momento in cui invalidiamo le query, queste verranno automaticamente rilanciate.

Nel caso del delete questo comportamento potrebbe però portare ad un side-effect non voluto, infatti trovandoci ancora nella pagina dell'elemento da cancellare, viene rilanciata anche la query per il fetch dell'elemento, risultando in un `404`.

Per questo esiste una proprietà dell'oggetto passato per l'invalidazione della query, `refetchType`, che gestisce il comportamento del `refetch`, ad es. per fare in modo che il `refetch` non venga fatto subito, ma solo quando necessario, ad es. al ritorno sulla pagina degli eventi.

```tsx
import { queryClient } form '...';

export default function NewEvent() {
	const { mutate, isPending, isError, error } = useMutation({
		mutationFn: createNewEvent,
		onSuccess: () => {
			queryClient.invalidateQueries({ 
				queryKey: ['events'], 
				refetchType: 'none'         // <==
			});
			navigate('/events');
		},
	});
//...
```

## Optimistic Update

Ogni volta che aggiorniamo dei dati mostrati a schermo e ne salviamo le modifiche, dobbiamo attendere la risposta del server perché queste modifiche si riflettano sulla [UI](UI), creando una esperienza utente non delle migliori.

>Per questo è consigliabile implementare un `pattern` differente, mostrando le modifiche istantaneamente e nel caso in cui qualcosa dovesse andar storto, dare un feedback all'utente e fare il revert delle modifiche fatte alla [UI](UI). Questo viene chiamato `Optimistic UI`.

Con `React Query` è possibile utilizzare questo `pattern` con semplicità grazie alla proprietà dell'oggetto di config della mutation `onMutate`.

`onMutate` vuole un func che verrà eseguita nel momento in cui `mutate` viene chiamato, quindi prima della chiamata e prima della risposta del server.
Tramite questa funzione possiamo cambiare direttamente il dato nella `cache` di React Query, grazie all'utilizzo dell'istanza che abbiamo a disposizione per interagire con React Query e la sua cache, `QueryClient`, e il suo metodo `setQueryData`.

Questo metodo prenderà 2 parametri, la `queryKey` nella forma in cui la stessa query è stata dichiarata, e i nuovi dati che vogliamo vengano settati, che vengono passati alla nostra `onMutete` func da React Query.

Altra cosa **importante** che va fatta prima di settare il dato in maniera ottimistica, è quella di cancellare tutte le query rimaste in sospeso (per la stessa `queryKey` o in relazione con questa), per evitare che ci siano collisioni fra i dati che ritorneranno da queste e i dati che stiamo settando in maniera ottimistica. 
Questo grazie al metodo `cancelQueries`, e passandogli la chiave della relativa `query`.

>Da precisare che `cancelQueries` cancellerà solo le query derivanti da `useQuery`, non le `mutation`.

```tsx
export default function EditEvent() {
  //...
  const params = useParams();
  const {data, isPending, isError, error} = useQuery({
  	queryKey: ['events', params.id],
	queryFn: ({signal}) => fetchEvent({signal, id: params.id})
  });

  const { mutate } = useMutation({
	mutationFn: updateEvent,
	// React Query passerà i dati che verranno utilizzati per la mutation
	onMutate: async (data) => {
      const newEvent = data.event;

	  // Cancellare tutte le query pending per evitare data collision
	  await queryClient.cancelQueries({queryKey: ['events', params.id]});
	  
	  // Modifico in maniera ottimistica la cache con i nuovi dati
	  queryClient.setQueryData(['events', params.id], newEvent);
	}
  });
}
```

Ovviamente è da gestire anche un possibile `fail` nel caso in cui la chiamata di update dovesse non andare bene, e quindi i dati immessi in maniera ottimistica non sarebbero corretti e fosse necessario un `rollback`.

Per fare ciò dovremmo conservare i dati originali prima di fare l'update, da poter riutilizzare al `rollback`. Per fare ciò possiamo avere i dati originali presenti in `cache` grazie al metodo del queryClient `getQueryData`, ovviamente passandogli la corretta `queryKey`:

```tsx
//...
const { mutate } = useMutation({
  mutationFn: updateEvent,
  onMutate: async (data) => {
	const newEvent = data.event;
		
	await queryClient.cancelQueries({queryKey: ['events', params.id]});
		
	const prevEvent = queryClient.getQueryData(['events', params.id]);  // <==
	
	queryClient.setQueryData(['events', params.id], newEvent);
  }
});
//...
```

A questo punto per poter fare il `rollback` in caso di errore, possiamo usare la proprietà `onError` presente nell'oggetto config della `mutation`, resettando ai dati originali.
A questa funzione vengono passati automaticamente dei parametri:

- `error`: l'errore sollevato.
- `data`: i dati inviati con la `mutation`
- `context`: importante perché fra le altre cose conterrà il dato originale *conservato e ritornato* dalla funzione passata  a `onMutate`

```tsx
//...
const { mutate } = useMutation({
	mutationFn: updateEvent,
	onMutate: async (data) => {
	  //...
	  const prevEvent = queryClient.getQueryData(['events', params.id]);

	  // Importante
	  return { previousEvent: prevEvent };
	},
	onError: (error, data, context) => {
	  // Resettiamo al dato originale usando context e il dato ritornato
	  queryClient.setQueryData(['events', params.id], context.previousEvent)
	}
});
//...
```

>Un'ultima cosa che va fatta quando utilizziamo la [optimistic UI](optimistic%20UI.md), è gestire la fine della mutazione a prescindere dal risultato (un po come il `finally` del try/catch), invalidando le query coinvolte in modo da assicurarsi che vengano aggiornate a prescindere, forzando così il sync fra client e server. 

Possiamo farlo grazie ad un'altra prop della config di `useMutation`, `onSettled`:

```tsx
//...
const { mutate } = useMutation({
	mutationFn: updateEvent,
	onMutate: async (data) => {
	  //...
	  const prevEvent = queryClient.getQueryData(['events', params.id]);

	  // Importante
	  return { previousEvent: prevEvent };
	},
	onError: (error, data, context) => {
	  queryClient.setQueryData(['events', params.id], context.previousEvent)
	},
	onSettled: () => {                                       // <===
		queryClient.invalidateQueries(['events', params.id]);
	}
});
//...
```

---

# React Query & React Router

Nella sezione [4 - Routing](4%20-%20Routing.md) abbiamo visto che di per se `React Router` permette di configurare azioni per fare il fetch e update dei dati. Quindi a questo punto non rimane che mettere assieme i due mondi per farli collaborare in maniera efficiente.

## Loader

per quanto riguarda il loader da passare alla route, possiamo dichiarare la funzione loader nel file del componente come avremmo fatto normalmente per definire il loader di `React Router`.

>Poiché in questo caso non ci troviamo all'interno del componente, non possiamo usare `useQuery`, quindi dobbiamo avere accesso all'istanza del `queryClient` e usare il suo metodo `fetchQuery`.

Questo prenderà come parametri gli stessi utilizzati da `useQuery` ma essendo un loader utilizzato da `React Router`, riceverà anche l'oggetto contenente il contesto della `route`, quindi avremo a disposizione `query params` ecc.:

```tsx
//...
export function loader({params}) {
	return queryClient.fetchQuery(
		queryKey: ['events', params.id],
		queryFn: ({signal}) => fetchEvent({signal, id: params.id})
	);
}
```

Questo potrebbe farci pensare che adesso nel componente non avremo più bisogno di usare  `useQuery` per poter accedere a quei dati, perché potremmo usare l'[[hook]] di React Router `useLoaderData`, e sarebbe anche possibile, ma **NON CONVIENE**. Infatti così facendo perderemmo tutte le feature che ci mette a disposizione `useQuery`, come il controllo dietro le quinte della validità dei dati posseduti al cambio del tab del browser ecc.

>È però importante ricordare che lasciando `useQuery`, al mount del component partirà la chiamata dietro le quinte per controllare che i dati in cache siano aggiornati, chiamata che si sommerà a quella fatta dal `loader`.
>Per questo motivo è importante nella config di `useQuery` aumentare il valore dello `staleTime`, in modo che il fetch venga fatto solo se la cache è più vecchia di un tot, ad es. se cambiamo tab e ritorniamo, controllerà se la cache è più vecchia del valore indicato e solo in quel caso lancerà la chiamata:

```tsx
//...
export function loader({params}) {
	return queryClient.fetchQuery(
		queryKey: ['events', params.id],
		queryFn: ({signal}) => fetchEvent({signal, id: params.id}),
		staleTime: 10000
	);
}
```

Utilizzando il `loader` di React Router, non sarà più necessario usare la prop `isPending` ritornata da `useQuery`, perché i dati saranno caricati e salvati nella `cache` prima che venga servito il componente.
Volendo se utilizziamo la gestione degli errori messa a disposizione da `React Router`, non ci sarebbe bisogno neanche di utilizzare le prorp `error` e `isError`.

>Il loader però potrebbe prendere un po di tempo per concludere il fetch e quindi creare un'[UX](UX) non delle migliori. Per questo possiamo usare l'[hook](hook) fornito da React Query `useIsFetching` che ci darà un valore che indicherà numericamente (0 se non sta facendo il fetch) se cu sibi fetch attivi (in qualsiasi componente della pagina), in modo da poter mostrare un'indicazione all'utente. 

## Action

Come abbiamo fatto per il `leader`, possiamo usare `React Query` e le sue `mutation` anche per definire la `action` della route.
Anche in questo caso la funzione riceverà (come abbiamo visto in [4 - Routing](4%20-%20Routing.md)) un oggetto con informazioni riguardo la richiesta di `submit` che ha scatenato la `action` della route.

```tsx
//...
import { ..., redirect } from 'react-router-dom';
//...

export function action({request, params}) {
	const formData = await request.formData();
// trasforma l'oggetto più complesso 'formData' in un oggetto semplice key/value
	const updatedEventData = Object.fromEntries(formData);
	
	await updateEvent({ id: params.id, event: updatedEventData });

	await queryClient.invalidateQueries(['events']);

	return redirect('../')
}
```

>Possiamo chiamare direttamente la nostra funzione `updateEvent`, senza bisogno di dover usare una `mutation`, inquanto la mutation è semplicemente un wrapper della nostra funzione, che aggiunge funzionalità come ad es. la prop `isPending` ecc. Utilizzando `React Router` non ne abbiamo di bisogno.
>Quello di cui abbiamo bisogno è utilizzare il `queryClient` per invalidare la relativa `query` e scatenare l'aggiornamento del dato e quindi anche della `cache`.

Nel nostro componente possiamo utilizzare il metodo fornito da `useSubmit` per lanciare in maniera programmatica l'evento di submit e quindi scatenare la action della route.

>**NOTA**: con questo metodo non potremmo però utilizzare le feature delle `mutation` per utilizzare la [optimistic UI](optimistic%20UI.md). Possiamo utilizzare la prop `state` dell'oggetto ritornato da `useNavigation` (fornito da React Router) per poter dare un feedback visivo dello stato della chiamata.

```tsx
export default function EditEvent() {
	//...
	const {state} = useNavigation();

	//...
	return (
		//...
		{ state === 'submitting' && <div>Sending data...</div> }
		//...
	)
}
```

---

# Other
## ReactQueryConfigProvider

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

## prefetchQuery

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