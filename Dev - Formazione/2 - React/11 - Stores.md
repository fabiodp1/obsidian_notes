# MobX

## WHAT

`MobX` è una libreria che si basa sui `signal`, iper-testata, che permette di gestire lo state in maniera semplice e scalabile (non solo store). La filosofia dietro `MobX` è semplice:

- **Straightforward**: codice senza `boilerplate`. Per aggiornare il valore di un campo basta fare una normale assegnazione JS ( x = y ). Il sistema di reattività si occuperà di captarlo e propagarlo per scatenare gli aggiornamenti. Non servono utility speciali per aggiornare dati in maniera asincrona.
- **Rendering ottimizzato**: tutti i cambiamenti dello state vengono captati a runtime, creando un albero di dipendenze che cattura tutte le relazioni fra state e output. Tutto ottimizzato, nessun bisogno di ottimizzare manualmente i componenti tramite memoizzazione.
- **Libertà architetturale**: `MobX` è non-opinionata, permettendo di gestire lo state al di fuori del framework UI. Questo rende il codice disaccoppiato, flessibile e soprattutto *facilmente testabile*.

## HOW

Esistono diversi modi per utilizzare `MobX`:

- Classi + utility `makeObservable`;
- Classi + decoratori;
- Factory Function + `makeAutoObservable`;

Il metodo più semplice e diretto di utilizzare `MobX` con [React](React.md) e in maniera funzionale, è l'ultimo.
Per fare ciò basta creare una `Factory Function` che ritorni il nostro state wrappato dall'utility `makeAutoObservable`:

```ts
import { makeAutoObservable } from "mobx"

function createDoubler(value) {
    return makeAutoObservable({
	    // Prop
        value,
        
        // Computed (derived state)
        get double() {
            return this.value * 2
        },
        
        // Method
        increment() {
            this.value++
        }
    })
}
```

I componenti che dipenderanno dallo state andranno wrappati con il metodo `observer`:

```tsx title:DoublerView.tsx
const { double, increment } = createDoubler(2);

const DoublerView = observer(({ timer }) => (
	<>
	  <h3>Current value: { double }</h3>
      <button onClick={ increment }>
        Increment
      </button>
    </>
));
```

### makeAutoObservable

Di default fa l'`infer` di tutte le proprietà del nostro `state`, e quindi il `Type` (sarà comunque possibile fare l'override della configurazione di default).
Nello state che ci verrà ritornato:

- Tutte le proprietà dichiarate diventeranno `observable` (reattive), e in maniera ricorsiva;
- Tutti i `getter` diventeranno `computed`;
- Tutti i `setter` diventeranno `action`;
- Tutte le funzioni diventeranno `autoAction`;
- I membri marcati come false nell'argomento `overrides`, non saranno annotate. Per esempio può essere usato per campi `read only`.

>La reattività è **DEEP**, nel senso che anche se una proprietà dichiarata è un oggetto profondamente innestato, in maniera ricorsiva sarà reso interamente reattivo, per cui non ci sarà bisogno di utilizzare `spred operators` ecc.

## MobX come Store + React

In [React](React.md) normalmente avremo uno store principale che detiene l'app state, e magari (non sempre) un UI state.

Per dichiarare uno state che sia un store, condiviso in tutto l'applicativo, basta creare/instanziare il nostro store ed esportarlo, in modo che funga da `singletone` condiviso:

```ts title:myStore.ts
import { makeAutoObservable } from "mobx"

// Factory Function
function createStore() {
    return makeAutoObservable({
	    // Prop
        isLogged: false,
        items: [],

        // Method
        addItem(item) {
            this.items.push(item);
        },
        
        // Computed (derived state)
        get itemsLength() {
            return this.items.length;
        },
    })
}

export const myStore = createStore();
```

I componenti per usarlo basterà che lo importino e utilizzino come un normale oggetto JS.

### Sub-stores

Potrebbe capitare di avere necessità di gestire dei sub-store, ad esempio se abbiamo una lista di user che vorremmo ognuno di questi avesse una gestione separata dello state, con proprie `computed` e metodi ecc.

La versatilità di `MobX` ci permette di crearli e gestirli in maniera semplice:

```ts title:createUserStore.ts
export const createUserStore = (user) => {
	return makeAutoObservable(
	  {
	    // Added state
	    isSelected: false,

        // Added computed
        get fullName() {
          return `${user.firstName} ${user.lastName}`;
        }
	    ...user
	  }
	)
}
```

