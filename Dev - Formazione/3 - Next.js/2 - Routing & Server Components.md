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