# Global styles

Normalmente nella cartella `/app` è presente il file `globals.css`. E' possibile usare questo file per aggiungere regole CSS a tutte le [[route]] dell'applicativo.
E' possibile importare questo file in qualsiasi componente, ma è buona pratica aggiungerlo al **top level component** che in [[Next.js]] è il [[root layout]].

```tsx 
// layout.tsx
import '@/app/globals.css';

export default function RootLayout({ 
	children,
}: { 
	children: React.ReactNode;
}) { 
	return ( <html lang="en"> <body>{children}</body> </html> );
}
```

---

# CSS Modules

[[CSS Modules]] permette di creare CSS specifico per un componente ( `scoped` ) creando automaticamente nomi di classi uniche, così non c'è bisogno di preoccuparsi di [[style collision]].

```CSS
/* home.module.css */
.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```

```tsx
// page.tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import styles from '@/app/ui/home.module.css';
 
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col p-6">
      <div className={styles.shape} />
    // ...
  )
}
```

---

# Usare `clsx` per attivare e disattivare classi

Ci sono casi in cui ci potrebbe essere bisogno di stilizzare un elemento dinamicamente al cambio di state o altre condizioni.

[[clsx]] (https://www.npmjs.com/package/clsx) permette di farlo facilmente:

```tsx
import clsx from 'clsx';
 
export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```