Lo state principale potrà così creare/inizializzare e utilizzare lo state aggiunto dei singoli elementi, che manterranno una propria gestione e reattività indipendente:

```ts title:appStore.ts
// Factory Function
function createStore() {
    return makeAutoObservable({
	    // Prop
        isLogged: false,
        users: [],

        // Method
        addUser(item) {
            this.users.push(createUserStore(user));
        },
        
        // Computed (derived state from sub-states)
        get selectedUsers() {
            return this.users.filter(u => u.isSelected);
        },
    })
}

export const usersStore = createStore();
```

# Redux

## WHAT

>[[Redux]] è un sistema di gestione dello stato trasversalmente ai componenti e il resto dell'app.

Permette quindi di gestire lo [state](state), quindi dati che al cambiamento dovrebbero coinvolgere la [UI](UI).
Possiamo suddividere gli `state` in 3 tipologie:

- **`Local State`**
	- Lo stato che appartiene ad un **singolo componente**, in genere gestito usando `useState` e `useReducer`.
	
- **`Cross-component State`**
	- State che coinvolge **diversi componenti**, ad es. per l'apertura o chiusura di un componente Modal. Anche qui si tratta di state creato con `useState` o `useReducer` e passato attraverso `prop drilling`.
	
- **`App-wide state`**
	- State che coinvolge **tutto l'applicativo**, ad es. il theme scelto e lo stato di autenticazione di un utente. Anche questo richiede `prop drilling`

>Gli ultimi due possono essere più complessi da gestire attraverso il `prop drilling`, rispetto al local state, per questo spesso viene usato il `Context`. [Redux](Redux) aiuta a risolvere lo stesso tipo di problema.

---

## WHY

Se [Redux](Redux) è semplicemente un'alternativa ai [6 - Context](6%20-%20Context.md), perché dovremmo usarlo?

### Contro del Context

Parliamo di potenziali 'contro', perché potrebbe essere che nella nostra app non rappresentino un problema.

- In certi casi potremmo avere **elevata complessità di setup e gestione**
	- Soprattutto in grandi applicazioni, potremmo avere multipli `Context` per la gestione di diversi tipi di state. Ovviamente potremmo anche crearne uno unico, ma diventerebbe un `Provider` molto grande e difficile da gestire, da manutenere e avrebbe anche state che non hanno relazione diretta fra loro.
- Problemi di **performance** per cambiamenti frequenti allo stato
	- È consigliato soprattutto per state che vengono aggiornati poco frequentemente, ad es. cambio theme e autenticazione, ma no se i dati cambiano frequentemente. **Non può in tutti gli scenari essere utilizzato** come `store`.

Librerie come `Redux` nascono per risolvere queste criticità.

---

## HOW

[[Redux]] nasce per poter creare uno `Central Data Store`, avere tutti dati (state) centralizzati in **un unico** store.
Questo potrebbe sembrare difficile da mantenere, come per il Context, ma in realtà non abbiamo bisogno ogni volta di gestire direttamente l'intero store,

I componenti che hanno bisogno di utilizzare lo store, fanno una `Subscription` ad esso e ogni volta che lo state all'interno dello store cambia, il componente verrà notificato per reagire di conseguenza.

>I componenti non manipolano **===MAI===** lo stato dello store in maniera diretta, per fare ciò utilizzano (attraverso le `Action`) i **Reducer**.

I `Reducer` sono delle funzioni che si occupano di modificare lo state dello store e vengono attivati da delle funzione di `Dispatch` usati dai componenti, le `Action`.
Le `Action` sono degli oggetti che descrivono le azioni che i `Reducer` devono compiere. 
Quindi il componente chiama l'`Action`, [[React]] la passa al `Reducer` che genererà il nuovo `state` e tutti i componenti che hanno fatto la `Subscription` a quello state verranno informati.

---

## Concepts

### Reducer

Sono delle funzioni [JS](JS) che prendono come argomenti 2 parametri:

- il vecchio `state`
- la `action` di cui è stato fatto il `dispatch`

Ritornerà **SEMPRE** un nuovo oggetto `state` e deve essere una funzione "pura", dato un input deve ritornare sempre lo stesso output.

>Semplice esempio che non usa React perché [[Redux]] può essere usato in **qualsiasi progetto JS**, per usarlo con React basta installare `react-redux`.

```tsx
const redux = require('redux');

// Lo state deve avere un default perchè se no alla creazione sarà undefined
const counterReducer = (state = { counter: 0 }, action) => {
	return {
		counter: state.counter + 1
	}
};

// Lo store deve sapere quali sono i Reducer che lo modificheranno
const store = redux.createStore(counterReducer);
```

