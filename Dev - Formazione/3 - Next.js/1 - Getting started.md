.# Creare un nuovo progetto
Viene utilizzato il comando [[create-next-app]] :

```terminal
npx create-next-app@latest <my-app-name> --use-pnpm
```

> [[pnpm]] è un'alternativa preferibile a [[npm]], più veloce e gestisce meglio memoria e moduli.

## Struttura progetto

- `/app`: contiene tutte le route, componenti e la logica dell'applicativo. Dove si lavorerà per la stra grande maggioranza del tempo.
- `/app/lib`: contiene le funzioni utilizzate nell'applicativo, come utility condivise e funzioni per il fetch dei dati e le type definitions.
- `/app/ui`: contiene tutti i componenti della UI, come card, tabelle, form ecc.
- `/public`: contiene tutti gli asset statici come ad es. le immagini.
- **File di config**: nella route del progetto sono presenti file come `next.config.js`.
  Molti di questi file vengono creati e pre-configurati allo scaffolding di un nuovo progetto utilizzando [[create-next-app]].

## Placeholder data

quando si sta lavorando ad un'interfaccia e la parte di db o api non è ancora pronta, può essere utile creare dei dati placeholder sotto forma di costanti JSON che verranno ritornati da librerie di mock come ad es. [[mockAPI]] (https://mockapi.io/).