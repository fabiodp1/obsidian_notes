# NextAuth.js

Normalmente per aggiungere autenticazione ad una app [[Next.js]] viene utilizzato [NextAuth.js](https://nextjs.authjs.dev/). Estrae gran parte della complessità nella gestione delle sessioni, sign-in e sign-out e altri aspetti dell'autenticazione.

## Configurazione

Dopo aver installato [[Next.js]] creare una chiave che verrà usata per criptare i cookies, assicurando la sicurezza delle sessioni utente:

```terminal
openssl rand -base64 32
```

dopo di che aggiungere la chiave nel file [[.env]]:

```.env
AUTH_SECRET=your-secret-key
```

Creare nella root del progetto il file [[auth.config.ts]] che esporterà un oggetto [[authConfig]], che conterrà le opzioni di configurazione per [[NextAuth.js]] :

```ts
import type { NextAuthConfig } from 'next-auth';
 
export const authConfig = {
  pages: {
    signIn: '/login',
  },
} satisfies NextAuthConfig;
```

L'opzione `pages` può essere usata per specificare la [[route]] per il sign-in sign-out e le pagine di errore. Non è richiesto, ma aggiungendo il campo `signIn: '/login'` l'utente verrà rediretto alla pagina di login custom e non quella di default di [[NextAuth.js]].

## Proteggere le route con i middleware [[Next.js]]

Altro cosa da fare è aggiungere della logica per evitare che l'utente acceda a pagine a cui non è autorizzati se non loggato.

```ts
import type { NextAuthConfig } from 'next-auth';
 
export const authConfig = {
  pages: {
    signIn: '/login',
  },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL('/dashboard', nextUrl));
      }
      return true;
    },
  },
  providers: [], // Add providers with an empty array for now
} satisfies NextAuthConfig;
```

La callback `authorized` viene usata per verificare se la richiesta è autorizzata per l'accesso ad una pagina tramite il [[Next.js Middleware]]. Questo viene chiamato prima che la richiesta venga completata, e riceve un oggetto con le proprietà `auth` e `request`. L proprietà `auth` contiene la sessione utente, mentre `request` contiene la richiesta in arrivo.
L'opzione `providers` è un'array dove vengono elencate diverse opzioni per il login.

Una volta fatto questo, è necessario importare la configurazione [[authConfig]] nel file [[middleware.ts]] nella root del progetto:

```ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
 
export default NextAuth(authConfig).auth;
 
export const config = {
  // https://nextjs.org/docs/app/building-your application/routing/middleware#matcher
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
};
```

Viene iniazializzato [[NextAuth.js]] con l'[[authConfig]] e viene esportata la proprietà `auth`. Viene anche usata l'opzione `matcher` del middleware per specificare che dovrebbe funzionare su specifiche path.

>Il vantaggio di utilizzare il middleware è che le rotte protette non inizieranno neanche a renderizzare fino a quando il middleware verifica l'autenticazione, aumentando sia la sicurezza che le performance dell'applicativo.

## Password hashing

È buona pratica fare il [[password hashing]] prima di salvarle nel [[DB]]. Una libreria molto utilizzata per farlo è [[bcrypt]], ma poiché fa affidamento sulle [[API]] di [[Node.js]] non disponibili in [[Next.js Middleware]] bisognerà creare un altro file per utilizzarlo, non potendolo usare nello stesso del middleware:

```ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
});
```

## Aggiungere il provider delle credenziali

È una lista in cui vengono passate le opzioni per il login come ad es. Google o GitHub. In questo caso useremo il semplice [Credentials provider](https://authjs.dev/getting-started/providers/credentials-tutorial) che permette il login tramite username e password.

```ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [Credentials({})],
});
```

>È consigliato normalmente di utilizzare metodi di autenticazione più sicuri, come provider [[OAuth]] o email, per le varie possibilità consultare i [NextAuth.js docs](https://authjs.dev/getting-started/providers).

## Aggiungere una funzionalità di sign-in

È possibile utilizzare la funzione `authorize` per gestire la logica di #autenticazione, un po come per le [[server action]], è possibile usare [[Zod]] per validare la email e password prima di controllare se l'utente esiste a [[DB]]:

```tsx
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
import { z } from 'zod';
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);
      },
    }),
  ],
});
```

In questa logica possiamo utilizzare [[bcrypt]] per validare la password:

```ts
import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { sql } from '@vercel/postgres';
import { z } from 'zod';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';
 
// ...
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        // ...
 
        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
          const passwordsMatch = await bcrypt.compare(password, user.password);
 
          if (passwordsMatch) return user;
        }
 
        console.log('Invalid credentials');
        return null; // ritornando null si evita che l'utente venga loggato
      },
    }),
  ],
});
```

A questo punto è possibile aggiungere utilizzare la funzione di autenticazione per creare l'action di autenticazione:

```ts
'use server';
 
import { signIn } from '@/auth';
import { AuthError } from 'next-auth';
 
// ...
 
export async function authenticate(
  prevState: string | undefined,
  formData: FormData,
) {
  try {
    await signIn('credentials', formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return 'Invalid credentials.';
        default:
          return 'Something went wrong.';
      }
    }
    throw error;
  }
}
```

Nel caso in cui il tipo di errore dovesse essere un '[[CredentialsSignin]]', viene mostrato l'errore appropriato.

Alla fine il form di autenticazione dell'esempio sarà:

```tsx
'use client';
 
import { lusitana } from '@/app/ui/fonts';
import {
  AtSymbolIcon,
  KeyIcon,
  ExclamationCircleIcon,
} from '@heroicons/react/24/outline';
import { ArrowRightIcon } from '@heroicons/react/20/solid';
import { Button } from '@/app/ui/button';
import { useActionState } from 'react';
import { authenticate } from '@/app/lib/actions';
 
export default function LoginForm() {
  const [errorMessage, formAction, isPending] = useActionState(
    authenticate,
    undefined,
  );
 
  return (
    <form action={formAction} className="space-y-3">
      <div className="flex-1 rounded-lg bg-gray-50 px-6 pb-4 pt-8">
        <h1 className={`${lusitana.className} mb-3 text-2xl`}>
          Please log in to continue.
        </h1>
        <div className="w-full">
          <div>
            <label
              className="mb-3 mt-5 block text-xs font-medium text-gray-900"
              htmlFor="email"
            >
              Email
            </label>
            <div className="relative">
              <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                id="email"
                type="email"
                name="email"
                placeholder="Enter your email address"
                required
              />
              <AtSymbolIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
            </div>
          </div>
          <div className="mt-4">
            <label
              className="mb-3 mt-5 block text-xs font-medium text-gray-900"
              htmlFor="password"
            >
              Password
            </label>
            <div className="relative">
              <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                id="password"
                type="password"
                name="password"
                placeholder="Enter password"
                required
                minLength={6}
              />
              <KeyIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
            </div>
          </div>
        </div>
        <Button className="mt-4 w-full" aria-disabled={isPending}>
          Log in <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50" />
        </Button>
        <div
          className="flex h-8 items-end space-x-1"
          aria-live="polite"
          aria-atomic="true"
        >
          {errorMessage && (
            <>
              <ExclamationCircleIcon className="h-5 w-5 text-red-500" />
              <p className="text-sm text-red-500">{errorMessage}</p>
            </>
          )}
        </div>
      </div>
    </form>
  );
}
```

## Aggiungere la funzionalità di logout

Per fare ciò a questo punto basta chiamare la funzione `signOut` esportata sempre dal file `auth.ts`:

```tsx
import Link from 'next/link';
import NavLinks from '@/app/ui/dashboard/nav-links';
import AcmeLogo from '@/app/ui/acme-logo';
import { PowerIcon } from '@heroicons/react/24/outline';
import { signOut } from '@/auth';
 
export default function SideNav() {
  return (
    <div className="flex h-full flex-col px-3 py-4 md:px-2">
      // ...
      <div className="flex grow flex-row justify-between space-x-2 md:flex-col md:space-x-0 md:space-y-2">
        <NavLinks />
        <div className="hidden h-auto w-full grow rounded-md bg-gray-50 md:block"></div>
        <form
          action={async () => {
            'use server';
            await signOut();
          }}
        >
          <button className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3">
            <PowerIcon className="w-6" />
            <div className="hidden md:block">Sign Out</div>
          </button>
        </form>
      </div>
    </div>
  );
}
```