>Nel momento in cui lo store viene creato, i reducer verranno chiamati per la prima volta, quindi è importante dare un default alla proprietà `state` del `Reducer` o essendo al primo avvio `undefined`, cercare di accedere alle prop dello state darebbe errore.

### Subscriber

Il subscriber è una funzione che restituirà tutto o parte dello state, e andrà registrato nello store tramite il metodo `subscribe`.

```tsx
const counterSubscriber = () => {
	// getState è un metodo built-in che ci darà l'ultimo state disponibile
	const latestState = store.getState();
};

// Adesso il subscriber viene sottoscritto allo store
store.subscribe(counterSubscriber);
```

### Action

Un po come si fa con `useReducer`, la `Action` è un oggetto che tramite una prop `type`, definisce il tipo di azione che andrà fatta dal `Reducer`

```tsx
// dispatch è un metodo built-in che fa il dispatch di una Action
store.dispatch({ type: 'increment'}); // qui lo stiamo invocando
// OR
store.dispatch({ type: 'decrement'});
```

In questa maniera il reducer potrà ad es. gestire la logica così:

```tsx
const counterReducer = (state = {counter: 0}, action) => {
	if(action.type === 'increment') {
		return {
			counter: state.counter + 1
		}
	}

	if(action.type === 'decrement') {
		return {
			counter: state.counter - 1
		}
	}
	
	return state;
}
```

---

## React & react-redux

Con `React` installeremo `react-redux` che semplifica un po le cose.

### Creare lo store

Creiamo un file `.ts` (o .js) nella cartella che conterrà lo store:

```ts
import {createStore} from 'redux';

const counterReducer = (state = {counter: 0}, action) => {
	if(action.type === "increment") {
		return {
			counter: state.counter + 1
		}
	}
	if(action.type === "decrement") {
		return {
			counter: state.counter - 1
		}
	}

	return state;
};

const store = createStore(counterReducer);

export default store;
```

A differenza di quello che abbiamo visto prima, non sarà in questo file che faremo il subscribe:

Nel file di root dell'applicativo `index.ts` registreremo lo store importando il componente `Provider` (un vero a proprio `Context`) e con questo faremo il wrap dell'applicativo:

```ts
import { Provider } from 'react-redux';
import store from './store/index';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
	<Provider store={store}> // Adesso tutti i componenti possono accedere allo store
		<App />
	</Provider>
)
```

>Come tutti i `Context`, non è obbligatorio fare il wrap a livello di root, possiamo farlo anche innestato in punti diversi del tree di componenti.

### useSelector

Per leggere lo store in componente faremo uso di un custom hook appartenente alla libreria `react-redux`, `useSelector`. A questo [[hook]] va passata una funzione che verrà eseguita da `Redux` per estrarre la parte di state che ci serve:

```tsx
import { useSelector } from 'react-redux';

const Counter = () => {

	// In questo caso lo state da estrarre è semplice
	const counter = useSelector(state => state.counter);
	
	return (
		<div>{ counter }</div>
	)
};
```

>`useSelector` si occuperà di fare la `subscription` allo store per questo componente, quindi il componente verrà aggiornato ogni volta che lo stato per cui è stata fatta la subscription cambia.
>Si occuperà anche di cancellare la sottoscrizione al destroy del componente.

### useDispatch

Per fare il `dispatch` di un'`action` all'interno di un componente possiamo usare l'`hook` di react-redux `useDispatch.

```tsx
import {useDispatch} from 'react-redux';

const Counter = () => {
	const dispatch = useDispatch();

	const incrementHandler = () => {
		dispatch({type: 'increment'});
	}
	//...
}
```

#### Passare un payload

Come per le funzioni reducer utilizzate da `useReduce`, anche qui assieme al tipo di azione, l'`action` può contenere un payload:

```ts
const counterReducer = (state ..., action) => {
	//...
	if(action.type === 'increase') {
		return {
			counter: state.counter + action.amount // o 'value', quale vogliamo
		}
	}
	//...
}

//...
// COMPONENT

const increaseHandler = (amount: number) => {
	dispatch({type: 'increase', amount});
};

```

### Proprietà multiple dello state

