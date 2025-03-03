Fare il submit di un form e l'estrazione dei relativi campi è abbastanza semplice, ad es. è possibile:

1. Attraverso lo `state` e il `two-way binding[[]]`
2. Attraverso le `refs` e quindi `useRef`
3. Attraverso `FormData` e le feature messe a disposizione dal browser
4. Con [React 19](React%2019.md) abbiamo possiamo usare le `Form Actions`.

Quello che è un po più complicato è la validazione dei dati offrendo anche una buona `user experience`:

1. Possiamo validare ad ogni `keystroke` ma gli errori potrebbero essere mostrati troppo presto.
2. Possiamo validare alla perdita del `focus` ma gli errori potrebbero essere mostrati troppo a lungo.
3. Possiamo validare al `submit` del form ma gli errori potrebbero essere mostrati troppo tardi.

# 1. State e two-way binding

Metodo classico, utilizzerà:

- useState
- handler
- two-way binding su input field

```tsx
// ...
const [state, setState] = useState("Pippo");

const handleStateChaange = (event) => {
	setState(event.target.value);
}

//...
<input ... value={state} onChange={handleStateChange}/>
```

>Ovviamente non dobbiamo per forza avere uno state per ogni campo di input, possiamo anche gestire il form con un unico oggetto e quindi un unico setter.

