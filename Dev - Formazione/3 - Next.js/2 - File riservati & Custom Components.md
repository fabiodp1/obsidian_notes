# File riservati

All'interno della directory `/app` ci sono anche altri file oltre a `page.tsx` e `layout.tsx` che hanno un nome riservato (almeno all'interno di `/app`):

- `globals.css`: viene importato nel `layout.tsx` e contiene tutte le classi [CSS](CSS) disponibili per tutto l'applicativo;
- `icon.png`: verrà usato in automatico da [Next.js](Next.js) come una `favicon` (non ci sarà bisogno di importarla o inserirla nell'html del `Root Layout`);
- `not-found.tsx`: pagina di fallback per errori `404` (lanciati da fratelli o pagine innestate o layout);
- `error.tsx`: pagina di fallback per gli altri tipi di errore;
- `loading.tsx`: pagina di fallback da mostrare mentre fratelli o pagine innestate (o layout) stanno facendo il fetch di dati;
- `route.ts`: permette di creare una `API route` (ad es. una pagina che non ritorna `tsx` ma invece dati in formato JSON);

Per maggiori info vedere la [documentazione ufficiale](https://nextjs.org/docs/app/api-reference/file-conventions).

---

# Custom Components

Ovviamente oltre al file `page.tsx`, per creare le nostre pagine, possiamo anche creare dei componenti da importare in giro e utilizzare, come in una normale app [React](React.md).

>A differenza di [React](React.md), in [Next.js](Next.js) per convenzione il nome dei file dei componenti può anche essere in `lowercase`.

Non essendo file speciali, i custom components non verranno trattati in nessun modo da [Next.js](Next.js), per essere usati andranno importati come normalmente avviene in un'app [React](React.md).

```tsx
// In Next.js il simbolo '@' indica la root del progetto
import Header from '@/components/header';

export default function Home() {
  return (
    <main>
      <Header />
      //...
    </main>
  )
}
```

>Anche mettendoli in una sotto-cartella `components`, non cambierà niente. Non essendo `components` un nome riservato e non contenendo il file `page.tsx`, non sarà considerato una `route`.

Dove posizionare i componenti è una scelta personale, un altro pattern potrebbe essere quello di lasciarli fuori dalla cartella `/app`, in modo da lasciarvi dentro solo file che hanno a che fare con la gestione delle `route`.