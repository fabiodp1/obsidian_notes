Per la maggior parte delle web apps moderne, la scelta è fra [[static rendering]] e [[dynamic rendering]] per l'intera applicazione o per specifiche rotte. In [[Next.js]] se viene chiamata una [[dynamic function]] in una [[route]], l'intera [[route]] diventa dinamica.

Però la maggior parte delle [[route]] non sono completamente statiche o dinamiche. Per esempio se consideraiamo un ecommerce potremmo avere la renderizzazione statica della maggior parte dei prodotti, ma il carrello utente e i prodotto suggeriti potremmo volere che vengano caricati dinamicamente.

# WHAT
[[PPR]] ([[partial prerendering]]) è un modello di rendering che permette di combinare i benefici del rendering statico e dinamico nella stessa [[route]]:

![[Pasted image 20241216113742.png]]

Quando un utente visita una rotta:
- Viene servito un guscio della rotta che include la navbar e le info sui prodotti, permettendo un caricamento iniziale più veloce.
- Il guscio lascia buchi in cui il contenuto del carrello e i prodotti suggeriti verranno caricati asincronicamente.
- I buchi asincroni vengono streammati in parallelo, riducendo il caricamento generale della pagina.

# HOW
Il [[partial prerendering]] utilizza [[Suspense]], il fallback viene incluso nell'[[HTML]] iniziale con il contenuto statico, il contenuto statico così viene prerenderizzato creando il guscio statico. Il rendering del contenuto dinamico viene rimandato a quando l'utente richiederà la rotta.

>Wrappere un componente in [[Suspense]] non lo rende di per se dinamico, piuttosto [[Suspense]] viene utilizzato come confine fra il contenuto statico e quello dinamico.

Per attivare il [[PPR]]:

```ts
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  experimental: {
    ppr: 'incremental'
  }
};
export default nextConfig;


// layout.tsx
import SideNav from '@/app/ui/dashboard/sidenav'; 

export const experimental_ppr = true; 
// ...

```

>Non c'è bisogno di cambiare il codice, basta configurare e aver usato i [[Suspense]].

