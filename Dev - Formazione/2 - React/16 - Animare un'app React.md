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