Al build dell'applicativo, tutti i file e relativi `import` verranno collegati e mergiati in un unico file bundle che il browser dovrà caricare e utilizzare prima di poter mostrare qualcosa. Nella maggior parte dei casi potrebbe non essere un problema, ma nel caso di grandi applicazioni, questi bundle possono essere molto grandi e quindi richiedere più tempo prima di essere richiesti e caricati dal browser, causando una [UX](UX) pessima.

Per questo esiste il [[lazy loading]] che evita che venga creato un unico bundle, ma fa in modo che venga spezzettato in più file da caricare solo all'occorrenza.

## Implementazione

In [React](React.md) per implementarlo bisogna:

1. Non importare i componenti e codice che vogliamo caricare in un secondo momento, tramite `import`
2. Per importare ad es. i loader da assegnare alle `route`, possiamo, possiamo sostituire il loader normale con una funzione che chiamerà il loader dopo averlo caricato in maniera `lazy` utilizzando `import(...)` che utilizzata in questa maniera importerà dinamicamente e in maniera `lazy` la dipendenza, restituendo una `Promise`.

```ts
// import BlogPage, { loader as postsLoader } form './pages/Blog';

<Route ... element={<BlogPage />}
	// import() ritorna una Promise
	// il loader verrà chiamato una volta risolta la Promise
	loader={(meta) => 
		import('./pages/Blog').then(module => module.loader(meta))       <===
	}
/>
```

3. Per importare invece componenti [[React]], non possiamo semplicemente utilizzare `import()` e utilizzarlo come fosse un function component, perché di per se non ritorneremmo il tsx del componente ma la Promise. Per questo ci viene in aiuto l'utility `lazy` di [React](React.md):

```ts
// import BlogPage, { loader as postsLoader } form './pages/Blog';

const BlogPage = lazy(() => import('./pages/Blog'));     <===

<Route ... element={<BlogPage />} .../>
```

4. Utilizzare il componente `Suspense` per wrappare i componenti che verranno caricati in maniera `lazy` (comunque ci metteranno un po ad essere caricati):

```ts
import {lazy, Suspense} from 'react';

<Route ... element={
		<Suspense fallback={<p>Loading...</p>}>
			<BlogPage />
		</Suspense>
	}
	...
/>
```

>`Suspense` è un componente speciale che altri componenti possono utilizzare per aspettare il caricamento di un child prima di essere renderizzati.