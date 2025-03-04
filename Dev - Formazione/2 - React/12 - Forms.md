# Data management

Fare il submit di un form e l'estrazione dei relativi campi è abbastanza semplice, ad es. è possibile:

1. Attraverso lo `state` e il `two-way binding[[]]`
2. Attraverso le `refs` e quindi `useRef`
3. Attraverso `FormData` e le feature messe a disposizione dal browser
4. Con [React 19+](React%2019+.md) abbiamo possiamo usare le `Form Actions`.

Quello che è un po più complicato è la validazione dei dati offrendo anche una buona `user experience`:

1. Possiamo validare ad ogni `keystroke` ma gli errori potrebbero essere mostrati troppo presto.
2. Possiamo validare alla perdita del `focus` ma gli errori potrebbero essere mostrati troppo a lungo.
3. Possiamo validare al `submit` del form ma gli errori potrebbero essere mostrati troppo tardi.

## State e two-way binding

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

## Refs e useRef

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

## FormData & native browser features

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

## Reset form

Esistono diversi modi per resettare un form:

- usando la built-in feature del campo `input`:

```tsx
//...
<button type="reset" ...>Reset</button>
//...
```

- Programmaticamente, semplicemente rimettendo lo state ad un valore di default.
	- Se abbiamo usato `useRef` bisognerà anche li resettarli manualmente in maniera imperativa, ma non è un pattern consigliato perché dovrebbe essere [React](React.md) ad occuparsi della manipolazione del [DOM](DOM). È meglio usare i ref solo in lettura.
- Utilizzando il metodo `reset()` fornito dall'evento `onSubmit`, è questa è una forma imperativa ma sempre meglio di resettare ogni singolo campo usando i ref:

```tsx
//...
//...
const handleSubmit = (event) => {
	event.preventDefault();

	const formData = new FormData(event.target);
	const myCheckboxes = formData.getAll('myCheckboxes');
	const data = Object.fromEntries(formData.entries());

	data.checkboxes = myCheckboxes;

	event.target.reset(); // <==
}
//...
<form onSubmit="handleSubmit">
	//...
</form>
```

>Il metodo `reset` è lo stesso che viene chiamata quando usiamo il built-in `type="reset"`.

---
# Validation

## Al keystroke

Per poter avere questo tipo di validazione, non è fattibile usare i metodi visti prima come `useRef`, ma servirà una gestione `statefull`.

Uno dei modi è utilizzare delle computed che ad ogni re-render vengono aggiornati:

```tsx
//...

const isEmailInvalid = !email.includes('@'); // solo un esempio di validazione

//...
{isEmailInvalid && <div>Please enter a valid email</div>}
//...
```

Nell'esempio il messaggio comparirà da subito e in molti casi vorremmo dare all'utente la possibilità di inserire un valore corretto, prima di mostrare il messaggio di errore. Ad esempio controllando nella computed se il campo non è una stringa vuota e quindi l'utente ha già iniziato a scrivere. Questo però non risolve il fatto che alla prima lettera inserita comparirà il messaggio di errore.

## Alla perdita del focus

A differenza della validazione al `keystroke`, questo metodo permette di evitare che il messaggio di errore compaia troppo presto.
Per fare ciò facciamo utilizzo del built-in event `onBlur`, scatenato ogni volta che il campo input perde focus:

```tsx
//...
const [didEdit, setDidEdit] = useState(
	email: false,
	password: false
); // Uno dei modi per gestirlo, potrebbe anche essere un campo `isValid` all'interno dello state principale

const handleInputBlur = (identifier: string) => {
	setDidEdit(prev => ({
		...prev, 
		[identifier]: true
	}));
};

//...
<input ... onBlur={() => handleInputBlur('email')} .../>
```

Il tradeoff è che i messaggi di errore potrebbero rimanere troppo a lungo visibili. Infatti una volta comparso, se l'utente modifica il valore l'errore scomparirà solo quando il valore sarà corretto. Per questo va gestito anche al change del valore:

