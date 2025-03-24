# Server Actions

Le `server action` di [[React]] permettono di lanciare codice asincrono direttamente sul server. Eliminano la necessità di creare endpoint [[API]] per cambiare i dati.
Basta scrivere delle funzioni asincrone che verranno eseguite sul server e possono essere invocate dal client o dai [[Server Component]].

Inoltre offrono una soluzione efficace per la sicurezza, proteggendo contro diversi tipi di attacco, mettendo al sicuro i dati e assicurando un accesso autorizzato. Le [[server action]] permettono questo attraverso tecniche come `POST` request, encrypted closure, strict input check, hashing dei messaggi di errore, restrizioni all'host, che tutti assieme collaborano per potenziare la sicurezza dell'applicativo.

>Le [[server action]] sono profondamente integrate con il sistema di [[caching]] di [[Next.js]].
>Quando un form viene confermato attraverso le [[server action]], non solo è possibile cambiare i dati, ma è anche possibile rivalidare la [[cache]] associata utilizzando [[API]] come [[revalidatePath]] e [[revalidateTag]].

## How

Per crearle basta inserire il tag `"use server"` all'interno del nostro handler, all'interno del `Server Component`, per trasformarlo in una `server action`:

```tsx title:my-form.tsx
// Server Component
export default function MyForm() {
  // Action
  async function handleSubmit(formData: FormData) {
    'use server';                                      // <===
 
    // Logic to mutate data...
  }
  //...
```

>Non è possibile usare una `server action` dichiarata all'interno di un `Client Component`.

Se vogliamo esportare tutte le nostre action da un file `.ts` per poi poterle riutilizzare sia su `Server` che `Client` components, basta mettere in cima al file il flag `"use server"`.
In questa maniera le funzioni esportate sono marcate come `server action` e possono essere usate sia da componenti Client che Server:

```tsx
// actions.ts
'use server'

export async function createInvoice(formData: FormData) {
	// ...
}

// create-form.tsx
import { createInvoice } from '@/app/lib/actions';

export default function Form({ customers }: { customers: CustomerField[] }) {
  return (
    <form action={createInvoice}>
    {...}
```

---

# Form

## Utilizzare i `form` con le [[server action]]

In [React 19+](React%2019+.md) è possibile utilizzare l'attributo [[action]] nell'elemento `<form>` per invocare le action. L'action riceverà automaticamente l'oggetto `FormData` nativo, contenente i dati catturati, come avviene in [React 19+](React%2019+.md) (vedi [12 - Forms](12%20-%20Forms.md) ):

```tsx title:my-form.tsx
// Server Component
export default function MyForm() {
  // Action
  async function create(formData: FormData) {
    'use server';
 
    // Logic to mutate data...
  }
 
  // Invoke the action using the "action" attribute
  return <form action={create}>...</form>;
}
```

> Normalmente all'attributo action verrebbe passato l'URL dell'endpoint dell'API di destinazione a cui inviare il form. In [[React]] invece l'attributo action è una prop speciale su cui React costruisce per dare la possibilità di invocarlo.
> Dietro le quinte la `server action` crea un endpoint [[API]] di POST. Questo è il motivo per cui non c'è bisogno di creare endpoint manualmente quando utilizziamo le `server action`.

Un vantaggio di invocare la `server action` è che essendo eseguito tutto sul server, i form funzionano anche se JS è disabilitato sul client.

## Estrarre i dati del form

Esistono diversi modi per farlo, uno di questi è l'utilizzo del metodo `.get(<inputName>)`.

```ts
'use server';
 
export async function createInvoice(formData: FormData) {
  const rawFormData = {
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  };

  // ...
}
```

>Nel caso in cui stiamo usando form con molti dati, conviene usare il metodo `entries()` con `Object.fromEntries()`:

```tsx
const rawFormData = Object.fromEntries(formData.entries())
```

## Validazione e preparazione dati

Prima di inviare i dati al database è necessario validare i dati per assicurarsi che siano nel formato corretto, con i type corretti e che non siano potenzialmente una minaccia alla sicurezza (es. [[XSS]] o [[DOS]]).

Per la validazione dati esistono diverse opzioni, fra cui l'utilizzo di librerie come  [Zod](https://zod.dev/):

```ts
'use server';
 
import { z } from 'zod';
 
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(['pending', 'paid']),
  date: z.string(),
});
 
const CreateInvoice = FormSchema.omit({ id: true, date: true });
 
export async function createInvoice(formData: FormData) {
  // ...
}
```

