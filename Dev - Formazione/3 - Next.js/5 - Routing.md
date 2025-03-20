Come abbiamo visto prima [Next.js](Next.js) utilizza la composizione delle cartelle nella directory `app` e i relativi file `page.tsx`, per definire le `route` dell'applicativo.
Rimane da capire come sia possibile definire `route` più complesse come quelle dinamiche.

# Dynamic Routes

Se ad es. abbiamo un'applicativo che contiene una `route` blog, da questa pagina vorremo anche poter accedere ai singoli `post`, ma come possiamo definire l'endpoint per il post-1, o il post-2 ecc.? Non possiamo di sicuro creare una cartella per ogni nuovo post creato.

In questo caso abbiamo bisogno di rotte dinamiche, definite una volta sola, ma capaci di mostrare il contenuto in maniera dinamica.

>In [Next.js](Next.js) creiamo rotte dinamiche creando una cartella nominata con dell parentesi quadre `[]` definendo all'interno delle parentesi quello che vogliamo.

```
/app
  |__/blog
        |__page.tsx
        |__[id]
              |__page.tsx
```

```tsx
// /app/blog/page.tsx

import Link from 'next/link';

export default function BlogPage() {
  return (
    <main>
      <h1>The Blog</h1>
      <p><Link href="/blog/post-1">Post 1</Link></p>
      <p><Link href="/blog/post-2">Post 2</Link></p>
    </main>
  )
}
```

```tsx
// /app/blog/[id]/page.tsx

import Link from 'next/link';

export default function BlogPostPage( { params } ) {
  
  
  return (
    <main>
      <h1>Blog Post {params.id}</h1>
    </main>
  )
}
```

>Per poter accedere ai parametri della `route`, [Next.js](Next.js) passerà automaticamente delle prop al nostro componente che potremo utilizzare per accedervi, non ci sarà bisogno di farlo manualmente.