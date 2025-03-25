È importante animare un'applicativo web per la [UX](UX) e per farlo esistono diversi modi:

- Normale [CSS](CSS)
	- [Transitions](https://www.w3schools.com/css/css3_transitions.asp)
	- [Animations](https://www.w3schools.com/css/css3_animations.asp)
- [Framer Motion](https://framermotion.framer.website/): permette di creare animazioni mantenendo alte performance.

# Framer Motion

## Basics

Per animare in `Framer Motion`, basta importare l'utility `motion` nel componente che vogliamo animare e usarlo così:

```tsx title:App.jsx
import { motion } from "framer-motion";

function App() {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);
  const [rotate, setRotate] = useState(0);

  return (
    <div>
      <!-- È presente qualsiasi tag HTML -->
      <motion.div id="box" 
        animate={{ x, y, rotate }}
        transition={{ 
          duration: 0.3, <!-- durata animazione -->
          type: 'spring', <!-- tipo di animazione -->
          bounce: 1 <!-- effetto di rimbalzo -->
        }}
      /> 
    </div>
  );
}
```

- Tramite l'oggetto passato alla prop `animate` possiamo animare qualsiasi proprietà e stile dell'elemento.
- Tramite `transition` possiamo configurare come ciò che è in `animate` deve comportarsi (senza verranno usati dei default).

## Animare fra valori condizionali

```tsx
...
<button onClick={onViewDetails}>
  View Details{' '}
  <motion.span className="details-icon"
    animate={{ rotate: isExpanded ? 180 : 0 }}
  >&#9650;</motion.span>
</button>
```

## Animazioni d'entrata

In questo caso nell'es. sotto non abbiamo degli indizi da utilizzare per capire se la modal è visibile o meno. Ciò non importa perché `motion` mette a disposizione la prop `initial` che permette di settare dei comportamenti di base non appena l'elemento viene aggiunto alla [DOM](DOM) (lo state iniziale). Se lo stato iniziale è diverso da quello presente in `animate`, motion animerà l'elemento per portarlo allo state definitivo e desiderato (presente in `animate`):

```tsx title:MyModal.tsx
export default function Modal({ title, children, onClose }) {
  return createPortal(
    <>
      <div className="backdrop" onClick={onClose} />
      <motion.dialog open className="modal"
        initial={{ opacity: 0, y: 30 }}
        animate={{ opacity: 1, y: 0 }}
	  >
        <h2>{title}</h2>
        {children}
      </motion.dialog>
    </>,
    document.getElementById('modal');
  )
}
```

## Animazioni d'uscita (scomparsa)

Come abbiamo fatto per le animazioni d'entrata, possiamo utilizzare la stessa logica per quelle di uscita. Ma al posto di utilizzare la prop `initial`, utilizzeremo `exit`:

```tsx title:MyModal.tsx
export default function Modal({ title, children, onClose }) {
  return createPortal(
    <>
      <div className="backdrop" onClick={onClose} />
      <motion.dialog open className="modal"
        initial={{ opacity: 0, y: 30 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: 30 }}
	  >
        <h2>{title}</h2>
        {children}
      </motion.dialog>
    </>,
    document.getElementById('modal');
  )
}
```

>Esiste però un problema, se la presenza di un componente che contiene motion viene gestita da una logica condizionale, React rimuoverà l'elemento senza chiedere, non facendo scatenare l'animazione di uscita.

Per fortuna per risolvere il problema `motion` ci mette a disposizione un componente speciale che wrappando il componente in questione ne gestirà il comportamento di scomparsa:

```tsx title:Header.tsx
import { AnimatePresence } from 'framer-motion';

...
return (
  <>
    <AnimatePresence>
      { isVisible && <MyModal /> }
    </AnimatePresence>
  </>
)
```

In questa maniera si assicurerà che l'animazione definita dall'`exit` venga eseguita prima che il componente venga rimosso dalla [DOM](DOM).

## Animare all'hover

```tsx
...
<motion.button
  whileHover={{ scale: 1.1 }}
  transition={{ type: 'spring', stiffness: 500 }} //<== Aggiunge il bounce
>
  Add Challenge
</motion.button>
```

## Riutilizzare gli animation state

Poiché alla fine stiamo passando degli oggetti alle prop di `motion`, potremmo anche metterli prima in una variabile e riutilizzarli.
Ma esiste un modo migliore per farlo, utilizzando la prop `variants`, che prende un oggetto in cui possiamo settare le key che preferiamo:

```tsx title:MyModal.tsx
export default function Modal({ title, children, onClose }) {
  return createPortal(
    <>
      <div className="backdrop" onClick={onClose} />
      <motion.dialog open className="modal"
        variants={{
          hidden: { opacity: 0, y: 30 }, // nomi key liberi
          visible: { opacity: 1, y: 0 }
        }}
        initial="hidden"
        animate="visible"
        exit="hidden"
	  >
        <h2>{title}</h2>
        {children}
      </motion.dialog>
    </>,
    document.getElementById('modal');
  )
}
```

## Variants

Come abbiamo visto `variants` possono essere utilizzate per riutilizzare gli animation state, ma non solo.

>Possono essere usati anche per triggerare animazioni in elementi profondamente innestati, lungo l'albero degli elementi del [DOM](DOM), semplicemente settando un'animazione ad una certa `variant` presente sul componente antenato.

```tsx
// MyModal.tsx

export default function Modal({ title, children, onClose }) {
  return createPortal(
    <>
      <div className="backdrop" onClick={onClose} />
      <motion.dialog open className="modal"
        variants={{
          hidden: { opacity: 0, y: 30 },
          visible: { opacity: 1, y: 0 }
        }}
        // Questi attiveranno la config delle varianti anche per i child
        initial="hidden"
        animate="visible"
        exit="hidden"
	  >
        <h2>{title}</h2>
        {children}    // Tutti i children potranno definire la propria variante
      </motion.dialog>
    </>,
    document.getElementById('modal');
  )
}

// MyComponent.tsx
...
<motion.li
  variants={{
    // La propria config per le varianti attivate all'initial ecc. da Modal
    hidden: { opacity: 0, scale: 0.5 },
    visible: { opacity: 1, scale: 1 }
  }} 
  transition={{ type: "spring" }}
  // Non c'è bisogno di settare initial, animate e exit, perchè è già stato definito da Modal 
  >
  ...
<motion.li>
...

// App.tsx

...
<Modal>
  <MyComponent />
</Modal>

```

>Bisogna però stare attenti al fatto che all'exit, ad es. chiusura della modal, i componenti condivideranno l'attivazione della propria versione della variante `exit`, quindi ad es. se la modal viene chiusa, gli elementi al suo interno dovranno prima finire la loro animazione di exit prima che scompaiano dallo schermo, potenzialmente creando effetti visivi strani.

Uno dei modi per risolvere il problema è fare l'override della prop specifica, in questo caso `exit`:

```tsx title:MyComponent.tsx
...
<motion.li
  variants={{
    hidden: { opacity: 0, scale: 0.5 },
    visible: { opacity: 1, scale: 1 }
  }} 
  transition={{ type: "spring" }}
  // Se riutilizzo "visible" come valore, perderò anche le altre anim.
  exit={{ opacity: 1, scale: 1 }} // Non cambiando niente non partirà l'animazione
  >
  ...
<motion.li>
...
```

## Animare liste

Per quanto riguarda le liste, vorremmo che l'animazione non fosse un'unica che faccia comparire tutti gli elementi, ma che avvenga lo `staggering`, cioè che vengano animate in ordine uno per uno gli elementi della lista.

Per farlo dobbiamo usare `motion` nel componente padre, quindi l'elemento `ul`, ma utilizzeremo la prop speciale `transition`, utilizzabile anche al di fuori della prop `variants` (ad es. anche solo alla prop `exit`), e che permette di passare lo stesso oggetto che normalmente viene passato alla prop `transition` che abbiamo visto prima.

Nella nostra `variant` potremo in questo caso utilizzare l'opzione `staggerChildren`, che permette di definire il delay fra elementi prima che inizi la propria animazione:

```tsx
...
<motion.ul
  variants={{ 
    visible: { transition: { staggerChildren: 0.05 } }
  }}
>
  {images.map(image => (
    <motion.li
      variants={{
        hidden: { opacity: 0, scale: 0.5 },
        visible: { opacity: 1, scale: 1 }
      }} 
      transition={{ type: "spring" }}
      exit={{ opacity: 1, scale: 1 }}
    >
      ...
    <motion.li>
  ))}
</motion.ul>
```

## Animare colori e keyframe

Come per le altre proprietà come `scale`, `x` ecc., è anche possibile animare un cambio di colori:

```tsx
...
<motion.button
  whileHover={{ scale: 1.1, backgroundColor: '#8b11f0' }} // <===
  transition={{ type: 'spring', stiffness: 500 }}
  >
  Add Challenge
</motion.button>
```

Anche in questo caso verranno applicate le caratteristiche definite in `transition`.

Altra cosa da sapere è che alle varie prop degli oggetti passati, possiamo passare un'array, in quel caso gli elementi al suo interno verranno interpretati come `keyframe`:

```tsx
...
<motion.li
  variants={{
    hidden: { opacity: 0, scale: 0.5 },
    visible: { opacity: 1, scale: [0.8, 1.3, 1] } // <===
  }} 
  transition={{ type: "spring" }}
  ...
  >
  ...
<motion.li>
```

## Imperative Animation

Ci sono circostanze in cui vorremmo iniziare un'animazione in maniera imperative piuttosto che dichiarativa.
Ad es. se un utente inserisce dei valori non validi nel nostro form, vorremmo che questi venissero animati al submit. Per fare ciò possiamo usare un [[hook]] di `motion`, `useAnimate`, che possiamo chiamare all'interno della logica del nostro componente. questo restituirà 2 elementi:
- `scope`: è una `ref` che usiamo sugli elementi
- `animate`: è la funzione per triggerare l'animazione

### animate

Accetta 3 argomenti, il primo definisce gli elementi da animare (classi, html tag ecc.), il secondo definisce il comportamento che vogliamo triggerare, il terzo è l'equivalente della prop `transition`:

```tsx
import { motion, useAnimate, stagger } from 'framer-motion';

export default function MyComponent() {
  const [scope, animate] = useAnimate();

  if(isFormInvalid) {
    // si passano gli elementi da animare per tag o classe ecc.
    animate(
      "input, textarea", 
      { x: [-10, 0, 10, 0] },
      // stagger è una func per fare l'effetto stagger imperativamente
      // in modo che non vengano animati tutti allo stesso momento
      { type: "spring", duration: 0.2, delay: stagger(0.5) }
    );
    return;
  }
}
```

### scope

Serve a definire l'area di selezione per l'animazione, in modo che non siano ad es. tutti i campi `input` e `textarea` della pagina ad essere coinvolti, ma solo quelli all'interno dell'area definita.

```tsx
...
// Solo gli elementi al suo interno verranno animati
<form ... ref={scope}>
  ...
</form>
```

## Animare cambiamenti nel layout

Per gestire dei cambiamenti nel `layout`, ad es. la rimozione di un elemento da una lista che provocherebbe lo spostamento repentino degli altri rimanenti, possiamo utilizzare la prop `layout` per fare in modo che lo spostamento venga animato piacevolmente:

```tsx
...
<motion.li layout>
  ...
</motion.li>
```