Nell'esempio il campo `amount` è settato per cambiare da stringa a numero oltre a validarne il tipo (nel form un campo input di tipo number darà comunque come valore una stringa, in questa maniera è possibile fare la conversione).

```ts
// ...
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
}
```

>**NOTA** per la validazione di stringhe di html da renderizare client-side, esiste la libreria `xss` che si assicura che l'[HTML](HTML.md) da parsare non sia malevolo.

---

# Persistere i dati

Per persistere i dati a [[DB]] basta creare una `query SQL` per fare l'inserimento:

```ts
import { sql } from '@vercel/postgres';

export async function createInvoice(formData: FormData) {
	// ...
	await sql` INSERT INTO invoices (customer_id, amount, status, date) VALUES (${customerId}, ${amountInCents}, ${status}, ${date}) `;
	// ...
```

>Ovviamente nell'esempio manca la gestione degli errori e **IMPORTANTE** i valori vanno parametrizzati, è pericoloso costruire le query con string interpolation o string concatenation.

>Le immagini caricate non vanno salvate a [DB](DB) ma sulla cartella `public` grazie all'API `fs`.

---

# Revalidate and redirect

[[Next.js]] ha un [[client-side]] [[router cache]] che mantiene nel browser dell'utente i segmenti della [[route]] per un certo periodo di tempo.
Assieme al [[prefetching]], questa [[cache]] si assicura che gli utenti possano muoversi fra rotte in maniera veloce, limitando al minimo le richieste fatte al server.

Se volessimo pulire questa cache, ad es. perchè vogliamo che i dati mostrati vengano aggiornati, possiamo utilizzare la funzione di Next.js [[revalidatePath]]:

```tsx
'use server';
 
import { z } from 'zod';
import { sql } from '@vercel/postgres';
import { revalidatePath } from 'next/cache';
 
// ...
 
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];
 
  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
 
  revalidatePath('/dashboard/invoices');
}
```

Nell'es. una volta che il db è stato aggiornato, la path `/dashboard/invoices` sarà rivalidata e dati freschi verranno fetchati dal server.
Una volta fatto ciò possiamo fare il redirect alla pagina precedente, utilizzando la funzione [[redirect]]:

```tsx
'use server';
 
import { z } from 'zod';
import { sql } from '@vercel/postgres';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
 
// ...
export async function createInvoice(formData: FormData) {
  // ...
  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```

---

# Rotte dinamiche

[[Next.js]] permette la creazione di [[dynamic route]] segment, ad es. quando ci serve una rotta che contenga un id o un endpoint creato dinamicamente in base ai dati posseduti.
Per fare ciò basta creare un un folder wrappandone il nome con parentesi quadre:

![[Pasted image 20241217142847.png]]

## Leggere i parametri della route

Oltre a [[searchParams]], i componenti della pagina accettano anche un prop chiamata [[params]] che può essere utilizzata per leggere i parametri della [[route]]:

```tsx
import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchInvoiceById, fetchCustomers } from '@/app/lib/data';
 
export default async function Page(props: { params: Promise<{ id: string }> }) {
  const params = await props.params;
  const id = params.id;
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);
  // ...
}
```

>Nel momento in cui dobbiamo aggiornare il valore del modello ricevuto, invece di semplicemente chiamare il nostro metodo passando il valore, possiamo assicurarci che il parametro passato al [[server action]] sia #codificato utilizzando [[bind]]:

```tsx
// ...
import { updateInvoice } from '@/app/lib/actions';
 
export default function EditInvoiceForm({
  invoice,
  customers,
}: {
  invoice: InvoiceForm;
  customers: CustomerField[];
}) {
  const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
 
  return <form action={updateInvoiceWithId}></form>;
}
```

>Nel caso di un delete basta ad es. wrappare il pulsante per il #delete in un `<form>` tag:

```tsx
import { deleteInvoice } from '@/app/lib/actions';
 
// ...
 
export function DeleteInvoice({ id }: { id: string }) {
  const deleteInvoiceWithId = deleteInvoice.bind(null, id);
 
  return (
    <form action={deleteInvoiceWithId}>
      <button type="submit" className="rounded-md border p-2 hover:bg-gray-100">
        <span className="sr-only">Delete</span>
        <TrashIcon className="w-4" />
      </button>
    </form>
  );
}
```

> #security 
>[How to Think About Security in Next.js | Next.js](https://nextjs.org/blog/security-nextjs-server-components-actions)