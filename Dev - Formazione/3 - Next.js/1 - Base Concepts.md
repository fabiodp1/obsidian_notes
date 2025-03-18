# Creare un nuovo progetto

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

---

# Routing & Server Components

La cartella `app` è quella più importante, qui verranno definite le pagine del nostro sito web e le sue `route`.
Al suo interno troviamo 2 file con 2 nomi riservati: 

- `page.ts`
	- Dice a [Next.js](Next.js) che tramite il suo contenuto dovrà renderizzare una pagina.
	- Al suo interno si trova un **componente React**, che in questo caso viene definito un `Server Component`, un componente che richiede un "environment" speciale fornito da Next. **Viene eseguito e renderizzato sul server**, mai lato client.
- `layout.ts`

## Routing

[Next.js](Next.js) utilizzerà la struttura di cartelle definita all'interno della cartella `app` per definire l'albero delle `route` dell'applicativo.

Ad esempio se vogliamo aggiungere una nuova pagina e quindi una nuova route, dobbiamo:

1. creare una cartella con il nome dell'endpoint
2. creare al suo interno il file `page.ts` che conterrà il `Server Component`.

```
/app
 |__page.ts
 |__layout.ts
 |__/about
	   |__page.ts
```

```tsx
//page.ts

// Il nome non è importante
export default function AboutPage() {
	return <h1>About Us</h1>
}
```
