# Perché ottimizzare la navigazione

Per creare link fra pagine normalmente utilizzeremmo il tag `<a>`, questo però nel caso ad esempio di un passaggio fra pagine tramite link sulla barra laterale, provoca il refresh di tutta la pagina.

---

# Il componente `<Link>`

In [[Next.js]] è possibile utilizzare il componente `<Link />` di [[next/link]] per linkare pagine, permette di fare [[client-side navigation]] con JavaScript.
Nonostante parti dell'applicativo vengono renderizzate server-side, non avviene il refresh, facendola sembrare una web app.

## [[code-splitting]] e [[prefetching]] automatico

Per migliorare l'esperienza della navigazione, [[Next.js]] automaticamente fa la separazione del codice dell'applicativo in base ai segmenti della route. Questo è diverso dalla tradizionale [[React]] SPA, in cui il browser carica tutto il codice applicativo al caricamento iniziale.
Inoltre in produzione, ogni qualvolta compare nella viewport un [[<Link>]] component, Next.js automaticamente fa in background il [[prefetch]] del codice per la rotta linkata. Nel momento in cui l'utente clicca sul link, il codice per la destinazione sarà già stata caricata in background, e questo è ciò che rende la transizione istantanea.

# Pattern: mostrare il link attivo

Un comune pattern UI è quello di mostrare all'utente dove si trova mostrando il link alla pagina attivo, per fare ciò è necessario avere il path corrente dall'[[URL]].
[[Next.js]] fornisce un [[hook]] chiamato [[usePathname]] che da questa informazione.

>Poichè [[usePathname]] è un hook, c'è bisogno di trasformare il componente che deve usarlo in un [[Client Component]]. Per fare ciò basta aggiungere in cima al file del componente [["use client"]] e poi importare [[usePathname]] da [[next/navigation]].

```tsx
'use client';
 
import {
  UserGroupIcon,
  HomeIcon,
  InboxIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
 
// ...
```

Poi sarà possibile usando [[clsx]] aggiungere dinamicamente la classe per rendere il link attivo confrontando il `pathname` con `link.href`:

```tsx 
// ...

  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className={clsx(
              'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
              {
                'bg-sky-100 text-blue-600': pathname === link.href,
              },
            )}
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}

```

