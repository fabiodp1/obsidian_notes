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
 Passando una funzione ci assicuriamo, nel caso in cui il nostro valore sia il risultato di un'azione, che la stessa azione venga ripetuta ad ogni render, creando rallentamenti.

```typescript
// In questa maniera verrà fatta la lettura dello store ad ogni render
React.useState(window.localStorage.getItem('name') ?? '')

// Così verrà chiamato solo al primo render/mount
React.useState(() => window.localStorage.getItem('name') ?? '')
```
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

## [[custom hook]]
E' possibile riutilizzare codice che potrebbe essere utile altrove in funzioni che vengono chiamate dai componenti che ne hanno di bisogno, proprio come normali funzioni JS e ovviamente ritornano la stessa dupletta di un normale hook.
Queste funzioni vengono chiamate "custom hooks".

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

## lifting state
Molte volte capita che vi sia la necessità di condividere lo state fra componenti fratelli, la risposta è semplice "lifting [[state]]" cioè spostare la gestione dello state nel parent più vicino che si occuperà di passarlo ai figli assieme al meccanismo per aggiornarlo.