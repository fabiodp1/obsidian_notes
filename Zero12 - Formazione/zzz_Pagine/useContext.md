Il componente derivante da [[useContext]], oltre a fornire la proprietà `.Provider`, fornisce anche la proprietà [[Consumer]], in questa maniera è possibile wrappare il componente che deve consumare il context. Questo wrapper deve avere come child una funzione avente come parametro il contrext, e ritornerà il [[JSX]] del componente:

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
