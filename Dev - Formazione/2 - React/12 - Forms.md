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

Un modo elegante per l'handler potrebbe essere quello di farsi dare il nome del campo in modo da avere un handler unico:

```tsx
const [state, setState] = useState(
	{email: '', name: ''}
);

const handleStateChaange = (field: string, value) => {
	setState(prev => ({
		...prev,
		[field]: value
	}));
}

//...
<input ... value={state.name} type="text" name="name" 
	onChange={(event) => handleStateChange('name', event.target.value)}/>
<input ... value={state.email} type="email" name="email" 
	onChange={(event) => handleStateChange('email', event.target.value)}/>
```

# Refs e useRef

Come abbiamo visto in precedenza:

```tsx
//...
const nameRef = useRef();

const handleSubmit = () => {
	const name = nameRef.current.value;
}

//...
<input ... type="name" name="name" ref={nameRef}/>
```

# FormData & native browser features

Il browser mette a disposizione un oggetto che ci può aiutare nell'estrazione dei dati, `FormData`.
basta nel metodo che gestisce il submit, creare l'oggetto `FormData` passando al suo costruttore l'evento passata dall'`onSubmit`, che conterrà le info di tutto il form:

```tsx
//...
const handleSubmit = (event) => {
	event.preventDefault();

	const formData = new FormData(event.target);
	const name = formData.get('name');
	const password = formData.get('password');
}
//...
<form onSubmit="handleSubmit">
	<input name="name"...
	<input name="password"...
</form>
```

>È importante che tutti i campi del form abbiano l'attributo `name`.

Gestire al submit ogni singolo campo come nell'esempio precedente può essere tedioso, per questo normalmente viene usato un pattern molto pratico:

```tsx
const handleSubmit = (event) => {
	event.preventDefault();

	const formData = new FormData(event.target);

	const data = Object.fromEntries(formData.entries());
}
```

`entries()` restituirà un'array di tuple key/value (es. [ [ key, value ] ]) mentre `fromEntries` creerà un oggetto dall'array creata da `entries` con i corrispettivi `key/value`.

>Questo metodo però non restituirà i valori di campi di tipo `checkbox`, per poterli ottenere sarà necessario prima di tutto che ogni input `checkbox` appartenente allo stesso gruppo abbia lo stesso `name`. Poi per ottenerne i valori:

```tsx
const handleSubmit = (event) => {
	event.preventDefault();

	const formData = new FormData(event.target);
	const myCheckboxes = formData.getAll('myCheckboxes');
	const data = Object.fromEntries(formData.entries());

	data.checkboxes = myCheckboxes;
}

//...
<input name="myCheckboxes" type="checkbox"...
<input name="myCheckboxes" type="checkbox"...
```

# Reset form

