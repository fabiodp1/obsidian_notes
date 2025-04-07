Esistono diversi metodi in [[Next.js]] per migliorare l'[[accessibilità]]:
- Utilizzare il plugin di [[ESLint]] [eslint-plugin-jsx-a11y](https://www.npmjs.com/package/eslint-plugin-jsx-a11y) incluso nella configurazione di [[ESLint]]. Ad esempio ci avverte se ci sono immagini senza `alt`, se vengono utilizzati non in maniera corretta gli attributi `aria-*` e `role`. Una volta configurato basta lanciare il comando `next lint` per fare un'analisi del codice:

```json
"scripts": {
    "build": "next build",
    "dev": "next dev",
    "start": "next start",
    "lint": "next lint"
},
```

- Migliorare l'accessibilità dei `form` utilizzando ad esempio:
	- `Semantic HTML`: utilizzando gli elementi di semantica (`<input>`, `<option>` ecc.) al posto del semplice `<div>`.
	- `Labelling`: includere il `<label>` e l'attributo `htmlFor` assicura che ogni campo del form abbia una label descrittiva.
	- `Focus Outline`: I campi sono correttamente stilizzati per mostrare una linea di contorno nel momento in cui sono in focus.
- [[client-side]] validation: una delle maniere più facili sarebbe di utilizzare la validazione form fornita dal browser, aggiungendo ad es. l'attributo `required` agli elementi di input e select dei form:

```jsx
<input
  id="amount"
  name="amount"
  type="number"
  placeholder="Enter USD amount"
  className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
  required
/>
```

- [[server-side]] validation, con [[Zod]] possiamo settare dei messaggi in caso i campi non dovessero passare la validazione e rendendo il componente interessato un [[Client Component]] possiamo utilizzare l'[[hook]] [[useActionState]]:

```tsx
'use client';
// ...
import { useActionState } from 'react';
 
export default function Form({ customers }: { customers: CustomerField[] }) {
  const initialState: State = { message: null, errors: {} };
  const [state, formAction] = useActionState(createInvoice, initialState);
 
  return <form action={formAction}>...</form>;
}
```

La [[server action]] diventerà:

```ts
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string({
    invalid_type_error: 'Please select a customer.',
  }),
  amount: z.coerce
    .number()
    .gt(0, { message: 'Please enter an amount greater than $0.' }),
  status: z.enum(['pending', 'paid'], {
    invalid_type_error: 'Please select an invoice status.',
  }),
  date: z.string(),
});

export type State = {
  errors?: {
    customerId?: string[];
    amount?: string[];
    status?: string[];
  };
  message?: string | null;
};
 
export async function createInvoice(prevState: State, formData: FormData) {
  // Validate form using Zod
  const validatedFields = CreateInvoice.safeParse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
 
  // If form validation fails, return errors early. Otherwise, continue.
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Missing Fields. Failed to Create Invoice.',
    };
  }
 
  // Prepare data for insertion into the database
  const { customerId, amount, status } = validatedFields.data;
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];
 
  // Insert data into the database
  try {
    await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
  } catch (error) {
    // If a database error occurs, return a more specific error.
    return {
      message: 'Database Error: Failed to Create Invoice.',
    };
  }
 
  // Revalidate the cache for the invoices page and redirect the user.
  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```

`prevState` contiene lo state passato dall'hook [[useActionState]].
[[safeParse]]() ritornerà un oggetto contenente un campo `success` o `error`.  Questo permetterà la gestione della validazione in maniera più elegante, senza dover mettere la logica dentro un [[try/catch]].