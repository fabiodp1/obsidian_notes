In React gli [[hook]] vengono utilizzati per la gestione dello state, i più comuni sono:
- [[useState]]
- [[useEffect]]
- [[useContext]]
- [[useRef]]
- [[useReducer]]
Tutti questi possono essere utilizzati all'interno di un componente React per storare dati (useState) o compiere azioni (side-effects).

Ognuno di questi hook possiede una propria [[API]], alcune ritornano un valore (come useRef e useContext), altre coppie di valori (useState, useReducer) e altri non ritornano niente (useEffect).

# [[useState]]

E' una funzione che accetta un parametro come initial state e ritorna una coppia di valori, ritornando una array con 2 valori (utilizzato con la destrutturazione per dare un nome ai due valori), lo stato e una funzione setter per cambiare lo stato (per convenzione viene nominata iniziando con 'set').

## lazy state initialization

 useState può prendere come valore iniziale sia il valore direttamente, o una funzione che lo restituisca. La differenza è che passando una funzione, questa verrà chiamata solo al primo mount del componente, mentre nel primo caso il valore verrà estrapolato ogni volta che la funzione del componente verrà chiamata (ad es. al ri-render).
 Passando una funzione ci assicuriamo, nel caso in cui il nostro valore sia il risultato di un'azione, che la stessa azione non venga ripetuta ad ogni render, creando rallentamenti.

```typescript
// In questa maniera verrà fatta la lettura dello store ad ogni render
React.useState(window.localStorage.getItem('name') ?? '')

// Così verrà chiamato solo al primo render/mount
React.useState(() => window.localStorage.getItem('name') ?? '')
```

>Se dobbiamo cambiare uno stato in base ad uno precedente, non va utilizzato direttamente il valore dello stato, ma passare una callback:

```jsx
//...

const [state, setState] = useState(false);

function handleStateChange() {
	setState(!state); // WRONG
	setState(oldState => !oldState)
}

```

>Se lo state da cambiare è una reference (oggetto o array), non va modificato direttamente ma va fatta prima una copia.

```jsx
// WRONG
function handleChange(){
	setNewState(oldState => {
		oldState.name = "Pippo";
		return oldState;
	})
}

// RIGHT
function handleChange(){
	setNewState(oldState => {
		const newState = {...oldState};
		newState.name = "Pippo";
		return newState;
	})
}
```

>Come regola di base, usiamo [[useState]] se la variabile da usare e modificare ha un effetti nella UI (es. viene mostrata nella UI) e quindi c'è bisogno che la sua modifica scateni una riesecuzione del componente. In caso contrario meglio pensare di usare altri metodi, ad es. [[useRef]].

>È inoltre importante capire prima di creare un nuovo stato, se può essere #derived o #computed da uno stato esistente, è importante mantenere il minimo lo state per evitare re-rendering inutili.
# [[useEffect]]

Permette di fare delle azioni dopo che React ha renderizzato (e ri-renderizzato) il componente nel [[DOM]] (quindi da considerare che viene chiamata ogni volta).
Accetta una funzione di callback che React chiamerà DOPO l'update del DOM:

```javascript
React.useEffect(() => {
  // your side-effect code here.
  // this is where you can make HTTP requests or interact with browser APIs.
})
```

![[Pasted image 20241112161806.png]]

## effect dependencies
Ovviamente non sempre vogliamo che il nostro useEffect venga chiamato ad ogni re-render, ad esempio se viene triggerato dal component parent e quindi non c'è un reale cambio nello state, l'useEffect viene chiamato, magari rifacendo inutili chiamate http o salvando nel localStore ecc.
Per evitare questo useEffect prende un altro parametro opzionale chiamato [[dapendency array]], che farà in modo che useEffect chiami la callback solo nel caso in cui uno degli elementi nell'array cambi effettivamente di valore.

```typescript
React.useEffect(() => {
    window.localStorage.setItem('name', name)
  }, [name]) // In questa maniera al re-render lo useEffect chiamerà la callback solo se name ha cambiato valore
```

## lifting state

