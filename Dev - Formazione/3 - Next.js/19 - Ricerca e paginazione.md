[Learn Next.js: Adding Search and Pagination | Next.js](https://nextjs.org/learn/dashboard-app/adding-search-and-pagination)
# Perché i parametri di ricerca dell'[[URL]]
Ci sono diversi benefici nell'implementare la ricerca con i parametri [[URL]]:
- **URL condivisibili e salvabili**
- **SSR e caricamento iniziale**: i parametri dell'[[URL]] possono essere consumati direttamente dal server per renderizzare lo stato iniziale, rendendo più semplice il rendering [[server-side]].
- **Analytics e Tracking**: avere search query e filtri direttamente sull'[[URL]] rende più facile tenere traccia del comportamento utente senza bisogno di logica [[client-side]].

# Ricerca
- [[useSearchParams]] : permette di accedere ai parametri dell'[[URL]] corrente:

```ts
// /dashboard/invoices?page=1&query=pending
useSearchParams() // => {page: '1', query: 'pending'}
```

- [[usePathname]] : permette di leggere il pathname dell'[[URL]] corrente:

```ts
// /dashboard/invoices
usePathname() // => '/dashboard/invoices'
```

- [[useRouter]] : permette in maniera programmatica la navigazione fra rotte con componenti client ([Functions: useRouter | Next.js](https://nextjs.org/docs/app/api-reference/functions/use-router#userouter))

```tsx
'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { usePathname, useRouter, useSearchParams } from 'next/navigation';

export default function Search({ placeholder }: { placeholder: string }) {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const {replace} = useRouter();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if(term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }

    replace(`${pathname}?${params.toString()}`)
  }
  // ...
}
```

## Mantenere l'URL e input in sync
Per assicurarsi che il campo input e l'[[URL]] siano in sync si può passare un `defaultValue` al campo input:

```tsx
<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
  defaultValue={searchParams.get('query')?.toString()}
/>
```
### `defaultValue` vs. `value` / Controlled vs. Uncontrolled
Se viene usato lo [[state]] per gestire il valore di un input, allora verrà usato l'attributo `value` per renderlo un componente controllato. Ciò vuol dire che [[React]] gestirebbe lo stato dell'input.

Nel caso sopra non viene usato lo state, per cui si può usare `defaultValue`. Ciò vuol dire che il campo input gestirà il suo stato nativamente. Questo perché in quel caso il search query viene salvato nell'[[URL]] e non nello state.

>**Quando usare `useSearchParams` vs. `searchParams`**
>Come si può vedere dall'esempio abbiamo 2 modi diversi per estrarre il parametri di ricerca. Per decidere se usare uno o l'altro dipende dal fatto se stiamo lavorando sulla parte client o server.
>Se a doverli utilizzare è un [[Client Component]] usare [[useSearchParams]],  se è un [[Server Component]] [[searchParams]].

## Debouncing
Nell'esempio precedente, se facciamo una ricerca sul campo di input, la query e quindi chiamata per il dato, verrà triggerata ad ogni lettera inserita. Questo potrebbe essere un problema nel caso di applicativi più pesanti.

Il [[debouncing]] è una pratica di programmazione che limita la frequenza con cui una funzione viene triggerata. Nel caso precedente vorremmo che la chiamata parta solo quando l'utente ha finito di digitare.

>**Come funziona**
>1. **Trigger event**: quando l'evento che dovrebbe subire il debounce viene triggerato, parte un timer.
>2. **Wait**: se avviene un altro evento prima che il timer scada, il timer viene resettato.
>3. **Execution**: se il timer raggiunge la fine del countdown, la funzione viene eseguita.

E' possibile implementare il [[debouncing]] in diversi modi, fra cui crearlo manualmente. Il metodo più semplice è utilizzare librerie apposite come [[use-debounce]] (https://www.npmjs.com/package/use-debounce).
