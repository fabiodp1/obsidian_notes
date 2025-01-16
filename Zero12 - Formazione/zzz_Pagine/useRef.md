[[useRef]] non serve solo ad accedere agli elementi del [[DOM]], ma anche a gestire lo state del componente che non va mostrato nella [[UI]], e quindi non ha bisogno che venga scatenata una riesecuzione del componente, ma ovviamente va gestito manualmente, non è ne uno state #derived ne #computed .

```tsx
// In questo caso usiamo useRef per tenere traccia di dei valori senza che vengano azzerati alla riesecuzione del componente, e li usiamo con l'unico scopo di computarne il timeLeft, non vengono utilizzati direttamente nella UI
const timer = useRef();
const startingTime = useRef();
const endingTime = useRef();
const timeLeft = endingTime.current * 1000 - endingTime.current * 1000;
//...
```

## Come fare il forward verso [[custom component]]
Per passare una ref ad un [[custom component]] figlio