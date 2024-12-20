# WHAT
Lo [[streaming]] è una tecnica di trasferimento dati che permette di spezzettare una [[route]] in pezzi più piccoli ([[chunk]]) e progressivamente farne lo stream dal server al client man mano che son pronti.

# WHY
E' possibile in questa maniera evitare che richieste di dati più lente vadano a bloccare l'intera pagina. Questo fa in modo che l'utente veda e possa interagire con parti della pagina senza dover aspettare che tutti i dati vengano caricati.

# HOW
Lo [[streaming]] funziona bene con i componenti di [[React]] visto che ogni componente può essere considerato un [[chunk]].

Ci sono 2 modi per implementare lo streaming in [[Next.js]]:
1. A livello di pagina, tramite il file [[loading.tsx]]
2. Per specifici componenti, tramite <[[Suspense]]>

# [[loading.tsx]]
E' un file speciale di [[Next.js]] costruito su [[Suspense]] che permette di creare UI di fallback da mostrare come sostituto nel mentre il contenuto della pagina carica.

```tsx
// loading.tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

# Loading skeletons
Uno scheletro di caricamento ([[loading skeleton]]) è una versione semplificata dell'[[UI]]. Molti siti web l'utilizzano come placeholder per indicare all'utente che il contenuto sta caricando.

Qualsiasi [[UI]] venga aggiunta a [[loading.tsx]], sarà inserita come parte dei file statici e inviato come prima cosa, poi il resto del contenuto dinamico verrà inviato come [[stream]] dal server al client.

```tsx
import DashboardSkeleton from '@/app/ui/skeletons';
 
export default function Loading() {
  return <DashboardSkeleton />;
}
```

Poiché in genere il file [[loading.tsx]] si trova ad un livello più alto rispetto ad altri file contenuti in sub-route, automaticamente viene applicato a tutti quelli che stanno sotto. Per questo in certi casi può essere necessario creare [[route group]]. 
Per fare ciò basta inserire i file interessati in una sottocartella nominata con `()`, in questa maniera il nome del contenuto non verrà incluso nel path dell'[[URL]]

![[Pasted image 20241216094250.png]]

Nell'esempio il file [[loading.tsx]] verrà applicato solo alla pagina overview della dashboard.
Comunque è possibile utilizzare [[route group]] per separare l'applicazione in sezioni (es. le rotte per `(marketing)` e `(shop)`) o per team in applicativi più grandi.

# Streaming di un componente
E' possibile essere molto granulari e fare lo [[stream]] dei singoli componenti grazie a [[Suspense]].
[[Suspense]] permette di differire parti da renderizzare dell'applicativo fino a quando una determinata condizione viene soddisfatta. Basta fare il wrap del componente dinamico e passare un componente di fallback da mostrare nel mentre l'altro viene caricato:

```tsx
<Suspense fallback={<RevenueChartSkeleton />}>
	<RevenueChart  />
</Suspense>
```

>E' consigliato nell'utilizzo di [[Suspense]] di spostare il fetch dei dati dal padre al componente wrappato.