Normalmente viene utilizzato [[react-router]]:
[Docs | React Router](https://reactrouter.com/en/main)

>Di seguito viene mostrato come configurare rotte usando i componenti di `react-router-dom`, ma è anche possibile farlo usando l'hook `createBrowserRouter` e `RouterProvider`.
# BrowserRouter / Routes / Route

La libreria possiede i suoi componenti e i suoi [[hook]] per manipolare e gestire il [[routing]].
Normalmente per introdurre il routing in una web app si procede così:
1. Si importano il componente `BrouwserRouter` che wrapperà la parte di applicazione che utilizzarà il routing

```ts
import {BrowserRouter as Router} from 'react-router-dom'
...
<Router>
  <MyAppWithRouting {...props} />
</Router>
...
```

2. I componenti al suo interno utilizzeranno il componente `Routes` per definere le rotte e i relativi componenti

```ts
...
(<div>
	...
	<main css={{width: '100%'}}>
	    <AppRoutes user={user} />
    </main>
	 ...
</div>)
...

import {Routes, Route, Link} from 'react-router-dom'

function AppRoutes({user}) {
  return (
    <Routes>
      <Route path="/discover" element={<DiscoverBooksScreen user={user} />} />
      <Route path="/book/:bookId" element={<BookScreen user={user} />}>
	    // Nested route
		<Route path="cover" element={<BookCover />} />
	  </Route>
      <Route path="*" element={<NotFoundScreen />} />
    </Routes>
  )
}
```

---

# Layout

Spesso nell'applicativo abbiamo per una pagina un determinato layout, es. navbar e menu laterale, che contiene le pagine corrispondenti allo route corrente.
Per fare ciò viene usato il componente `Outlet` che indica dove i componenti child devono essere renderizzati:

```tsx
<Route element={<AuthLayout />}>
	<Route path="/login" element={<Login />} />
	<Route path="/register" element={<Register />} />
</Route>

//..
export const AuthLayout = ()=> {
	reuturn (
		//...
		<div>
			<h1> Layout Title </h1>
			// This is where the child components will be rendered
			<Outlet/>
		</div>
	)
}
```

---

# useMatch
Se vogliamo sapere se la route corrente fa match con una determinata path, utilizziamo l'hook [[useMatch]]:

```ts
import {Link, useMatch} from 'react-router-dom'

function NavLink(props) {
  const match = useMatch(props.to)
  return (
    <Link
      css={[
        {
          borderLeft: '5px solid transparent',
          ':hover': {
            color: colors.indigo,
            textDecoration: 'none',
            background: colors.gray10,
          },
         },
        },
        match
          ? {
              borderLeft: `5px solid ${colors.indigo}`,
              background: colors.gray10,
              ':hover': {
                background: colors.gray10,
              },
            }
          : null,
      ]}
      {...props}
    />
  )
}
```

---

# Link

Se vogliamo creare dei link alle pagine, ad es. in una barra di navigazione, utilizziamo il componente `<Link />`:

```ts
import { Link } from 'react-router-dom'
//...
<Link to="/discover">Discover</Link>
// Sintassi speciale per indicare di andare indietro
<Link to="..">Back</Link>
//...
```

Un'alternativa al componente `Link` è il componente **`<NavLink />`**, che funziona alla stessa maniera ma funziona un po diversamente, infatti le classi e gli stili possono essere passati fornendo una funzione che avrà come parametro un oggetto con diverse proprietà fra cui `isActive`, per assegnare classi in base allo stato del link:

```tsx
//...
<NavLink to="/" className={({ isActive }) => (isActive ? '...' : '...')}
	style={({ isActive }) => ({ textAlign: isActive ? 'center' : 'left' })} >
	Home
</NavLink>
//...
```

>Bisogna però fare attenzione al fatto che ad es. nell'esempio la route è `/`, questo vuol dire che la classe verrà assegnata alla route `/` ma anche a tutte le sotto route (es. `/products`). Per questo esiste la proprietà `end`, che indicherà che la route deve terminare con la path indicata:

```tsx
//...
<NavLink to="/" end className={({ isActive }) => (isActive ? '...' : '...')} >
	Home
</NavLink>
```

---

# Error Page

Possiamo anche definire una pagina di default nel caso in cui l'utente voglia andare in una pagina non esistente. Potrebbero però esserci diversi tipi di errore che vorremmo gestire con pagine diverse, per questo esiste la prop apposita `errorElement`:

```tsx
return (
    <Routes>
      <Route path="/" element={<DiscoverBooksScreen user={user} />} 
      errorElement={<ErrorPage/>}/>
    //...
```

In questa maniera nel momento in cui avviene un errore nella path `/` (quindi l'intero applicativo), si verrà rediretti a `<ErrorPage />`. Essendo il `404` un errore, verrà servita la pagina di errore.

---

# Navigare programmaticamente

Se volessimo cambiare route in maniera programmatica e imperativa, `react-router` mette a disposizione l'[[hook]] `useNavigate`:

```tsx
import { useNavigate } from 'react-router-dom';

//...
const navigate = useNavigate();

function navigateHandler() {
	nabigate('/products');
}
//...
```

---

# Dynamic Routes

```tsx
return (
    <Routes>
	  // i due punti indicano che quel valore è dinamico
      <Route path="/products/:id" element={<ProductDetails user={user}/>
    //...
```

Per estrarre il parametro useremo l'[hook](hook) `useParams`:

```tsx
import { useParams }

export default function ProductDetails () {
	const params = useParams();
	const productId = params.id;

	//...
}
```

# Relative and Absolute paths

>Se la route inizia con `/` avremo una **absolute path**, quindi ad esempio cliccando su un link verremo rimandati a quel path che verrà subito dopo il dominio.
>Se invece non viene anteposto il `/`, ad es. `products/:id`, ad es. al click di un link avente quel path, verremo indirizzati al path indicato *che però verrà aggiunto al path padre*.

Esiste anche la prop `relative: 'path' | 'route'` che permette di scegliere a quale parte della route andrà aggiunta la *relative path* alla prop `to`.
Di default è `route`, quindi ad es. se il parent di `products/:id` è `/root`, facendo back andremo su `/root`, se invece è `path`, React router rimuoverà un segmento dalla path andando indietro relativamente alla path. Quindi da `products/:id` andremo in `products`:

```tsx
//...
<Link to=".." relative='path'>Back</Link>
```

---

# Index route

Negli esempi precedenti abbiamo impostato la route principale `/` che espone il `Layout` dell'applicativo, e una prima route figlia con path `""` in modo da indicare che dovrebbe essere la pagina principale.
Esiste però un metodo più leggibile per gestire questo caso, ed è quello di usare la prop `index`, che serve ad indicare qual è la route `index`:

```tsx
return (
	<Route element={<AuthLayout />}>
		<Route index element={<Main />} />
		<Route path="/products" element={<Products />} />
	</Route>
    //...
```


---

# Data fetching

In molti casi, abbiamo bisogno che dei dati vengano inizializzati per la pagina e quindi il componente corrente, normalmente potremmo farlo usando uno `useEffect` all'interno del componente, ma potrebbe essere poco efficiente perché il componente deve prima essere creato e se possiede diversi sotto componenti che hanno bisogno di quel dato devono essere creati anche loro e avranno bisogno di aspettare che il dato arrivi.

Per questo si può fare in modo che `react-router` faccia il lavoro facendo il fetch del dato **PRIMA** di servire la pagina.
Infatti alla definizione della `route` possiamo utilizzare la prop `loader` per configurare il fetch dei dati (può anche essere sincrono). Il valore ritornato da questa funzione sarà quello che il componente si ritroverà al mount:

```tsx
//...
<Route element={<ProductPage />} path="/prodict/:id" 
loader={async () => {
	const response = await fetch('...');
	
	if(!response.ok) {
		
	} else {
		const resData = await response.json();
		return resData;
	}
}}
/>
```

`react-router` gestirà autonomamente le `Promise`, per cui saremo sempre certi che al componente arriverà il dato, non la `Promise`.
Poi nel componente potremo far uso del dato utilizzando `useLoaderData()` [hook](hook):

```tsx
import {useLoaderData} from 'react-router-dom';

//...
const data = useLoaderData();
```

>Le route che stanno sopra quella su cui è configurato il `loader` non potranno utilizzare il dato, mentre potranno usarlo tutti i suoi figli e nipoti.

Normalmente la funzione `loader` viene definita nello stesso file della pagina ed esportato per essere utilizzato dal file che definisce le route.

## Ritornare una Response

Il nostro `loader` può anche ritornare una `Response`, un oggetto messo a disposizione dal `browser`. Questo prende come primo argomento il valore che vogliamo ritornare (se custom object meglio ritornarlo come stringa) e come secondo argomento un oggetto che descrive la risposta (possiamo ad es. indicare lo status code):

```tsx
loader={async () => {
	const response = await fetch('...');
	
	if(!response.ok) {
		
	} else {
		const resData = await response.json();
		const res = new Response('any data', {status: 201}); // <==
		return res;
	}
}}
```

Il valore ritornato da `useLoaderData` sarà comunque il valore della risposta (es. 'any data').
Considerando che `fetch` ritornerà un oggetto `Response`, questo vuol dire che sarà possibile ritornare direttamente l'oggetto ritornato dalla request, e sarà `react-router` a gestirlo per noi fornendoci il valore ritornato direttamente tramite `useLoaderData`:

```tsx
loader={async () => {
	const response = await fetch('...');
	
	if(!response.ok) {
		//...
	} else {
		return response;
	}
}}
```

## Che codice può andarci dentro?

Il loader viene eseguito nel browser, non nel server, quindi al suo interno possiamo eseguire qualsiasi codice vogliamo, anche utilizzare l'API del browser.
Quello che non possiamo fare è utilizzare i React [hook](hook).

## Error handling

All'interno del metodo `loader` non gestiamo gli errori come se fossimo in uno `useEffect`, quello che possiamo fare è restituire un `Response` o qualsiasi altro oggetto per descrivere che è avvenuto un errore:

```tsx
loader={async () => {
	const response = await fetch('...');
	
	if(!response.ok) {
		return { isError: true, message: "Could not fetch products!" } // <==
	} else {
		return response;
	}
}}
```

In questa maniera possiamo gestire la cosa all'interno del componente:

```tsx
//...
if(data.isError) {
	return <p>{ data.message }</p>
}
//...
```

Ma esiste un'altra alternativa, possiamo lanciare un errore e lasciare che sia `react-router` a gestire la cosa, infatti cercherà il più vicino `errorElement` configurato nelle route (l'error farà un bubble-up dal componente a risalire), ritornando il componente configurato. Infatti `errorElement` non serve solo a gestire il caso in cui l'utente dovesse richiedere una rotta non esistente.

```tsx
//...
loader={async () => {
	const response = await fetch('...');
	
	if(!response.ok) {
		throw { message: "Could not fetch products!" } // <==
	} else {
		return response;
	}
}}
//...
```

## Estrarre error data e Responses

In molti casi vorremmo sapere il tipo di errore che è stato lanciato, in modo da poter gestire la pagina di errore da mostrare. Come detto in precedenza, il metodo `loader` può anche ritornare un oggetto `Response`:

```tsx
//...
loader={async () => {
	const response = await fetch('...');
	
	if(!response.ok) {
		throw new Response(
			JSON.stringify({message: 'Could not fetch events'} // <==
		), { status: 500 });
	} else {
		return response;
	}
}}
//...
```

Per poter usare quell'errore nel componente, `react-router` mette a disposizione l'[hook](hook) `useRouteError` che ci metterà a disposizione l'oggetto rappresentante l'errore, sia che esso sia un `Response` o altro:

```tsx
import { useRouteError } from 'react-router-dom';

function ErrorPage() {
	const error = useRouteError();
	let title = 'An error occurred!';
	let message = 'Something went wrong';

	// in questo caso avendo un oggetto Response abbiamo la prop 'status'
	if( error.status === 500 ) {
		message = JSON.parse(error.data) .message;
	}

	if( error.status === 404 ) {
		title = 'Not found!';
		message = 'Could not find resource or page.';
	}

	return (
		<PageContent title={title}>
			<p>{ message }</p>
		</PageContent>
	)
}
```

>È comodo lanciare un oggetto `Response` perché così possiamo gestire i codici errore.

## loader & dynamic routes

Se la route per cui va fatto il pre-load è dinamica, bisogna fare in modo che il loader possa utilizzare il parametro della route.
Per questo in automatico al loader viene passato un oggetto che contiene i dati passati alla route:

```tsx
// COMPONENT (loader in relative component file)
//...
export async function eventLoader( { request, params } ) {

	const response = await fetch(`http://mysite.com/events/${ params.id }`);
	
	if(!response.ok) {
		throw new Response(
			JSON.stringify({message: 'Could not fetch event details'}
		), { status: 500 });
	} else {
		return response;
	}
}
```

## Accedere al loader data da altre rotte

Se abbiamo più pagine che hanno bisogno dello stesso dato fornito dal `loader`, possiamo metterli sotto la stessa route padre:

```tsx
<Route path=":eventId" loader={ eventLoader }>
	<Route index element={<EventDetails />} />
	<Route path="edit" element={<EventEdit />} />
