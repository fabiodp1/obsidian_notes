[Epic React V1 Pro | Epic React by Kent C. Dodds](https://www.epicreact.dev/products/epic-react-pro-v1)

# [[React]]
E' un [[declarative language]]
Supporta diverse piattaforme (native e web), ognuna di queste possiede il suo corrispettivo codice per interaggire con quella piattaforma, ed esiste anche codice condiviso fra le piattaforme.

Per scrivere applicazioni React per il web sono necessari 2 file [[JS]]:
- React: responsabile per la creazione degli elementi React (simile a document.createElement())
- ReactDOM: responsabile per la renderizzazione degli elementi React nel DOM (simile a rootElement.append())
```typescript
// Esempio Row di utilizzo
const rootElement = document.getElementById('root')
    const elProps = {children: 'Hello World', className: 'container'}
    const elType = 'div'
    // Corrisponde a document.createElemntent()
    const el = React.createElement(elType, elProps)

	// Queste 2 righe corrispondono a rootElement.append() 
    const root = ReactDOM.createRoot(rootElement)
    root.render(el)
```

## [[JSX]]
https://react.dev/learn/writing-markup-with-jsx

[[Fragment]]: corrisponde a "<></>" e viene utilizzato per wrappare il markup che verrà restituito da un componente React nel momento in cui non si vuole utilizzare un div che lo faccia, perchè non si vuole creare un div inutilmente.

```jsx
<>  
<h1>Hedy Lamarr's Todos</h1>  
<img  
src="https://i.imgur.com/yXOvdOSs.jpg"  
alt="Hedy Lamarr"  
class="photo">  
<ul>  

...  

</ul>  
</>
```

[[Interpolation]]: vengono utilizzate singole {}

```JSX
export default function Avatar() {
  const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';
  const description = 'Gregorio Y. Zara';
  return (
    <img
      className="avatar"
      src={avatar}
      alt={description}
    />
  );
}
```

Anche questo è ammissibile:

```JSX
function message(props) {
      return <div class="message">{props.children}</div>
    }
    const element = (
      <div className="container">
        {React.createElement(message, {children: 'Hello World'})}
        {React.createElement(message, {children: 'Goodbye World'})}
      </div>
    )

    ReactDOM.createRoot(document.getElementById('root')).render(element)
```
## [[Components]]

In React i components sono semplicemente delle funzioni JS che ritornano del contenuto renderizzabile (altri elementi React, stringhe, numeri, null ecc.).

## Styling

Esistono due modi per applicare lo styling ai componenti, tramite il normale attributo "style" o tramite l'attributo "calssName".

In [[JSX]] per passare lo style inline, viene fatta la normale interpolazione passando un oggetto avente come proprietà le classi [[CSS]].

```JSX
<div style={{marginTop: 20, backgroundColor: 'blue'}} />
```

Tenere in considerazione che i nomi dell proprità CSS non sono in [[kebab-case]] ma [[CamelCase]].
Un esempio di styling con interpolazione:

```JSX
function Box({size, style, ...otherProps}) {
  const className = size ? `box--${size}` : ''
  return (
    <div
      className={`${className} box`}
      style={{fontStyle: 'italic', ...style}}
      {...otherProps}
    />
  )
}

function App() {
  return (
    <div>
      <Box size="small" style={{backgroundColor: 'lightblue'}}>
        small lightblue box
      </Box>
      <Box size="medium" style={{backgroundColor: 'pink'}}>
        medium pink box
      </Box>
      <Box size="large" style={{backgroundColor: 'orange'}}>
        medium porange box
      </Box>
    </div>
  )
}
```

## [[Form]]

Non ci sono molti modi in React per poter interaggire con le Form, possiamo aggiungere un handler che verrà triggerato al submit [[onSubmit]] questo verrà chiamato avendo come parametro l'evento, contenetne la proprietà target che rappresenta il nodo DOM della form.
Tramite questo è possibile accedere ai songoli elementi della form e ai loro valori.

Esistono diversi modi per accedere ai valori delle form:
- tramite index: event.target.elements[0].value
- tramite id o name: event.target.elements.usernameInput.value

Un esempio di gestione della form:

```JSX
function UsernameForm({onSubmitUsername}) {

  function handleSubmit(event) {
    event.preventDefault()
    const username = event.target.usernameInput.value
    onSubmitUsername(username)
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="usernameInput">Username:</label>
        <input type="text" name="usernameInput" />
      </div>
      <button type="submit">Submit</button>
    </form>
  )
}

function App() {
  const onSubmitUsername = username => alert(`You entered: ${username}`)
  return <UsernameForm onSubmitUsername={onSubmitUsername} />
}

export default App
```

### Ref
Un altro modo per accedere agli elementi di un DOM è tramite l'attributo [[ref]].

>la proprità [[ref]] è un oggetto che rimane consistente fra i render del componente React e possiede una proprietà *current* che può essere aggiornata in qualsiasi momento con qualsiasi valore.
>Se stiamo interagendo con i nodi del [[DOM]] possiamo passare un ref ad un elemento React and React imposterà come valore del current il nodo DOM che viene renderizzato.

Se creiamo un oggetto [[inputRef]] tramite *React.useRef* possiamo accedere al suo valore con *inputRef.current.value*.

>**NOTA**: In React ref non funziona come in Vue, è un semplicemente un oggetto, ma la sua modifica non scatena il re-render del componente.

Pricipali differenze con [[Vue]]:

| Caratteristica            | Vue `ref`                         | React `useRef`                      |
| ------------------------- | --------------------------------- | ----------------------------------- |
| Sintassi                  | `const nome = ref(initialValue)`  | `const nome = useRef(initialValue)` |
| Reattività                | Reattivo (triggera aggiornamenti) | Non causa render automatici         |
| Accesso al valore         | .value                            | .current                            |
| Uso tipico                | Dati reattivi e DOM refs          | Valori persistenti e DOM refs       |
| Modifica DOM nei template | ref="nomeRef"                     | ref={nomeRef}                       |

### State
In react per accedere allo state si utilizza un "[[hook]]" chiamato [[useState]].

```jsx
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(count + 1)
  return <button onClick={increment}>{count}</button>
}
```

React.useState accetta un valore di default e ritorna un array. Normalmente, come si vede nell'esempio, si fa il destructuring dell'array per avere lo state e la funzione per farne l'update.

```JSX
function UsernameForm({onSubmitUsername}) {
  const usernameInput = React.useRef('fabio')
  const [error, setError] = React.useState(null)

  function handleSubmit(event) {
    event.preventDefault()
    if (usernameInput.current.value) {
      onSubmitUsername(usernameInput.current.value)
    }
  }

  function handleChange(event) {
    const value = usernameInput.current.value
    const isValid = value === value.toLowerCase()

    setError(isValid ? null : 'Username must be lower case')
  }

  

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="usernameInput">Username:</label>
        <input
          type="text"
          name="usernameInput"
          onChange={handleChange}
          ref={usernameInput}
        />
      </div>
      <div style={{color: 'red'}}>{error}</div>
      <button type="submit" disabled={error !== null}>
        Submit
      </button>
    </form>
  )
}
```

Negli esempi precedenti semplicemente abbiamo lasciato che il browser avesse il controllo del valore del campo input, noi semplicemente lo richiediamo nel momento in cui veniamo notificati del suo cambiamento, per poter applicare la nostra logica.
Ma ci sono casi in cui vogliamo avere il controllo sul suo valore. React ci permette di farlo tramite il set della prop *value*:

```jsx
<input value={myInputValue} />
```

Una volta fatto ciò Reat si assicura che il valore dell'input non può mai essere differente dal valore della variabile passata.

Normalmente viene anche utilizzato un handler per l'[[onChange]] in modo da poter avere la consapevolezza dei "cambiamenti suggeriti" al campo di input.
Normalmente si fa lo store del valore del campo input in una variabile state (con React.useState) e poi l'onChange handler