# Perché React ri-renderizza
In [[React]], un componente viene ri-renderizzato ogni volta che una sua variabile di **stato** cambia. Questo comportamento si propaga a tutti i suoi figli, anche se non sono direttamente legati alla variabile che ha subito il cambiamento. React adotta questa strategia per garantire che la **UI sia sempre sincronizzata** con lo stato attuale dell’applicazione.

Esempio:

```jsx
<App>
  <Counter count={count} />
  <Decoration />
</App>
```

Se lo stato `count` cambia, `<Counter>` e `<Decoration>` verranno ri-renderizzati. Anche se `<Decoration>` non utilizza `count`, viene comunque incluso nel processo di ri-render per precauzione. React non può sapere con certezza che non esistano dipendenze nascoste.

## Comportamenti Impuri

React si aspetta che i componenti siano **puri**, cioè che producano sempre lo stesso risultato a parità di props. Tuttavia, è facile creare componenti **impuri**, per esempio usando variabili dipendenti da `Date` o mutando direttamente i refs.

React preferisce **ri-renderizzare troppo piuttosto che mostrare una UI obsoleta**. Di conseguenza, molti re-render sono previsti e accettabili.

# Ottimizzare con React.memo

**`React.memo`** è uno strumento che permette di memorizzare il risultato di un componente puro e riutilizzarlo fintanto che le sue props non cambiano. Questo è utile per ridurre i re-render in componenti che non subiscono cambiamenti di stato o props.

Esempio:

```jsx
const Decoration = React.memo(() => {
  return <div>Decorazione</div>;
});
```

Con `React.memo`, React non ri-renderizzerà `<Decoration>` se le props rimangono invariate. È come se React "scattasse una foto" del componente e la riutilizzasse fintanto che non ci sono modifiche.

## Quando usare React.memo

`React.memo` è più utile per:

1. Componenti con molte props o figli.
2. Operazioni interne costose.
3. Situazioni in cui i re-render influenzano negativamente le performance.
4. Va utilizzato il più in alto possibile nel tree dei componenti, per bloccare anche l'esecuzione dei figli

Tuttavia, non è sempre necessario: i re-render leggeri sono spesso meno costosi della logica aggiuntiva per verificarli.

## Quando NON usare React.memo

- Quando il componente da wrappare possiede props che vengono aggiornate spesso
- Per wrappare indiscriminatamente tutti i componenti
- Quando il ri-rendering non creerebbe problemi di performance

>con `memo` React dovrà controllare tutte le prop per vedere se sono cambiate, aggiungendo logica che potrebbe pesare più del semplice ri-rendering del componente.
>In molti casi una buona gestione dello state e della composizione dei componenti (ad es. esternalizzando in un nuovo componente parte di codice che modifica lo stato) può essere il metodo migliore per evitare inutili ri-rendering

# Memoizzare dati e funzioni con useMemo e useCallback

## `useMemo`

`useMemo` è usato per memoizzare il risultato di un'operazione complessa, evitando di ricalcolarlo inutilmente a ogni render e che quindi funzioni con logiche complesse vengano rieseguite anche se il risultato ritornato sarebbe lo stesso perché le condizioni (es. parametri) non sono cambiate. È utile quando si manipolano grandi quantità di dati.

```jsx
const filteredItems = useMemo(() => items.filter(item => item.active), [items]);
```

>Anche qui è importante capire che aggiungendo della logica di controllo in più, non bisogna utilizzarlo per funzioni che non hanno un impatto sulle performance, o si rischierebbe solo di fare peggio. Va utilizzato solo per memoizzare il risultato di funzioni con logiche complesse.

## `useCallback`

`useCallback` è usato per memoizzare funzioni che vengono passate come props ai componenti figli. Questo previene la creazione di nuove istanze della funzione a ogni render.

Esempio:

```jsx
const handleClick = useCallback(() => {
  setCount(c => c + 1);
}, []);
```

Entrambi gli strumenti utilizzano la **memoizzazione**, che conserva i risultati (o le funzioni) fintanto che le dipendenze non cambiano.

>La sostanziale differenza è che `useMemo` memoizza il risultato di una funzione, mentre `useCallback` la funzione stessa.

>Non c'è bisogno di utilizzare `useCallback` con i setter ritornati da `useState`, poiché non cambiano, anche se in ogni caso non è un buon pattern passare direttamente il setter.

# Context e Ottimizzazione

Anche il **context** segue le stesse regole di re-render. Quando lo stato condiviso cambia, tutti i componenti che consumano quel contesto vengono ri-renderizzati. In questi casi, è utile usare `React.memo` o `useContext` combinati con `useMemo` per ridurre al minimo i calcoli inutili.

# Utilizzare key per ri-renderizzare/resettare un componente

Esistono casi in cui vorremmo che un componente figlio venga ri-renderizzato al cambio di una circostanza, senza bisogno che questo debba utilizzare uno `useEffect`.
Per fare ciò si può utilizzare l'attributo `key`, ogni volta che questo cambia, [React](React.md) ri-renderizza il componente:

```tsx
function MyComponent() {
	const [currentIndex, setCurrentIndex] = useState(0);

	function changeIndex() {
		setCurrentIndex(1);
	}

	return (
		<div>
			{currentIndex}
			<my-child key={currentIndex}></my-child> // Al cambio di currentIndex                                                       il child verrà ri-renderizzato
		</div>
	)
}
```

In questa maniera il child non dovrà utilizzare uno `useEffect` per attivare della logica.

Un esempio potrebbe essere il voler resettare un counter al cambio di un altro state.
Uno degli approcci potrebbe essere quello di usare uno `useEffect`:

```tsx
//...
const [changingValue, setChangingValue] = useState(2);
const [count, setCount] = useState(0);

useEffect(()=>{
	setCounter(0);
}, [changeMe]);

return (
	//...
	<MyCounter count={count}/>
	//...
)

```

>Questo approccio funzionerebbe ed è percorribile, ma l'utilizzo di `useEffect` andrebbe limitato al massimo, per questo esiste un altro approccio per resettare un componente figlio:

```tsx
//...
const [changingValue, setChangingValue] = useState(2);
const [count, setCount] = useState(0);

const handleSetChangingValue = () => {
	setChangingValue(...);
};

return (
	//...
	<MyCounter key={changingValue} count={count}/>
	//...
)
```

In questa maniera ogni volta che il valore viene cambiato, cambia anche la key del figlio, forzando il ri-render e quindi reset che componente figlio.

>Per questo motivo questo pattern può essere usato ogni volta che il cambio dello state deve comportare anche il reset di componenti figli.

# Conclusione

Ottimizzare React significa bilanciare il controllo manuale dei re-render con gli strumenti forniti dalla libreria. Mentre strumenti come `React.memo`, `useMemo` e `useCallback` possono ridurre i re-render, è importante utilizzarli solo quando necessario. Nella maggior parte dei casi, React è sufficientemente efficiente per gestire i re-render autonomamente.

Per approfondimenti:

- [Why React Re-Renders](https://www.joshwcomeau.com/react/why-react-re-renders/)
- [Understanding useMemo and useCallback](https://www.joshwcomeau.com/react/usememo-and-usecallback/)