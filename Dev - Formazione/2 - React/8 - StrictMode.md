## StrictMode
[[StrictMode]] è un componente [[React]] speciale che è pensato per fare il wrap della parte di applicativo che vogliamo controllare (può essere tutta l'app o solo un componente specifico):

```jsx
ReactDOM.createRoot(document.getElementById('root')).render(
	<StrictMode>
		<App />
	</StrictMode>
);
```

>[[StrictMode]] ha effetto solo in dev mode.

Questo componente fa diverse cose per aiutarci a individuare dei bug, ad es:
- Esegue (ri-renderizza) i componenti 2 volte di fila (può portare all'individuazione di certi bug)