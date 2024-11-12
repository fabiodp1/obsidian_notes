In React gli [[hook]] vengono utilizzati per la gestione dello state, i più comuni sono:
- [[useState]]
- [[useEffect]]
- [[useContext]]
- [[useRef]]
- [[useReducer]]
Tutti questi possono essere utilizzati all'interno di un componente React per storare dati (useState) o compiere azioni (side-effects).

Ognuno di questi hook possiede una propria [[API]], alcune ritornano un valore (come useRef e useContext), altre coppie di valori (useState, useReducer) e altri non ritornano niente (useEffect).

# [[useState]]
E' una funzione che accetta un parametro come initial state e ritorna una coppia di valori, ritornando una array con 2 valori (utilizzato con la destrutturazione per dare un nome ai due valori), lo stato e una funzione setter per cambiare lo stato (per convenzione viene nominata iniziando con 'set').

> useState può prendere come valore iniziale sia il valore direttamente, o una funzione che lo restituisca. La differenza è che passando una funzione, questa verrà chiamata solo al primo mount del componente, mentre nel primo caso il valore verrà estrapolato ogni volta che la funzione del componente verrà chiamata (ad es. al ri-render).
> Passando una funzione ci assicuriamo, nel caso in cui il nostro valore sia il risultato di un'azione, che la stessa azione venga ripetuta ad ogni render, creando rallentamenti.

```typescript
// In questa maniera verrà fatta la lettura dello store ad ogni render
React.useState(window.localStorage.getItem('name') ?? '')

// Così verrà chiamato solo al primo render/mount
React.useState(() => window.localStorage.getItem('name') ?? '')
```

# [[useEffect]]
Permette di fare delle azioni dopo che React ha renderizzato (e ri-renderizzato) il componente nel [[DOM]] (quindi da considerare che viene chiamata ogni volta).
Accetta una funzione di callback che React chiamerà DOPO l'update del DOM:

```javascript
React.useEffect(() => {
  // your side-effect code here.
  // this is where you can make HTTP requests or interact with browser APIs.
})
```

![[Pasted image 20241112161806.png]]