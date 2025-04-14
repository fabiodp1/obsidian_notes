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

# [[JSX]]
https://react.dev/learn/writing-markup-with-jsx

[[Fragment]]: corrisponde a `<></>` e viene utilizzato per wrappare il markup che verrà restituito da un componente React nel momento in cui non si vuole utilizzare un div che lo faccia, perchè non si vuole creare un div inutilmente.

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
# Components

In React i components sono semplicemente delle funzioni JS che ritornano del contenuto renderizzabile (altri elementi React, stringhe, numeri, null ecc.).

# Styling

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

>È possibile utilizzare file [[CSS]] semplicemente importandoli, ma c'è da tenere presente che in questo modo non saranno #scoped rispetto al componente, quindi potremmo modificare tutto l'applicativo. 
>Se si vuole scrivere [[CSS]] specifico per per componente ( #scoped ), bisogna utilizzare i [[CSS Modules]], rinominando il file css in es. `Header.module.css` e importando nel componente le singole classi come se fossero delle variabili:

```jsx
import classes from './Header.module.css';
//...
// Il nome della classe deve essere quello dichiarato nel css
<p className={classes.paragraph}> ... </p>
//...
```

# Rendering Arrays

Se con Vue nel creare un v-for loop per renderizzare un [[array]] di elementi è consigliato ma non necessario utilizzare l'attributo #key, in React è OBBLIGATORIO.
Infatti è possibile ometterla ma nel momento in cui il componente deve ri-renderizzarsi perchè è stato aggiunto o rimosso un elemento, React non saprà in che posizione questo è stato inserito, potendo potenzialmente risolversi in un risultato non voluto.

```jsx
const list = ['One', 'Two', 'Three']
const listUI = list.map(listItem => <li>{listItem}</li>)
// notice that listUI is an array
const ui = <ul>{listUI}</ul>
```

Per cui la regola è che la prop #key va SEMPRE messa, e non va utilizzata la index dell'array o si ripropone lo stesso problema di non usare la #key, va utilizzato un valore univoco, come un id.
