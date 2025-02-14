Oltre al metodo tradizionale per gestire gli errori tramite [[try/catch]], [[Next.js]] mette a disposizione il file speciale [[error.tsx]] che serve da [[catch-all]] per gli errori che potrebbero capitare inaspettatamente e poterli gestire in una maniera standard:

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
    <main className="flex h-full flex-col items-center justify-center">
      <h2 className="text-center">Something went wrong!</h2>
      <button
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
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
- `'use client'`: [[error.tsx]] deve essere un [[Client Component]].
- accetta 2 prop:
	- `error`: istanza dell'oggetto JS nativo `Error`.
	- `reset`: una funzione per fare il reset del limitatore degli errori. Quando eseguita, la funzione cercherà di re-renderizzare il [[route segment]].
# Gestire gli errori 404 con [[notFound]]
Un altro modo per gestire gli errori è utilizzare la funzione [[notFound]].
Se [[error.tsx]] è utile per fare il catch degli errori, [[notFound]] può essere usato quando si cerca di fare il fetch di una risorsa che non esiste:

```tsx
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

Per mostrare una UI di errore all'utente, possiamo usare il file speciale [[not-found.tsx]].

```tsx
// not-found.tsx

import Link from 'next/link';
import { FaceFrownIcon } from '@heroicons/react/24/outline';
 
export default function NotFound() {
  return (
    <main className="flex h-full flex-col items-center justify-center gap-2">
      <FaceFrownIcon className="w-10 text-gray-400" />
      <h2 className="text-xl font-semibold">404 Not Found</h2>
      <p>Could not find the requested invoice.</p>
      <Link
        href="/dashboard/invoices"
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
      >
        Go Back
      </Link>
    </main>
  );
}
```

>[[notFound]] avrà precedenza su [[error.tsx]].

# Altre risorse per approfondire

- [Error Handling](https://nextjs.org/docs/app/building-your-application/routing/error-handling)
- [`error.js` API Reference](https://nextjs.org/docs/app/api-reference/file-conventions/error)
- [`notFound()` API Reference](https://nextjs.org/docs/app/api-reference/functions/not-found)
- [`not-found.js` API Reference](https://nextjs.org/docs/app/api-reference/file-conventions/not-found)
