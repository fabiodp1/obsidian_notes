[Your React App Isn’t Great — Here’s Why. | by Pierre G. | Dec, 2024 | Level Up Coding](https://levelup.gitconnected.com/your-react-app-isnt-great-here-s-why-5eb61b3f110b)

# Code splitting
[Code-Splitting – React](https://legacy.reactjs.org/docs/code-splitting.html)

Per ottimizzare le performance è importante che fra componenti le callback vengano passate utilizzando l'hook `useCallback`, in modo da evitare re-render inutili.
Inoltre se il nostro componente è un context provider, è buona pratica passare come value la versione memoized:

```ts
function AuthProvider(props) {
  const {
    data: user,
    error,
    ...
  } = useAsync()

  const login = React.useCallback(
    form => auth.login(form).then(user => setData(user)),
    [setData],
  )

  const register = React.useCallback(
    form => auth.register(form).then(user => setData(user)),
    [setData],
  )

  const logout = React.useCallback(() => {
    auth.logout()
    setData(null)
  }, [setData])

  const value = React.useMemo(() => ({user, login, logout, register}), [
    login,
    logout,
    register,
    user,
  ])

  if (isLoading || isIdle) {
    return <FullPageSpinner />
  }

  if (isError) {
    return <FullPageErrorFallback error={error} />
  }

  if (isSuccess) {
    return <AuthContext.Provider value={value} {...props} />
  }

  throw new Error(`Unhandled status: ${status}`)
}
```

---

# Compound components

I [[compound components]] sono un modo in [[React]] per organizzare i componenti in modo da renderli flessibili e componibili fra loro.
Non sono pensati per essere `standalone` ma per funzionare uniti fra loro.

Ad es. se dovessimo creare un componente `Accordion`, normalmente potremmo crearne uno che contiene un `ul` e accetta una prop `items` su cui lui fa il `map` per mostrarne gli elementi.
Ma utilizzando il pattern del `compound component` possiamo fare in modo che questo sia più elastico e riutilizzabile:

```tsx title:Accordion.tsx
export default function Accordion({children, className }) {
  return <ul className={className}>{children}</ul>
}
```

In questa maniera diventa un guscio personalizzabile a cui basta passare gli elementi che vogliamo.
Questo però non basta, non è l'idea di questo pattern, creeremo altri `compound components` che con `Accordion` concorreranno a strutturare il nostro componente finale:

```tsx title:AccordionItem.tsx
export default function AccordionItem({className, title, children}) {
  return (
    <li className={className}>
      <h3>{ title }</h3>
      <div>{ children }</div>
    </li>
  )
}
```

In questa maniera possiamo comporre i nostri componenti così:

```tsx title:App.tsx
function App() {
  return(
    ...
    <Accordion className="accordion">
      <AccordionItem title="One">
        <article>
          <p>One</p>
        </article>
      </AccordionItem>
      <AccordionItem title="Two">
        <article>
          <p>Two</p>
        </article>
      </AccordionItem>
    </Accordion>
  )
}
```

## Gestire lo stato fra componenti

Poiché stiamo lavorando con diversi componenti che devono rimanere fortemente configurabili, non possiamo aggiungere lo stato in uno solo di questi (ad. es. quello principale), perderebbero la loro indipendenza.

Per farlo possiamo usare il [Context](6%20-%20Context.md) di React.:

```tsx title:Accordion.tsx
import {createContext} from 'react';

// Generalmente viene creato nel file del componente principale
const AccordionContext = createContext();

export function useAccordionContext() {
  const ctx = useContext(AccordionContext);

  if(!ctx) {
    throw new Error("Accordion-related components must be wrapped by <Accordion>.");
  }
}

export default function Accordion({ children, className }) {
  const [openItemId, setOpenItemId] = useState();

  function toggleItem(id) {
    setOpenItemId(prevId => prevId === id ? null : id);
  }

  const contextValue = {
    openItemId,
    toggleItem
  };

  return (
    <AccordionContext.Provider value={ contextValue }>
      <ul className={ className }>{ children }</ul>
    </AccordionContext.Provider>
  )
}
```

```tsx title:AccordionItem.tsx
export default function AccordionItem({id, className, title, children}) {
  const { openItemId, toggleItem } = useAccordionContext();

  const isOpen = openItemId === id;

  function handleClick() {
    toggleItem(id);
  }

  return (
    <li className={className}>
      <h3 onClick={handleClick}>{ title }</h3>
      <div className={ isOpen ? 'open' : undefined }>
        { children }
      </div>
    </li>
  )
}
```

## Raggruppare i compound componentes

È buona norma, per rendere più leggibile e ovvio che dei componenti fanno parte dello stesso gruppo, mettere i componenti in uno stesso oggetto, che solitamente sarà il componente principale (nell'es. `Accordion`).

>In [JS](JS) le funzioni non sono altro che oggetti, e i componenti [[React]] non sono altro che *funzioni*, quindi possiamo accedere o creare nuove prop.

```tsx title:Accordion.tsx
import {createContext} from 'react';
import {AccordionItem} from './AccordionItem.tsx'

// Generalmente viene creato nel file del componente principale
const AccordionContext = createContext();

export function useAccordionContext() {
  ...
}

export default function Accordion({ children, className }) {
  ...
  return (
    <AccordionContext.Provider value={ contextValue }>
      <ul className={ className }>{ children }</ul>
    </AccordionContext.Provider>
  )
}

Accordion.Item = AccordionItem;

```

In questa maniera possiamo usare il nostro `AccordionItem` in questa maniera:

```tsx title:App.tsx
// import AccordionItem from './AccordionItem.tsx' <== Non serve più
import Accordion from './Accordion.tsx';

function App() {
  return(
    ...
    <Accordion className="accordion">
      <Accordion.Item title="One">        <===
        <article>
          <p>One</p>
        </article>
      </Accordion.Item>
      <Accordion.Item title="Two">
        <article>
          <p>Two</p>
        </article>
      </Accordion.Item>
    </Accordion>
  )
}
```

Adesso è chiaro che il componente `Item` appartiene ad `Accordion`.
Ovviamente partendo da queste basi si potrebbe anche pensare di spezzare i componenti visti negli esempi in altri `compound components`, ad es. per il content dell'`Item`, il titolo ecc.

>Se abbiamo bisogno di condividere dello state solo con una parte del gruppo (ad es. sottocomponenti di `Item`), possiamo creare un context nel componente che deve condividere lo state, wrappando i suoi figli:

```tsx title:AccordionItem.tsx
const AccordionItemContext = createContext();

export function useAccordionItemContext() {\
  const ctx = useContext(AccordionItemContext);

  if(!ctx) {
    throw new Error("AccordionItem-related components should be wrapped by <Accordion.Item>")
  };

  return ctx;
}

export default function AccordionItem({id, className, title, children}) {
  ...
  return (
    <AccordionItemContext.Provider value={id}>
      <li className={className}>
        { children }
      </li>
    </AccordionItemContext.Provider>
  );
}
```

---

# Render Props

L'idea alla base delle `Render Props` è quello di passare una funzione come valore per la prop `children` (in realtà è possibile farlo per qualsiasi prop, ma in genere è usato per `children`).

>L'idea è quella di avere un componente che definisce una funzione che **ritorna del contenuto renderizzabile**, ed è questa funzione che verrà passata come valore per la prop `children` (quindi ad un secondo componente). Questo secondo componente ritornerà il contenuto restituito dalla chiamata a quella func.

Ad es. possiamo volere un componente che andrebbe riutilizzato, ma si occupa di definire della logica, non tanto come il risultato di questa logica deve essere mostrato, e dovrà poter essere utilizzato anche per diversi tipi di dato:

Pensiamo di avere un componente come questo:

```tsx title:SearchableList.tsx
export default function SearchableList({items}) {
  return (
    <div>
      <input type="search" placeholder="Search" />
      <ul>
        {items.map(item => ...)}  <!-- <=== lo cambieremo dopo  -->
      </ul>
    </div>
  )
}
```

Vorremmo poterlo riutilizzare con diversi tipi di dato:

```tsx title:App.tsx
const PLACES = [
  {id: "1", name: "Place 1"},
  {id: "2", name: "Place 2"}
]

function App() {
  return (
    ...
    <SearchableList items={PLACES} />
    <SearchableList items={[ 'item 1', 'item 2' ]} />
    ...
  )
}
```

Se nel nostro componente di ricerca aggiungiamo della logica per il filtraggio, vedremo che se possiamo in qualche modo risolvere il problema logico legato alla differenza di type, per il rendering non avremo una soluzione adatta:

```tsx title:SearchableList.tsx
export default function SearchableList({items}) {
  const [searchTerm, setSearchTerm] = useState('');

  const searchResults = items.filter(item => 
    JSON.stringify(item).toLowerCase().includes(searchTerm.toLowerCase())
  );

  function handleChange(event) {
    setSearchTerm(event.target.value);
  }

  return (
    <div>
      <input type="search" placeholder="Search" onChange={handleChange} />
      <ul>
        {items.map((item, index) => (
          <!-- Gli oggetti saranno [Object][Object] -->
          <li key={index}>{item.toString()}</li>
        ))}
      </ul>
    </div>
  )
}
```

In questo ci viene in aiuto il pattern delle `Render Props`, dove utilizzeremo la prop `children` non come faremmo normalmente, ma come se fosse una funzione:

```tsx title:SearchableList.tsx
export default function SearchableList({items, children}) {
  ...
  return (
    <div>
      <input type="search" placeholder="Search" onChange={handleChange} />
      <ul>
        {items.map((item, index) => (
          <li key={index}>
            { children(item) }                    <===
          </li>
        ))}
      </ul>
    </div>
  )
}
```

>Utilizzandola nella maniera standard, l'[HTML](HTML.md) renderizzato sarà uguale per tutti gli item della lista, mentre noi vorremmo che fosse diverso per ogni elemento.
>Chiamandolo come funzione, facciamo in modo che il [JSX](JSX.md) da renderizzare sia specifico per quell'item. Ovviamente, come dicevamo, la chiamata dovrà ritornare del codice renderizzabile.

Quindi per utilizzarlo in questa maniera dovremo passare come child una funzione:

```tsx title:App.tsx
const PLACES = [
  {id: "1", name: "Place 1"},
  {id: "2", name: "Place 2"}
]

function App() {
  return (
    ...
    <SearchableList items={PLACES} >
    <!-- Un componente che si aspetta quel tipo di oggetto -->
      { item => <Place item={item} /> }                     <===
	</SearchableList>
    <SearchableList items={[ 'item 1', 'item 2' ]} >
      { item => item }                                      <===
	</SearchableList>
    ...
  )
}
```

>In questa maniera stiamo riutilizzando il componente `SearchableList` in maniera molto più versatile ed elastica.

## Gestire le key dinamicamente

Nell'esempio sopra, stiamo utilizzando come `key` per il nostro `map` la index dell'item, ma come sappiamo non è corretto, può portare a risultati inaspettati nel comportamento della nostra lista.
Il problema è però che i nostri item possono avere tipi diversi, quindi non possiamo fare affidamento sulla prop `id`.

Possiamo però passare al nostro componente che gestisce la lista una funzione che si occuperà di generare l'id dinamicamente, un po come abbiamo fatto per il `children`:

```tsx title:SearchableList.tsx
export default function SearchableList({items, children, itemKeyFn}) {
  ...
  return (
    <div>
      <input type="search" placeholder="Search" onChange={handleChange} />
      <ul>
        {items.map((item, index) => (
          <li key={ itemKeyFn(item) }>          <===
            { children(item) }
          </li>
        ))}
      </ul>
    </div>
  )
}
```

Adesso possiamo gestire come preferiamo la generazione dell'id a seconda del tipo passato.

```tsx title:App.tsx
function App() {
  return (
    ...
    <SearchableList items={PLACES} 
      itemKeyFn={ item => item.id }>                  <===
      { item => <Place item={item} /> }
	</SearchableList>
    <SearchableList items={[ 'item 1', 'item 2' ]} 
      itemKeyFn={ item => item }>                     <===
      { item => item }
	</SearchableList>
    ...
  )
}
```

---

# Debouncing

Il `debouncing` è un pattern che permette di fare in modo che una stessa azione possa essere eseguita solo dopo un determinato periodo di tempo, evitando la sua continua ri-esecuzione al compiere di una determinata azione.

Un es. può essere quello di un campo di ricerca, se la funzione che si occupa di fare il fetch dei dati filtrati venisse chiamata ad ogni keystroke, rischieremmo di avere inutili e numerose chiamate server. Con il `debouncing` possiamo fare in modo che la chiamata venga fatta solo dopo che l'utente ha smesso di scrivere per un toto periodo di tempo.