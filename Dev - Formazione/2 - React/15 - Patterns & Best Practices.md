[Your React App Isn’t Great — Here’s Why. | by Pierre G. | Dec, 2024 | Level Up Coding](https://levelup.gitconnected.com/your-react-app-isnt-great-here-s-why-5eb61b3f110b)

# Code splitting
[Code-Splitting – React](https://legacy.reactjs.org/docs/code-splitting.html)

Per ottimizzare le performance è importante che fra componenti le callback vengano passate utilizzando l'hook `useCallback`, in modo da evitare re-render inutili.
Inoltre se il nostro componente è un context provider, è buona pratica passare come value la versione memoized:

```ts
function AuthProvider(props) {
  const {
    data: user,
    error,
    ...
  } = useAsync()

  const login = React.useCallback(
    form => auth.login(form).then(user => setData(user)),
    [setData],
  )

  const register = React.useCallback(
    form => auth.register(form).then(user => setData(user)),
    [setData],
  )

  const logout = React.useCallback(() => {
    auth.logout()
    setData(null)
  }, [setData])

  const value = React.useMemo(() => ({user, login, logout, register}), [
    login,
    logout,
    register,
    user,
  ])

  if (isLoading || isIdle) {
    return <FullPageSpinner />
  }

  if (isError) {
    return <FullPageErrorFallback error={error} />
  }

  if (isSuccess) {
    return <AuthContext.Provider value={value} {...props} />
  }

  throw new Error(`Unhandled status: ${status}`)
}
```

---

# Compound components

I [[compound components]] sono un modo in [[React]] per organizzare i componenti in modo da renderli flessibili e componibili fra loro.
Non sono pensati per essere `standalone` ma per funzionare uniti fra loro.

Ad es. se dovessimo creare un componente `Accordion`, normalmente potremmo crearne uno che contiene un `ul` e accetta una prop `items` su cui lui fa il `map` per mostrarne gli elementi.
Ma utilizzando il pattern del `compound component` possiamo fare in modo che questo sia più elastico e riutilizzabile:

```tsx title:Accordion.tsx
export default function Accordion({children, className }) {
  return <ul className={className}>{children}</ul>
}
```

In questa maniera diventa un guscio personalizzabile a cui basta passare gli elementi che vogliamo.
Questo però non basta, non è l'idea di questo pattern, creeremo altri `compound components` che con `Accordion` concorreranno a strutturare il nostro componente finale:

```tsx title:AccordionItem.tsx
export default function AccordionItem({className, title, children}) {
  return (
    <li className={className}>
      <h3>{ title }</h3>
      <div>{ children }</div>
    </li>
  )
}
```

In questa maniera possiamo comporre i nostri componenti così:

```tsx title:App.tsx
function App() {
  return(
    ...
    <Accordion className="accordion">
      <AccordionItem title="One">
        <article>
          <p>One</p>
        </article>
      </AccordionItem>
      <AccordionItem title="Two">
        <article>
          <p>Two</p>
        </article>
      </AccordionItem>
    </Accordion>
  )
}
```

## Gestire lo stato fra componenti

Poiché stiamo lavorando con diversi componenti che devono rimanere fortemente configurabili, non possiamo aggiungere lo stato in uno solo di questi (ad. es. quello principale), perderebbero la loro indipendenza.

Per farlo possiamo usare il [Context](6%20-%20Context.md) di React.:

```tsx title:Accordion.tsx
import {createContext} from 'react';

// Generalmente viene creato nel file del componente principale
const AccordionContext = createContext();

export function useAccordionContext() {
  const ctx = useContext(AccordionContext);

  if(!ctx) {
    throw new Error("Accordion-related components must be wrapped by <Accordion>.");
  }
}

export default function Accordion({ children, className }) {
  const [openItemId, setOpenItemId] = useState();

  function toggleItem(id) {
    setOpenItemId(prevId => prevId === id ? null : id);
  }

  const contextValue = {
    openItemId,
    toggleItem
  };

  return (
    <AccordionContext.Provider value={ contextValue }>
      <ul className={ className }>{ children }</ul>
    </AccordionContext.Provider>
  )
}
```

```tsx title:AccordionItem.tsx
export default function AccordionItem({id, className, title, children}) {
  const { openItemId, toggleItem } = useAccordionContext();

  const isOpen = openItemId === id;

  function handleClick() {
    toggleItem(id);
  }

  return (
    <li className={className}>
      <h3 onClick={handleClick}>{ title }</h3>
      <div className={ isOpen ? 'open' : undefined }>
        { children }
      </div>
    </li>
  )
}
```

## Raggruppare i compound componentes

È buona norma, per rendere più leggibile e ovvio che dei componenti fanno parte dello stesso gruppo, mettere i componenti in uno stesso oggetto, che solitamente sarà il componente principale (nell'es. `Accordion`).

>In [JS](JS) le funzioni non sono altro che oggetti, e i componenti [[React]] non sono altro che *funzioni*, quindi possiamo accedere o creare nuove prop.

```tsx title:Accordion.tsx
import {createContext} from 'react';
import {AccordionItem} from './AccordionItem.tsx'

// Generalmente viene creato nel file del componente principale
const AccordionContext = createContext();

export function useAccordionContext() {
  ...
}

export default function Accordion({ children, className }) {
  ...
  return (
    <AccordionContext.Provider value={ contextValue }>
      <ul className={ className }>{ children }</ul>
    </AccordionContext.Provider>
  )
}

Accordion.Item = AccordionItem;

```

In questa maniera possiamo usare il nostro `AccordionItem` in questa maniera:

```tsx title:App.tsx
// import AccordionItem from './AccordionItem.tsx' <== Non serve più
import Accordion from './Accordion.tsx';

function App() {
  return(
    ...
    <Accordion className="accordion">
      <Accordion.Item title="One">        <===
        <article>
          <p>One</p>
        </article>
      </Accordion.Item>
      <Accordion.Item title="Two">
        <article>
          <p>Two</p>
        </article>
      </Accordion.Item>
    </Accordion>
  )
}
```

Adesso è chiaro che il componente `Item` appartiene ad `Accordion`.
Ovviamente partendo da queste basi si potrebbe anche pensare di spezzare i componenti visti negli esempi in altri `compound components`, ad es. per il content dell'`Item`, il titolo ecc.

>Se abbiamo bisogno di condividere dello state solo con una parte del gruppo (ad es. sottocomponenti di `Item`), possiamo creare un context nel componente che deve condividere lo state, wrappando i suoi figli:

```tsx title:AccordionItem.tsx
const AccordionItemContext = createContext();

export function useAccordionItemContext() {\
  const ctx = useContext(AccordionItemContext);

  if(!ctx) {
    throw new Error("AccordionItem-related components should be wrapped by <Accordion.Item>")
  };

  return ctx;
}

export default function AccordionItem({id, className, title, children}) {
  ...
  return (
    <AccordionItemContext.Provider value={id}>
      <li className={className}>
        { children }
      </li>
    </AccordionItemContext.Provider>
  );
}
```