</Route>
```

In questa maniera le sub-route potranno usare lo stesso dato ottenuto tramite il loader.

>Per poter utilizzare i dati nel componente però non andrà usato l'[[hook]] `useLoaderData`, perché questo cercherà il loader più vicino per la rotta che ha renderizzato il relativo componente, che non è la route padre (quella che detiene il loader). Non trovandolo sulla route restituirà errore.

Andrà aggiunto un `id` alla `route` padre e utilizzato un altro [hook](hook), `useRouteDataLoader` passandogli l'`id` assegnato al padre:

```tsx
<Route id='event-detail' path=":eventId" loader={ eventLoader }>
	<Route index element={<EventDetails />} />
	<Route path="edit" element={<EventEdit />} />
</Route>

// COMPONENTE
//...
const data = useRouteLoaderData('event-detail');
//...
```

---

# Data submit

Come è possibile fare il pre-load dei dati grazie alla prop `loader`, è anche possibile gestire l'invio di dati tramite:

- Proprietà `action`
- Programmaticamente

## action

Come per la prop `loader`, la prop `action` riceve una funzione (anche questa può essere dichiarata nel file del componente) che verrà eseguita nel browser.

```tsx
//...
<Route element={<ProductPage />} path="/prodict/:id" 
 action={async () => {
	const response = await fetch('...');
	
}}
/>
```


---

# Navigation state & UI

Spesso è necessario riflettere lo stato della navigazione sull'UI, ad es. se la pagina sta caricando ecc. Anche in questo caso è utile l'[[hook]] `useNavigation` e la sua proprietà `navigation.state: 'idle' | 'loading' | 'submitting'`.

```tsx
//..
{navigation.state === 'loading' && <div>Loading...</div>}
```