Molte volte capita che vi sia la necessità di condividere lo state fra componenti fratelli, la risposta è semplice "[[lifting state]]" cioè spostare la gestione dello state nel parent più vicino che si occuperà di passarlo ai figli assieme al meccanismo per aggiornarlo.

## colocating state

Come è importante fare il [[lifting state]], è anche importante ricordarsi di fare il [[colocate state]] cioè ricordarsi che la gestione dello [[state]] dovrebbe stare dove è necessario. Ad esempio se per comabio di feature non è più necessario che tutto lo state venga gestito dal padre, lo state di competenza dovrebbe ritornare al child. In questa maniera si miglioreranno le performance.

>[[managed state]]: state che va esplicitamente gestito
>[[derived state]]: state che può essere calcolato sulla base di un altro state
>[Don't Sync State. Derive It!](https://kentcdodds.com/blog/dont-sync-state-derive-it)

>[[React]] [[bad practice]]: in React non è bene cambiare direttamente lo state per ad es. fare un update, può creare diversi problemi. E' buona pratica fare una copia e usare quella per l'update:

```JSX
const [squares, setSquares] = React.useState(Array(9).fill(null))
  
const squaresCopy = [...squares]
squaresCopy[2] = "newValue"
setSquares(squaresCopy)
```

## utilizzo con [[useRef]]

Spesso capita lavorando con react di dover accedere ai nodi del DOM per manipolarlo, per far ciò utilizziamo useRef:

```javascript
function MyDiv() {
  const myDivRef = React.useRef()
  React.useEffect(() => {
    const myDiv = myDivRef.current
    // myDiv is the div DOM node!
    console.log(myDiv)
  }, [])
  return <div ref={myDivRef}>hi</div>
}
```

Dopo che il componente viene renderizzato, viene considerato [[mounted]], è il momento in cui la callback di [[useEffect]] viene chiamata e a quel punto la prop current della nostra ref dovrebbe essere già valorizzata con il nodo del DOM.
Per questo normalmente la manipolazione del [[DOM]] avviene all'interno della callback dell'useEffect.

>NOTA: In React, passare un array vuoto come dipendenze a `useEffect` significa che l'effetto verrà eseguito solo una volta, al primo render del componente, e non si riattiverà mai più.
>Se ometti l'array delle dipendenze, l'effetto verrà eseguito ad ogni render del componente, poiché React non ha indicazioni su quando dovrebbe limitare l'esecuzione dell'effetto. Questo comportamento può portare a esecuzioni inutili dell'effetto, causando potenzialmente problemi di prestazioni o comportamenti indesiderati, specialmente se l'effetto contiene operazioni pesanti o che interagiscono con l'esterno (come chiamate API).

## HTTP Request

Come abbiamo visto, in useEffect faremo operazioni che vanno svolte subito dopo il mount del componente e che devono ripetersi ad ogni re-render, ad ogni modifica delle dipendenze o una sola volta. Per questo motivo in genere le chiamate HTTP vengono fatte all'nterno di questo [[hook]].
Il problema però è che useEffect può solo ritornare una funzione per il cleanup, e facendo un await, automaticamente verrà rotornata una Promise, a prescindere se venga ritornata qualcosa o meno:

```javascript
// this does not work, don't do this:
React.useEffect(async () => {
  const result = await doSomeAsyncThing()
  // do something with the result
})
```

Il miglior modo per farlo è questo:

```javascript
React.useEffect(() => {
  async function effect() {
    const result = await doSomeAsyncThing()
    // do something with the result
  }
  effect()
})
```

Oppure un altro metodo è estrarre la funzione async e poi richiamarla usando il then:

```javascript
React.useEffect(() => {
  doSomeAsyncThing().then(result => {
    // do something with the result
  })
})
```

## Error component

E' possibile gestire gli errori anche attraverso dei componenti che fanno il wrap di quelli che lanciano l'[[errore]], in modo da poter gestire gli errori in maniera più flessibile.
Per far ciò si utilizza l'[[ErrorBoundary]] component. Ovviamente il componente che viene wrappato dovrà far il thorow dell'errore in modo che venga propagato al boundary.
[ErrorBoudary Component – React](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)

```typescript
function ErrorMessage({error}) {
  return (
    <div role="alert">
      There was an error:{' '}
      <pre style={{whiteSpace: 'normal'}}>{error.message}</pre>
    </div>
  )
}

class ErrorBoundary extends React.Component {
  state = {error: null}
  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return {error}
  }

  render() {
    const {error} = this.state
    if (error) {
      // You can render any custom fallback UI
      return <this.props.FallbackComponent error={error} />
    }
    return this.props.children
  }
}

function App() {
  return (
        <ErrorBoundary FallbackComponent={ErrorMessage}>
          <ThrowingComponent />
        </ErrorBoundary>
  )
}
```

>Normalmente viene utilizzata la libreria [[react-error-boundary]] che già mette a disposizione tutte le proprietà e funzioni per gestire al meglio gli errori.
>[bvaughn/react-error-boundary: Simple reusable React error boundary component](https://github.com/bvaughn/react-error-boundary)

# [[useReducer]]
Grazie a `useState` abbiamo già un ottimo strumento per gestire lo stato dell'applicativo, ma ci sono casi in cui abbiamo bisogno di separare la logica dello [[state]] dai componenti che fanno i cambi di stato. Inoltre se abbiamo diversi elementi di stato che normalmente cambiano assieme, avere un unico oggetto che li contiene può essere molto comodo.

In questi casi è molto utile `useReducer`:


```javascript
function nameReducer(previousName, newName) {
  return newName
}

const initialNameValue = 'Joe'

function NameInput() {
  const [name, setName] = React.useReducer(nameReducer, initialNameValue)
  const handleChange = event => setName(event.target.value)
  return (
    <>
      <label>
        Name: <input defaultValue={name} onChange={handleChange} />
      </label>
      <div>You typed: {name}</div>
    </>
  )
}
```

Dall'esempio si può notare che il riduttore (chiamto nameReducer) viene chiamato con 2 argomenti:
1. Lo stato corrente
2. qualsiasi valore con cui la funzione "setName" viene chiamata. In genere chiamato "action".
[How to implement useState with useReducer](https://kentcdodds.com/blog/how-to-implement-usestate-with-usereducer)
https://kentcdodds.com/blog/should-i-usestate-or-usereducer

```typescript
function myReducer(currentState, action) {
  return action
}
```

## action
La action può essere anche un oggetto..

```typescript
function myReducer(currentState, action) {
  return {...currentState, ...action}
}
```

O una funzione, o entrambe:

```typescript
function myReducer(state, action) {
  return {...state, ...(typeof action === 'function' ? action(state) : action)}
}
```

>Anche in questo caso lo `state` non deve essere modificato direttamente, ad es. facendo un push, ma va ricreato un nuovo oggetto utilizzando lo state precedente.

>Normalmente `useReducer` viene usato per gestire state più complessi, ad esempio liste che hanno bisogno di `add` o `remove`, per questo in genere l'`action` possiede diverse proprietà, ad es. `type` che definiscono l'azione da compiere sullo state, (es. `{ type: 'add', item: {...} }`). In questa maniere all'interno del reducer si possono scatenare diverse logiche a seconda dell'action `type`.
## lazy initialization
useReduce possiede un terzo parametro che viene utilizzato normalmente per inizializzare lo [[state]] applicativo, azione chiamata [[lazy initialization]].

```javascript
function init(initialStateFromProps) {
  return {
    pokemon: null,
    loading: false,
    error: null,
  }
}

// ...

const [state, dispatch] = React.useReducer(reducer, props.initialState, init)
```

Verrà passato il secondo argomento alla funzione di init come parametro e la funzione ritorna l'initial state.
Questo può risultare utile se la funzione init ad es. fa azioni come leggere dal localStorage o comunque qualcosa che non vogliamo venga lanciato ad ogni re-render.

# useCallback / useMemo

[[memoization]]: tecnica di optimizzazione delle performace che elimina il bisogno di ricalcolare un valore per un dato input, salvando il calcolo originale e ritornandolo nel momento in cui viene utilizzato lo stesso input. E' una forma di [[caching]]:

```typescript
const values = {}
function addOne(num: number) {
  if (values[num] === undefined) {
    values[num] = num + 1 // <-- here's the computation
  }
  return values[num]
}
```

Un altro aspetto della memoization è il "value referential equality":

```typescript
const dog1 = new Dog('sam')
const dog2 = new Dog('sam')
console.log(dog1 === dog2) // false
```

Utilizzando la memoization possiamo avere:

```typescript
const dogs = {}
function getDog(name: string) {
  if (dogs[name] === undefined) {
    dogs[name] = new Dog(name)
  }
  return dogs[name]
}

const dog1 = getDog('sam')
const dog2 = getDog('sam')
console.log(dog1 === dog2) // true
```

Un'astrazione generica del momoize può essere:

```typescript
function memoize<ArgType, ReturnValue>(cb: (arg: ArgType) => ReturnValue) {
  const cache: Record<ArgType, ReturnValue> = {}
  return function memoized(arg: ArgType) {
    if (cache[arg] === undefined) {
      cache[arg] = cb(arg)
    }
    return cache[arg]
  }
}

const addOne = memoize((num: number) => num + 1)
const getDog = memoize((name: string) => new Dog(name))
```

In React non serve implementare questo tipo di astrazione, ne abbiamo già a disposizione 2 tipi [[useMemo]] e [[useCallback]].
[Memoization and React | Epic React by Kent C. Dodds](https://www.epicreact.dev/memoization-and-react)

Consideriamo la dependency list di useEffect, se non gliela forniamo, per evitare che il side effect della cb rimanga fuori sync con il resto dello stato dell'applicativo, questo rilancerà la cb ad ogni render.

ma cosa succede se utilizzo una funzione nella mia cb:

```javascript
const updateLocalStorage = () => window.localStorage.setItem('count', count)
React.useEffect(() => {
  updateLocalStorage()
}, []) // <-- what goes in that dependency list?
```

potremmo mettere count nella lista e tutto funzionerebbe. Ma se un giorno qualcuno cambiasse `updateLocalStorage` rendendo la key per lo store dinamica? Dovremmo ricordarci di mettere anche `key` nella lista delle dipendenze, e questo ad ogni modifica della funzione.
Sarebbe molto più facile mettere fra le dipendenze la funzione stessa:

```javascript
const updateLocalStorage = () => window.localStorage.setItem('count', count)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage]) // <-- function as a dependency
```

Il problema con questa scelta è che la funzione viene definita all'interno del body del componente e quindi ricreata ad ogni render e quindi triggerando ogni volta lo `useEffect` .
Questo è il problema che risolve `useCallback`:

```javascript
const updateLocalStorage = React.useCallback(
  () => window.localStorage.setItem('count', count),
  [count], // <-- yup! That's a dependency list!
)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage])
```

Quello che succede è che noi passiamo a React la nostra funzione e lui si occupa di metterla in cache e restituircela. Al prossimo render, se gli elementi passati nella dependency list di `useCallback` non sono cambiati, react ci restituirà la stessa funzione cachata in precedenza (valorizzando quindi la variabile con la stessa reference alla funzione), evitando che venga triggerata di nuovo la cb dello `useEffect`.

[[useCallback]] è solo una scorciatoia per utilizzare [[useMemo]] con le funzioni:
[When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

```typescript
// the useMemo version:
const updateLocalStorage = React.useMemo(
  // useCallback saves us from this annoying double-arrow function thing:
  () => () => window.localStorage.setItem('count', count),
  [count],
)

// the useCallback version
const updateLocalStorage = React.useCallback(
  () => window.localStorage.setItem('count', count),
  [count],
)
```

>Normalmente `useCallback` e `useMemo` vengono utilizzati per fare il memoize di cb e dati per queste due situazioni:

- per le array delle dipendenze, in modo da evitare di triggerare `useEffect` inutilmente.
- per le prop su componenti di cui è stato fatto il momoize (in questo caso `useMemo`)

E' possibile usarli anche in altri casi, ma questi sono i più comuni

# useContext
[[5 - Context]]

Per condividere lo [[state]] fra componenti è un problema comune, possiamo innalzare lo stato nella catena padre/figlio ([[lifting state]]), ma poi ci troveremmo a dover fare un fastidioso [[prop drilling]]  per poter condividerlo con tutti i figli e figli dei figli.

Per evitare questo problema, possiamo inserire dello stato in una sezione del tree di React e poi estrarlo dove serve, senza doverlo esplicitamente passare. Questa feature viene chiamata context, è come una sorta di global variable, ma senza i problemi delle global.

```javascript
const FooContext = React.createContext()

function FooDisplay() {
  const foo = React.useContext(FooContext)
  return <div>Foo is: {foo}</div>
}

ReactDOM.render(
  <FooContext.Provider value="I am foo">
    <FooDisplay />
  </FooContext.Provider>,
  document.getElementById('root'),
)
// renders <div>Foo is: I am foo</div>
```

FooDisplay potrebbe trovarsi da qualsiasi parte della [[render tree]] e avrà comunque accesso al valore passato dal componente FooContext.Provider.

>NOTA: è possibile passare un argomento al `createContext` che verrà usato come valore di default nel caso in cui il contesto venisse utilizzato tramite `useContext` senza la presenza di un provider. Ma questo è SCONSIGLIATO, perchè non è raccomandato usare un contesto al di fuori di un provider.

## context provider component

Per questo in genere viene creato un componente apposito che si occupa di fare il provide. Inoltre è buona norma creare un custom hook che si occupi di verificare se il componente figlio si trova all'interno di un provider e in caso contrario lanci un eccezione. In questa maniera evitiamo che ad es. un componente non mostri un valore semplicemente perchè non avendo un provider, allo `useContext` riceverà `undefined`.

```javascript
const CountContext = React.createContext()

function useCount() {
  const countContext = React.useContext(CountContext)
  if (countContext === undefined) {
    throw new Error(
      'useCount may only be used from within a child of a CountProvider',
    )
  }
  return countContext
}

// Importante passare anche props al component provider
function CountProvider(props) {

  const [count, setCount] = React.useState(0)
  const value = [count, setCount]
  
  return <CountContext.Provider value={value} {...props} />
}

function CountDisplay() {
  const [count] = useCount()
  return <div>{'The current count is ${count}'}</div>
}

function Counter() {
  const [, setCount] = useCount()
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>Increment count</button>
}

function App() {
  return (
    <div>
      <CountProvider>
        <CountDisplay />
        <Counter />
      </CountProvider>
    </div>
  )
}

export default App
```

>Con [[React 19+]] è possibile fare il wrap senza bisogno di aggiungere `.Provider`:

```tsx
<CartContext>
	<App />
</CartContext>
```

>Inoltre con [[React 19+]] è possibile usare l'hook [[use]] invece di [[useContext]].
# useLayoutEffect

Ci sono 2 modi per dire a React di lanciare side-effects dopo il render:
3. [[useEffect]]
4. [[useLayoutEffect]]
La differenza fra questi è abbastanza subdola (hanno la stessa API).
Nel 99% dei casi verrà usato `useEffect`, ma in certi casi `useLayoutEffect` può migliorare l'esperienza utente.
[useEffect vs useLayoutEffect](https://kentcdodds.com/blog/useeffect-vs-uselayouteffect)

La semplice rule-of-thumb è:
>Se stai facendo dei cambiamenti chiaramente osservabili nel DOM, usa `useLayoutEffect`, negli altri casi solo `useEffect`.

# useImperativeHandle

Permette di decidere quello che viene esposto quando un componente genitore utilizza un `ref` su un componente figlio.

## Quando

- si usa quando si vuole esporre al genitore solo alcune funzionalità del componente figlio
- si sta lavorando con componenti personalizzati che racchiudono comportamenti complessi legati ad dom (es. modali, input personalizzati, focus ecc.)

```typescript
import React, { useImperativeHandle, useRef, forwardRef } from 'react';

// Componente figlio
const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  // Esponi solo i metodi specifici al genitore
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => (inputRef.current.value = ""),
  }));

  return <input ref={inputRef} {...props} />;
});

// Componente genitore
const Parent = () => {
  const inputRef = useRef();

  return (
    <div>
      <CustomInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
      <button onClick={() => inputRef.current.clear()}>Clear</button>
    </div>
  );
};

export default Parent;

```

In questo caso il componente figlio `CustomInput` usa `useImperativeHandle` per esporre solo i netordi `focus` e `clear`.
Il componente genitore può usare il ref per chiamare questi metodi, senza sapere nulla del nodo DOM interno.

# useDebugValue

Quando si inizia a scriver custom hooks, può risultare molto utile dargli delle sorte di label speciali per identificarli e per magari differenziare i differenti usi che se ne fanno all'interno del componente.

E' qui che `useDebugValue` risulta utile. Viene utilizzato in un [[custom hook]] in questa maniera:

```javascript
function useCount({initialCount = 0, step = 1} = {}) {
  React.useDebugValue({initialCount, step})
  const [count, setCount] = React.useState(initialCount)
  const increment = () => setCount(c => c + step)
  return [count, increment]
}
```

Adesso quando viene utilizzato `useCount` sarà possibile vedere i valori di `initialCount` e `step` nei React DevToolds.

Volendo prende anche un secondo parametro come formatter:

```javascript
const formatCountDebugValue = ({initialCount, step}) =>
  `init: ${initialCount}; step: ${step}`

function useCount({initialCount = 0, step = 1} = {}) {
  React.useDebugValue({initialCount, step}, formatCountDebugValue)
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + step)
  return [count, increment]
}
```

# useImperativeHandle

[[useImperativeHandle]] permette ad un componente di esporre una propria API chiamabile dal componente padre. Ci sono infatti certi casi in cui è utilie, ad esempio quando il padre tramite ref interagisce con degli elementi del componente figlio, elementi che potrebbero cambiare senza che il padre ne sia consapevole. Ad es. il padre potrebbe utilizzare il metodo showModal di un elemento figlio [[dialog]], ma questo in un futuro viene cambiato in div.

```tsx
// Child
export default function ResultModal({ result, targetTime, timeLeft, ref }) {
  const dialogRef = useRef();

  useImperativeHandle(ref, () => ({
    open() {
      dialogRef.current.showModal();
    },
  }));

  return (
    <dialog className="result-modal" ref={dialogRef}>
      <h2>You {result}</h2>
    //...

//Father

//...

function handleStopTimer() {
    resultModal.current.open();
}

return (
    <>
      <ResultModal
        ref={resultModal}
      />      
```

In questa maniera il padre può rimanere cieco alla reale implementazione del figlio, e sarà quest'ultimo a doverne gestire la logica.

# Custom hooks

E' possibile riutilizzare codice che potrebbe essere utile altrove in funzioni che vengono chiamate dai componenti che ne hanno di bisogno, proprio come normali funzioni JS e ovviamente ritornano la stessa dupletta di un normale hook.
Queste funzioni vengono chiamate "custom hooks".

>Non è necessario ritornare una dupletta, posso anche ritornare un oggetto contenente più stati es. `isLoading`, `data`, `error` ecc.
>In questa maniera saranno disponibili per i componenti che potranno farne il `distructuring`.

```jsx
function useLocalStorageState(key, initialState = '') {
  const [state, setState] = React.useState(
    () => window.localStorage.getItem(key) ?? initialState,
  )

  React.useEffect(() => {
    window.localStorage.setItem(key, state)
  }, [key, state])

  return [state, setState]
}

function Greeting({initialName = ''}) {
  const [name, setName] = useLocalStorageState('name', initialName)
  
  function handleChange(event) {
    setName(event.target.value)
  }

  return (
    <div>
      <form>
        <label htmlFor="name">Name: </label>
        <input value={name} onChange={handleChange} id="name" />
      </form>
      {name ? <strong>Hello {name}</strong> : 'Please type your name'}
    </div>
  )
}

function App() {
  return <Greeting initialName="Fabio" />
}
```

>Anche i `custom hook` devono avere un nome che inizia con `use..` in modo che venga riconosciuto da React come hook.

