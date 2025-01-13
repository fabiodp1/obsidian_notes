## Passare componenti via prop
È possibile passare ad un componente un altro componente da utilizzare o un tag html, ad es. potrebbe essere utile per wrappare del codice in un elemento dinamico:

```jsx
function WrapMyContent({children, container}) {
	const Container = container; // DEVE essere storato in una variabile in modo da diventare PascalCase ed essere riconisciuto come un custom component

	return (
	<Container> {// Adesso è PascalCase e React capirà che si tratta di un custom component
	}
		{children}
	</Container>
	)
}

function App () {
	return (
	<WrapMyContent container="menu"> {// in questo caso viene passato un elemento html built-in, ma tramite {} potrei passare anche custom components
	}
		<h1> Wrapped Child </h1>
	</WrapMyContent>
	)
}
```