# Routing innestato

[[Next.js]] utilizza il [[file-system routing]] in cui le cartelle vengono usate per creare rotte innestate. Ogni folder rappresenta un segmento della [[route]] che mappa ad un segmento dell'[[URL]].

![[Pasted image 20241209161745.png]]

> [[page.tsx]] è un file speciale [[Next.js]] che esporta un componente [[React]] ed è richiesto dalla [[route]] per essere accessibile ed essere riconosciuta da Next come tale.

```tsx
export default function Page() {
  return <p>Dashboard Page</p>;
}
```

Per creare una [[nested route]], si possono innestare le cartelle una nell'altra avendo ognuna una [[page.tsx]]:

![[Pasted image 20241209162837.png]]

Avendo un nome riservato per i file `page.tsx`, [[Next.js]] permette di collocare (`colocate`) i componenti UI, test e altro codice con la rotta di riferimento.

>Solo il contenuto del `page.tsx` file sarà pubblicamente accessibile. Per esempio le cartelle `/ui` e `/lib` sono *collocate* all'interno della cartella `/app` insieme alle altre rotte.

---

# Dynamic Routes

Se ad es. abbiamo un'applicativo che contiene una `route` blog, da questa pagina vorremo anche poter accedere ai singoli `post`, ma come possiamo definire l'endpoint per il post-1, o il post-2 ecc.? Non possiamo di sicuro creare una cartella per ogni nuovo post creato.

In questo caso abbiamo bisogno di rotte dinamiche, definite una volta sola, ma capaci di mostrare il contenuto in maniera dinamica.

>In [Next.js](Next.js) creiamo rotte dinamiche creando una cartella nominata con dell parentesi quadre `[]` definendo all'interno delle parentesi quello che vogliamo.

```
/app
  |__/blog
        |__page.tsx
        |__[id]
              |__page.tsx
```

```tsx title:/app/blog/page.tsx
import Link from 'next/link';

export default function BlogPage() {
  return (
    <main>
      <h1>The Blog</h1>
      <p><Link href="/blog/post-1">Post 1</Link></p>
      <p><Link href="/blog/post-2">Post 2</Link></p>
    </main>
  )
}
```

```tsx title:/app/blog/[id]/page.tsx
import Link from 'next/link';

export default function BlogPostPage( { params } ) {
  
  return (
    <main>
      <h1>Blog Post {params.id}</h1>
    </main>
  )
}
```

>Per poter accedere ai parametri della `route`, [Next.js](Next.js) passerà automaticamente delle prop al nostro componente che potremo utilizzare per accedervi, non ci sarà bisogno di farlo manualmente.

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

