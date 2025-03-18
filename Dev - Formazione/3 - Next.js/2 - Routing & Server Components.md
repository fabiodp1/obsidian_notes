La cartella `app` è quella più importante, qui verranno definite le pagine del nostro sito web e le sue `route`.
Al suo interno troviamo 2 file con 2 nomi riservati: 

- `page.ts`: dice a [Next.js](Next.js) che tramite il suo contenuto dovrà renderizzare una pagina.
	- Al suo interno si trova un **componente React**, che in questo caso viene definito un `Server Component`, un componente che richiede un "environment" speciale fornito da Next. **Viene eseguito e renderizzato sul server**, mai lato client.
- `layout.ts`
