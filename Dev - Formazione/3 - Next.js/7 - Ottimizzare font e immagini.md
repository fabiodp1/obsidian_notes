# Fonts
## WHY

L'utilizzo di custom fonts può creare problemi di performance nel caso in cui necessitassero il fetch e caricamento.
Il [[Cumulative Layout Shift]] è una metrica utilizzata da Google per valutare le performance e la UX di un sito web. Con i font il [[layout shift]] avviene quando il browser inizialmente renderizza il testo in un font di fallback o di sistema e poi passa a quello custom.
Questo può causare il cambiamento delle dimensioni, spaziature ecc. del testo, facendo spostare gli elementi adiacenti.

Next.js automaticamente ottimizza i font quando viene utilizzato il modulo [[next/font]]. Scarica i file dei font a *build-time* e li mantiene insieme agli altri file statici. Questo vuol dire che quando un utente visita l'applicazione, non ci sono richieste per font che peggiorerebbero le performance.

## Aggiungere un font primario

In genere viene creato un file `fonts.ts` nella cartella `ui`, che avrà il compito di contenere tutti i font utilizzati nell'applicazione.

```ts
import { Inter } from 'next/font/google'; // Font primario utiilizzato
 
export const inter = Inter({ subsets: ['latin'] }); // Specifico il subset
```

Dopo di che viene aggiunto il font all'elemento `<body>` in `/app/layout.tsx`:

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

Aggiungedo `Inter` al body facciamo in modo che venga applicato a tutta l'applicazione. In questo caso è stata applicata anche la classe [[Tailwind]] [[antialiased]] che migliora la visualizzazione del font.

---

# Images
## WHY

[[Next.js]] può servire i file statici, come immagini, presenti nella cartella `/public`. E' inoltre possibile referenziare i file presenti in questa cartella da per tutto nell'applicazione.

Con normale HTML per aggiungere una immagine faremmo:

```html
<img
  src="/hero.png"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```

Questo però comporta che ci toccherà manualmente:
- assicurarci che l'immagine sia responsive su diferse dimenzioni di schermo
- specificare le dimenzioni delle immagini per dispositivi differenti
- prevenire il [[layout shift]] al caricamento delle immagini
- fare il lazy loading delle immagini che sono al di fuori della viewport dell'utente

Invece di fare questo manualmente, è possibile utilizzare il componente [[next/image]] per ottimizzare automaticamente le immagini.

## Il componente [[<Image>]]

E' una estensione del tag html `<img>,` e ottimizza automaticamente le immagini come:
- Prevenire il [[layout shift]] al load delle immagini
- Ridimenzionare le immagini per evitare di pubblicare immagini troppo grandi per il device con una viewport più piccola
- [[lazy loading]] delle immagini di default (solo quando entrano nella viewport)
- Servire le immagini in formati pià moderni, come WebP e AVIF, nel caso in cui il browser le suportasse.

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';
 
export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000} // Importante to set both for avoiding layout shift
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
    </div>
    //...
  );
}
```