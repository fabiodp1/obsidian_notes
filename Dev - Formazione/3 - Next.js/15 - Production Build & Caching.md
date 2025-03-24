Al build [Next.js](Next.js) pre-genera tutte le pagine statiche che possono essere create, quindi tutte le pagine non dinamiche.
Così che quando un utente richiede una pagina, la risposta sarà immediata, creando una grande esperienza utente. Inoltre si occuperà anche di fare il cache `client-side` di quelle pagine.

>L'unico problema è che così facendo, pagine che mostrano dati di cui è stato fatto il pre-fetch, non verranno aggiornate una volta fatto il `cache`.

---

# Cache Revalidation

Per risolvere il problema descritto sopra, bisogna dire a [Next.js](Next.js) di eliminare la `cache` riguardante quella pagina, in modo a poter avere quella aggiornata.

Per fare ciò esiste un metodo `built-in`, `revalidatePath` che prende come argomento il `path` alla pagina da rivalidare:

```ts title:actions.ts
...
await saveMeal(meal);

revalidatePath('/meals');    // <===

redirect('/meals');
```

Il metodo di default NON farà il revalidate anche delle `sub-route`, per farlo dovremo chiamarlo anche indicando la sub-route oppure passando un secondo parametro e indicando (con il valore `layout`) che vogliamo che invalidi anche le `route` innestate:

```ts title:actions.ts
...
await saveMeal(meal);

revalidatePath('/meals', 'layout');    // di default è 'page'

redirect('/meals');
```