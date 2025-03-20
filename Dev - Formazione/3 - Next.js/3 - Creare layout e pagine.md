# Routing innestato

[[Next.js]] utilizza il [[file-system routing]] in cui le cartelle vengono usate per creare rotte innestate. Ogni folder rappresenta un segmento della [[route]] che mappa ad un segmento dell'[[URL]].

![[Pasted image 20241209161745.png]]

> [[page.tsx]] è un file speciale [[Next.js]] che esporta un componente [[React]] ed è richiesto dalla [[route]] per essere accessibile.

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

# Creare un layout

Come per [[page.tsx]], [[Next.js]] possiede un altro tipo di file speciale che serve a creare UI condiviso tra pagine, [[layout.tsx]].

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

>Un beneficio di utilizzare i layouts in [[Next.js]] è quello che durante la navigazione, solo i componenti delle pagine si re-renderizzano, mentre il [[layout]] no. Questo viene chiamato [[partial prerendering]].

>È quindi anche possibile innestare anche più `layout` nell'albero di pagine e sotto-pagine.

---

# Root layout

Nel capitolo [5 - Ottimizzare font e immagini](5%20-%20Ottimizzare%20font%20e%20immagini.md)  è stato aggiunto un font nel layout `/app/layout.tsx`, proprio alla root della cartella `app`.
Questo viene chiamato `root layout` ed è **obbligatorio**. Ogni UI aggiunta a questo layout sarà condivisa fra tutte le pagine dell'applicativo.
Può essere utilizzato per modificare i tag `<html>` e `<body>` e aggiungere metadati.

>Per aggiungere i `metadata` in [Next.js](Next.js) non useremo il tag `<head>`, ma basterà esportare una variabile `metadata` (nome riservato) e Next.js si occuperà in automatico di gestirlo. Per maggiori info guarda [16 - Aggiungere metadati](16%20-%20Aggiungere%20metadati.md).

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