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