```tsx
//...
const handleInputChange = (identifier: string, value: string) => {
	setEnteredValues(prev => ({
		...prev,
		[identifier]: value
	}));

	setDidEdit(prev => ({
		...prev,
		[identifier]: false // <==
	}));
};
```

## Al submit

Per validare il nostro form al submit e mostrare possibili errori nella UI, possiamo fare così:


```tsx
const [emailIsInvalid, setEmailIsInvalid] = useState(false);
const email = useRef();

const handleSubmit = (event) => {
	event.preventDefault();
	
	const enteredEmail = email.current.value;
	const emailIsValid = enteredEmail.includes('@');

	if(!emailIsValid) {
		setEmailIsInvalid(true);
		return;
	}

	setEmailIsInvalid(false); // This way the DOM will update removing possible error message
}
```

Qui il tradeoff è che l'utente avrà un feedback sulla correttezza dei dati solo al submit.

>In ogni caso validare i dati al submit è sempre una buona idea, anche se li stiamo già validando ad es. al `keystroke`.

### Built-in validation Props

Un altro tipo di validazione al `submit` è quella di usare le built-in props come `type="email"` (che permette di specificare il tipo di dato), ad esempio la prop `required`, o `minLength` ecc.:

- [Client-side form validation - Learn web development | MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation#using_built-in_form_validation)

Al submit o al focus sui campi di input, i messaggi di errore verranno gestiti.
Se non abbiamo il controllo sui messaggi di errore, però questo metodo facilita molto il check sulla validità dei campi.

### Mix fra custom e built-in

Il fatto di usare built-in props non esclude l'utilizza anche di custom validations. Ad es. se volessimo controllare che la password inserita e la "confirm password" sono uguali, possiamo aggiungere questo check all'interno del metodo `handleSubmit` come usato negli esempi precedenti e aggiungendo l'elemento html che mostrerà il messaggio in base al valore dello state (vedi esempi sopra).

---
# Actions ([[React 19+]])

È una feature presente da [React 19+](React%2019+.md) in poi e sfrutta l'attributo `action` degli elementi html `form` (esiste indipendentemente da React) che normalmente serve a definire il path per la chiamata del submit (l'endpoint).

Utilizzandolo con React, verrà fatto dietro le quinte un override dell'attributo, venendo gestito da React. Il metodo non riceverà un evento come per il `onSubmit`, ma un oggetto `FormData`:

```tsx
//...
const submitAction = (fromData: FormData) => {
	
}; 

//...
<form action={submitAction}...
```

>In questa maniera possiamo gestire il form come abbiamo visto in `FormData & native browser features`. A differenza però di quel metodo, React subito dopo il submit farà anche il reset (comportamento gestibile).

## Validation

Per la validazione possiamo gestire il form come visto prima, creandoci dei metodi di validazione, utilizzando librerie esterne, usando built-in props ecc.

## useActionState

È un hook messo a disposizione da [React 19+](React%2019+.md) che permette di gestire lo stato del form più agilmente.
Richiede 2 parametri, la funzione che gestirà il `submit` e l'`initial state` del form.

>	Utilizzando l'handler con `useActionState`,  Il metodo riceverà un parametro in più che contiene lo stato precedente del form.

```tsx
//...
const submitAction = (prevState: FormData, formData: FormData) => {
	//...
}; 

const [formState, formAction, pending] = useActionState(submitAction, {errors: null});

//...
<form action={formAction}
```

`useActionState` ritornerà:

- `formState`: rappresenta lo stato aggiornato del form
- `formAction`: è un metodo creato da React che fa il wrap del metodo che abbiamo passato come parametro e che si mette in ascolto dell'esecuzione di questo, fornendocene una versione "potenziata". Sarà questo nuovo metodo ad essere usato nel template come valore dell'attributo `action`.
- `pending`: indica se è stato fatto il submit o meno, utile per sapere lo stato delle operazioni `async`

## Manage form reset

