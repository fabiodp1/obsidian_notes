Oltre al metodo tradizionale per gestire gli errori tramite [[try/catch]], [[Next.js]] mette a disposizione il file speciale `error.tsx` che serve da `catch-all` per gli errori che potrebbero capitare inaspettatamente e poterli gestire in una maniera standard:

```tsx
'use client';
 
import { useEffect } from 'react';
 
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Optionally log the error to an error reporting service
    console.error(error);
  }, [error]);
 
  return (
    <main>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the invoices route
          () => reset()
        }
      >
        Try again
      </button>
    </main>
  );
}
```

Dall'esempio si possono capire diverse cose:

- `'use client'`: `error.tsx` deve essere un [[Client Component]]. Infatti non permette solo di fare il catch degli errori lato server, ma anche client.
- accetta 2 prop:
	- `error`: istanza dell'oggetto JS nativo `Error`.
	- `reset`: una funzione per fare il reset del limitatore degli errori. Quando eseguita, la funzione cercherà di re-renderizzare il [[route segment]].

---

# Gestire gli errori 404

## `notFound`

Un altro modo per gestire gli errori è utilizzare la funzione `notFound` fornita da `next/navigation`. `notFound` può essere usato quando si vuole lanciare lo specifico errore `404`:

```tsx title:page.tsx
import { fetchInvoiceById, fetchCustomers } from '@/app/lib/data';
import { updateInvoice } from '@/app/lib/actions';
import { notFound } from 'next/navigation';
 
export default async function Page(props: { params: Promise<{ id: string }> }) {
  const params = await props.params;
  const id = params.id;
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);
 
  if (!invoice) {
    notFound();
  }
 
  // ...
}
```

## `not-found.tsx`

Se `error.tsx` è utile per fare il catch-all degli errori, e mostrare una UI di errore all'utente, possiamo usare il file speciale `not-found.tsx` per fare la stessa cosa ma per l'errore `404`.

```tsx title:not-found.tsx
import Link from 'next/link';
import { FaceFrownIcon } from '@heroicons/react/24/outline';
 
export default function NotFound() {
  return (
    <main>
      <h2>404 Not Found</h2>
      <p>Could not find the requested invoice.</p>
      <Link
        href="/dashboard/invoices"
      >
        Go Back
      </Link>
    </main>
  );
}
```

>`notFound` avrà precedenza su `error.tsx`.

---

# Altre risorse per approfondire

- [Error Handling](https://nextjs.org/docs/app/building-your-application/routing/error-handling)
- [`error.js` API Reference](https://nextjs.org/docs/app/api-reference/file-conventions/error)
- [`notFound()` API Reference](https://nextjs.org/docs/app/api-reference/functions/not-found)
- [`not-found.js` API Reference](https://nextjs.org/docs/app/api-reference/file-conventions/not-found)