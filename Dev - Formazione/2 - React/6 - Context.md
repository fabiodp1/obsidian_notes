Una volta che abbiamo tutto il nostro server state cache in [React Query](5%20-%20Cache%20management%20-%20React%20Query.md), non rimane molto altro `state` rimasto in giro per l'applicazione che non possa essere gestito tramite composition e lifting state.

Comunque in alcuni casi abbiamo la necessità di avere certi UI state accessibili globalmente via context. Ad esempio se l'utente è loggato, o per le notifiche toast ecc.

[useContext – React](https://react.dev/reference/react/useContext)

# useContext

Il componente derivante da `useContext`, oltre a fornire la proprietà `.Provider`, fornisce anche la proprietà [[Consumer]], in questa maniera è possibile wrappare il componente che deve consumare il context. Questo wrapper deve avere come child una funzione avente come parametro il contrext, e ritornerà il [[JSX]] del componente:

```jsx
//...

return (
	<CartContext.Consumer>
		{(cartCtx) => {
			// ...Some logic that uses the context
			const item = cartCtx.items[0];

			return (
				<div>{ item }</div>
			)
		}}
	</CartContext.Consumer>
)
```

Questo approccio non è quello preferibile perché rende il codice meno leggibile e un po antiquato.

# Re-rendering

Nel momento in cui un componente utilizza il context e questo viene aggiornato, il componente viene ri-eseguito come se stesse aggiornando il proprio state.

# Context provider component

In genere viene creato un componente apposito che si occupa di fare il provide. Inoltre è buona norma creare un custom hook che si occupi di verificare se il componente figlio si trova all'interno di un provider e in caso contrario lanci un eccezione. In questa maniera evitiamo che ad es. un componente non mostri un valore semplicemente perchè non avendo un provider, allo `useContext` riceverà `undefined`.

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