Come detto prima di default una volta fatto il submit tramite action, React farà il reset dello state del form, ma non sempre vorremmo fosse questo il comportamento.
Per gestire il reset possiamo sfruttare il nostro metodo che si occupa del submit, e l'attributo html `defaultValue` (o nel caso degli elementi di tipo checkbox `defaultChecked`):

```tsx
//...
const submitAction = (prevState: FormData, formData: FormData) => {
	//...
	const password = formData.get('password');
	const errors = [];

	// Se sono presenti errori voglio che il reset conservi i valori
	if(errors.length){
		return {
			errors,
			enteredValues: {
				password,
				//...
		}
	}

	// Non essendoci errori va bene che il reset azzeri i campi
	return {
		errors: null
	}
};

const [formState, formAction, pending] = useActionState(submitAction, {
	errors: null,
	enteredValues: {
		password: ''
	}
});

//...
// Se il campo è presente al reset verrà impostato il suo valore
<input name="password" defaultValue={formState.enteredValues?.passowrd}>...
```

In questa maniera al submit React reimposterà i valori correnti, senza resettarli.

>Questo influenzerà anche il funzionamento standard dei button `type="reset"`.

>Poiché non utilizzerà nessuno stato del componente, non è necessario definire il metodo che gestisce il submit dentro il componente, può essere esternalizzato e importato.

## useFormStatus

Come abbiamo visto possiamo usare il booleano `pending` fornito da `useActionState` per sapere se l'action submit ha concluso la sua logica (probabilmente asincrona).

Ma esiste un altro [hook](hook) messo a disposizione da [React](React.md) che può essere molto utile e che ===va utilizzato assieme alle form action===, `useFormStatus`.

>**NON** può essere utilizzato dal componente che utilizza il `form`, ma deve essere utilizzato da un componente `nested` all'interno del form, che gestirà il submit.

```tsx
import { useFormStatus } from 'react-dom';

export default function Submit() {
	const { pending, data, method, action } = useFormStatus();

	return (
		<p>
			<button type="submit" disabled={pending}>
				{ pending ? "Submitting..." : "Submit"}
			</button>
		</p>
	)
}
```

## Multiple form actions

Ci sono casi in cui abbiamo bisogno di gestire più form action, ad esempio avendo bottoni che fanno cose diverse e chiamate diverse, quindi ognuno di loro sarebbe un submit e avremmo quindi più di un handler.

In questi casi possiamo evitare di usare l'attributo `action` a livello di `<form>` e invece utilizzare l'attributo `formAction` a livello di singolo elemento [HTML](HTML.md):

```tsx
//...
const handleFirstAction = (formData) => {
	// ...
};

const handleSecondAction = (formData) => {
	// ...
};

//...
<form>
	//...
	<button formAction={handleFirstAction} ...>
	<button formAction={handleSecondAction} ...>
```

Anche qui possiamo gestire diversamente lo state del form ad es. per avere lo stato `pending`, infatti possiamo usare `useFormStatus`, ma sarebbe necessario creare dei sottocomponenti da innestare.
Oppure possiamo usare `useActionState`:

```tsx
//...
const handleFirstAction = (prevData, formData) => {
	// ...
};

const handleSecondAction = (prevData, formData) => {
	// ...
};

const [firstFormState, firstFormAction, firstPending] = useActionState(handleFirstAction);

const [secondFormState, secondFormAction, secondPending] = useActionState(handleSecondAction);

//...
<form>
	//...
	<button formAction={firstFormAction} disabled={firstPending || secondPending} ...>
	<button formAction={secondFormAction} disabled={firstPending || secondPending} ...>
```

## useOptimistic

I metodi utilizzati fin adesso per gestire i form sono corretti ma in certi casi (ad es. in seguito a chiamate server lente), possono rendere la [[UX]] non ottimale e poco reattiva.
Per questo esiste un altro [[hook]] che permette di gestire agevolmente l'[optimistic UI](optimistic%20UI), `useOptimistic`.
Accetta 2 parametri:

- Il campo da gestire
- Il metodo che si occuperà della logica di aggiornamento

Restituisce 3 parametri:

- 

```tsx
//...
const 
```