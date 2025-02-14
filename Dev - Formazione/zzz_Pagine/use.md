A differenza di [[useContext]] è più flessibile. Infatti anche se si tratta di un [[hook]] e facendo lo stesso di [[useContext]], è possibile utilizzarlo anche in altri modi, ad es. all'interno di un `if statement`:

```tsx
//...
if(true) {
	const cartCtx = use(CartContext);
}

```