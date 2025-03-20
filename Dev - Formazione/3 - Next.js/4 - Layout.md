# Creare un layout

Come per `page.tsx`, [[Next.js]] possiede un altro tipo di file speciale che serve a creare UI condiviso tra pagine, `layout.tsx`.

```tsx
import SideNav from '@/app/ui/dashboard/sidenav';
 
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
      <div className="w-full flex-none md:w-64">
        <SideNav />
      </div>
      <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{ children }</div>
    </div>
  );
}
```

Il componente `<Layout />` riceve una prop `children`. Questo può essere sia una pagina o un altro layout (di una pagina innestata).

![[Pasted image 20241209170949.png]]

Nell'esempio le pagine dentro `/dashboard` saranno automaticamente innestate in `<Layout />`.

Ov

>Un beneficio di utilizzare i layouts in [[Next.js]] è quello che durante la navigazione, solo i componenti delle pagine si re-renderizzano, mentre il [[layout]] no. Questo viene chiamato [[partial prerendering]].

---

# Root layout

Nel capitolo [7 - Ottimizzare font e immagini](7%20-%20Ottimizzare%20font%20e%20immagini.md)  è stato aggiunto un font nel layout `/app/layout.tsx`, proprio alla root della cartella `app`.
Questo viene chiamato `root layout` ed è **obbligatorio**. Ogni UI aggiunta a questo layout sarà condivisa fra tutte le pagine dell'applicativo.
Può essere utilizzato per modificare i tag `<html>` e `<body>` e aggiungere metadati.

>Per aggiungere i `metadata` in [Next.js](Next.js) non useremo il tag `<head>`, ma basterà esportare una variabile `metadata` (nome riservato) e Next.js si occuperà in automatico di gestirlo. Per maggiori info guarda [16 - Metadati](16%20-%20Metadati.md).

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';

export const metadata = {
  title: 'NextJS Course',
  description: 'Your first NextJS app'
};

export default function RootLayout({ children }) { //<== Il contenuto iniettato 
  return (
    <html lang="en">
      <body>{ children }</body>
    </html>
  );
}
```

---

# Nested layouts

È possibile innestare anche più `layout` nell'albero di pagine e sotto-pagine, non siamo limitati al `Root Layout`.
Per fare ciò basta aggiungere alla sotto-route il file `layout.tsx` come facciamo per `page.tsx`.

>Così facendo il layout creato sara accessibile solo dalla route in cui è stato definito, e sarà innestato all'interno della `Route Layout`.

Anche per queste andrà utilizzata la prop `children` per mostrare la page per cui Next.js si occuperà di fare il wrap.