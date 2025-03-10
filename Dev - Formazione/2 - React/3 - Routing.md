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
		<Route path="/cover" element={<BookCover />} />
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
	<Route path="login" element={<Login />} />
	<Route path="register" element={<Register />} />
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
# useParams
Nel momento in cui un componente ha bisogno di usare un parametro della route (es. BookScreen dell'es. precedente), possiamo usare l'hook [[useParams]]:

```ts
import {useParams} from 'react-router-dom'

function BookScreen({user}) {
	...
	const {bookId} = useParams()
	...
}
```

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

# Link
Se vogliamo creare dei link alle pagine, ad es. in una barra di navigazione, utilizziamo il componente `<Link />`:

```ts
import { Link } from 'react-router-dom'

<Link to="/discover">Discover</Link>
```

Un'alternativa al componente `Link` è il componente **`<NavLink />`**, che funziona alla stessa maniera ma funziona un po diversamente, infatti le classi possono essere passate fornendo una funzione che avrà come parametro un oggetto con diverse proprietà fra cui `isActive`, per assegnare classi in base allo stato del link:

```tsx
//...
<NavLink to="/" className={({ isActive }) => (isActive ? '...' : '...')}>
	Home
</NavLink>
//...
```



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