```ts
const initialState = {
	counter: 0, 
	showCounter: true
}

const counterReducer = (state = initialState, action) => {
	// si potrebbe usare anche uno switch
	if(action.type === "increment") {
		return {
			counter: state.counter + 1,
			showCounter: state.showCounter // va inserita nello state ritornato, si può anche usare lo spred
		}
	}
	if(action.type === "decrement") {
		return {
			...state,
			counter: state.counter - 1,
		}
	}

	if(action.type === 'toggle'){
		return {
			...state,
			showCounter: !state.showCounter
		}
	}

	return state;
};

// COMPONENT
//...
// useSelector può essere usato più volte
const showCounter = useSelector(stete => state.showCounter);
```

>È necessario ritornare lo state per intero, perché [Redux](Redux) non farà il merge con lo stato corrente, ma lo sostituirà in toto. Se non passiamo una proprietà, questa verra valorizzata con `undefined`, creando side effects non voluti.

Quello che ci si potrebbe chiedere è perché ogni volta dobbiamo ritornare un oggetto invece di mutare lo state in maniera più semplice, ad es.

```ts
const counterReducer = (state = initialState, action) => {
	// si potrebbe usare anche uno switch
	if(action.type === "increment") {
		state.counter++
		return state;
	}
	//....

// OR

const counterReducer = (state = initialState, action) => {
	// si potrebbe usare anche uno switch
	if(action.type === "increment") {
		state.counter++
		
		return {
			counter: state.counter,
			showCounter: state.showCounter
		};
	}
```

==**WRONG**== :Se facciamo in questo modo, l'applicativo apparentemente sembra funzionare, ma non è per niente corretto, lo state è `immutable`, come ogni state di React. Facendo così incorriamo in side effects non voluti e bug.

---

## Redux Toolkit

Il fatto di dover stare molto attenti a copiare lo state correttamente prima che il reducer lo possa ritornare per rispettare l'immutabilità dello state di [Redux](Redux) può essere tedioso e pericolosi man mano che l'applicativo e lo state crescono.

>Installando `@reduxjs/toolkit` è possibile disinstallare `redux`, poiché è già incluso nel pacchetto

[[Redux Toolkit]] semplifica molte cose nell'utilizzo di [Redux](Redux), partendo dal file di root dello store dove dichiariamo il `Reducer`.

### createSlice

Aiuta nella creazione del `Reducer`. Esiste anche `createReducer`, ma è meno potente.
Accetta un oggetto che rappresenta una fetta, "slice", dello state.

>Possiamo creare quante slice vogliamo in quanti file vogliamo, rendendo lo store più facile da mantenere.

```ts
import {createSlice} from "@reduxjs/toolkit"

const initialState = { counter: 0, showCounter: true };

const counterSlice = createSlice({
	name: 'counter', // Nome della slice
	initialState,
	// tutti i reducer di cui avrà bisogno questa slice
	reducers: {
		// Non c'è più bisogno di fare i check sul type della action in un singolo 
		// reducer, al dispatch verrà chiamato il metodo corrispondente
		increment(state) {
			// Qui È possibile cambiare direttamente lo state
			state.counter++;			
		},
		decrement() {...},
		increase(state, action) {
			// comunque i reducer ricevono l'action, 
			// quindi è possibile usare il PAYLOAD
			state.counter += action.amount;
		},
		toggleCounter() {...}
	}
});
```

#### Unire più Slice

Per fare ciò viene utilizzato un altro [[hook]] del toolkit, `configureStore` che permette di configurare e creare uno store, ma rendendo facile l'unione fra più store:

```ts
const store = configureStore({
	// All fine Redux vuole in ogni caso un unico reducer globale
	
	// reducer: counterSlice.reducer // può essere usato così
	
	// Per unirne diversi
	reducer: {
		// Verrà fatto il merge fra tutti i reducer passati
		counter: counterSlice.reducer,
		...
	}
});
```

### Actions

Per accedere alle action possiamo fare tramite la prop `actions`:

```ts
counterSlice.actions.increment();

// Quindi dallo store possiamo anche farne l'export e importarli nei componenti
export const counterActions = counterSlice.actions;
```

Dietro le quinte `toolkit` creerà la action per noi.
All'interno dei componenti potremo usarli così:

```ts
import { counterActions } from './store/index';

//...
const incrementHandler = () => {
	dispatch(counterActions.increment());
};

// PAYLOAD
const increaseHandler = () => {
	dispatch(counterActions.increase(5));
};
```

Se l'action viene lanciata con un payload, nel reducer questo andrà estratto usando la key `payload` (verrà creato di default una action con forma `{ type: 'increase', payload: 5 }`)