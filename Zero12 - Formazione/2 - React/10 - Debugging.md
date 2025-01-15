## StrictMode
[[StrictMode]] è un componente [[React]] speciale che è pensato per fare il wrap della parte di applicativo che vogliamo controllare (può essere tutta l'app o solo un componente specifico):

```jsx
ReactDOM.createRoot(document.getElementById('root')).render(
	<StrictMode>
		<App />
	</StrictMode>
);
```

