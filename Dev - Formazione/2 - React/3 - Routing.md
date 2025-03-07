 Normalmente viene utilizzato [[react-router]]:
[Docs | React Router](https://reactrouter.com/en/main)

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
      <Route path="/book/:bookId" element={<BookScreen user={user} />} />
      <Route path="*" element={<NotFoundScreen />} />
    </Routes>
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
Se vogliamo creare dei link alle pagine, ad es. in una barra di navigazione, utilizziamo il componente [[Link]]:

```ts
import { Link } from 'react-router-dom'

<Link to="/discover">Discover</Link>
```