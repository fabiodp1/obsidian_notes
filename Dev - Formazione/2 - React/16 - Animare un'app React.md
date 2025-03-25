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