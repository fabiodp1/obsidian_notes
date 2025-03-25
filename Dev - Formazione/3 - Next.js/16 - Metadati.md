I [[metadata]] sono molto importanti per la [[SEO]]. Questi aggiungono dettagli addizionali sulla pagina web ma non sono visibili agli utenti. Lavorano dietro le quinte embeddate nell'[[HTML]], normalmente all'interno del tag `<head>`. Questa informazione è cruciale per i motori di ricerca e altri sistemi che necessitano di capire meglio il contenuto della pagina web.

---

# Tipi di [[metadata]]

- **Title metadata**: responsabile del titolo della pagina web che viene mostrato sul tab del browser, molto importante per i motori di ricerca.
- **Description metadata**: fornisce una breve descrizione del contenuto della pagina, spesso mostrata nei risultati dei motori di ricerca:

```HTML
<meta name="description" content="A brief description of the page content." />
```

- **Keyword metadata**: include le parole chiave sul contenuto della pagina, aiutando l'indicizzazione dei motori di ricerca.

```html
<meta name="keywords" content="keyword1, keyword2, keyword3" />
```

- **Open Graph metadata**: migliora il modo in cui una pagina viene rappresentata nel momento in cui viene condivisa nei social, fornendo informazioni come il titolo, la descrizione e la preview dell'immagine:

```html
<meta property="og:title" content="Title Here" />
<meta property="og:description" content="Description Here" />
<meta property="og:image" content="image_url_here" />
```

- **Favicon metadata**: lega la `favicon` alla pagina web, mostrandola sul tab del browser:

```html
<link rel="icon" href="path/to/favicon.ico" />
```

---

# Aggiungere metadati

[[Next.js]] possiede un'[[API]] per i [[metadata]] ed esistono 2 modi per utilizzarla:

- **Config-based**: esportando un oggetto statico [[metadata]] o una funzione dinamica `generateMetadata` in un file `layout.tsx` o `page.tsx`.
- **File-based**: [[Next.js]] possiede una serie di file speciali specifici per i [[metadata]]:
	- `favicon.ico`, `apple-icon.jpg` e `icon.jpg`: utilizzati per le `favicon` e icone.
	- `opengraph-image.jpg` e `twitter-image.jpg`: per le immagini dei social media.
	- `robots.txt`: fornisce info per i crawler dei motori di ricerca.
	- `sitemap.xml`: offre informazioni sulla struttura della pagina web.

Si ha la flessibilità di utilizzare questi file per [[metadata]] statici, o possono essere generati programmaticamente.

Con queste due possibilità [[Next.js]] automaticamente genererà gli elementi per l'`<head>`.

---

# Favicon e Open graph

Nel momento in cui spostiamo la `favicon` e l'immagine [[open graph]] nella root della cartella `app`, [[Next.js]] le identificherà e utilizzerà.

>È anche possibile creare immagini [[open graph]] in maniera dinamica utilizzando il costruttore [[ImageResponse]].

---

# Titolo e descrizione

È possibile anche includere l'oggetto [[metadata]] da qualsiasi file [[layout.tsx]] o [[page.tsx]] per aggiungere ulteriori informazioni alla pagina, come titolo e descrizione.

>Qualsiasi [[metadata]] in [[layout.tsx]] verrà ereditato da tutte le pagine che lo usano.

```tsx
// /app/layout.tsx

import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: 'Acme Dashboard',
  description: 'The official Next.js Course Dashboard, built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
 
export default function RootLayout() {
  // ...
}
```

Facendo così [[Next.js]] aggiungerà automaticamente titolo e [[metadata]] all'applicativo.

Cosa succede però se volessimo aggiungere un titolo custom per una specifica pagina?
È possibile aggiungendo un oggetto [[metadata]] alla stessa pagina.

>I [[metadata]] inseriti in pagine innestate faranno l'override di quelli nel parent.

Per esempio seguendo l'esempio precedente se volessimo un titolo custom per la pagina `invoices`, si può fare così:

```tsx
// /dashboard/invoices/page.tsx

import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: 'Invoices | Acme Dashboard',
};
```

Questo funzionerà ma stiamo ripetendo il tiolo dell'applicativo in ogni pagina. Se qualcosa cambia, come il nome della company, sarà necessario farne l'update in ogni pagina.

Per evitare ciò basta utilizzare il campo `title.template` nell'oggetto [[metadata]], per definire un template per i titoli della pagina. Questo template può includere oltre al titolo qualsiasi altra info.

```tsx
// /app/layout.tsx

import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: {
    template: '%s | Acme Dashboard',
    default: 'Acme Dashboard',
  },
  description: 'The official Next.js Learn Dashboard built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
```

Il `%s` nel template sarà sostituito con un titolo specifico:

```tsx
// /app/dashboard/invoices/page.tsx

export const metadata: Metadata = {
  title: 'Invoices',
};
```