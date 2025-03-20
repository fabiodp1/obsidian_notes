# WHAT

AWS [[Amplify]] è un collezione di servizi cloude e librerie per lo sviluppo di applicazioni fullstack.
Fornisce librerie forntend, componenti [UI](UI), building backend, a frontend hosting per il build di applicazioni fullstack in cloude. In particolare la [Gen 2](https://docs.amplify.aws/react/how-amplify-works/concepts/) migliora ulteriormente l'esperienza di sviluppo.

Mentre [[Cognito]] è una `user directory service` che gestisce la registrazione utente, autenticazione, account recovery e molto altro concernente i flussi auth.

---

# Collegare frontend a Cognito

Per configurare il nostro `frontend` per utilizzare il sistema di gestione utenze di `Cognito`, creeremo la nostra configurazione nel file di root dell'applicativo `main.ts`, utilizzando la classe `Amplify` messa a disposizione da `aws-amplify`:

```ts
// main.ts
import { Amplify } from "aws-amplify"

Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: "<your-cognito-user-pool-id>",
      userPoolClientId: "<your-cognito-user-pool-client-id>",
      identityPoolId: "<your-cognito-identity-pool-id>",
      loginWith: {
        email: true,
      },
      signUpVerificationMethod: "code",
      userAttributes: {
        email: {
          required: true,
        },
      },
      allowGuestAccess: true,
      passwordFormat: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireNumbers: true,
        requireSpecialCharacters: true,
      },
    },
  },
})
```

# Utilizzo

Per tutte le azioni utili al flusso di autenticazione e gestione utenze, si può utilizzare la libreria `auth` messa a disposizione da `aws-amplify`, semplicemente importandoli da `aws-amplify/auth`:

```ts
import { signIn } from 'aws-amplify/auth'

await signIn({
  username: "hello@mycompany.com",
  password: "hunter2",
})
```

Fare comunque riferimento alla [documentazione ufficiale](https://docs.amplify.aws/react/build-a-backend/auth/connect-your-frontend/).

# Ascoltare gli eventi di Auth

La libreria `auth` di [Amplify](Amplify) emette degli eventi durante il flow di autenticazione, che ci permettono di reagire in tempo reale per lanciare della logica custom (es. per ottenere i dati e aggiornare lo state dell'app ecc.).

Per fare ciò è possibile utilizzare la classe [Hub](https://docs.amplify.aws/gen1/react/build-a-backend/utilities/hub/) di [Amplify](Amplify) con i suoi eventi built-in a cui sottoscrivere un `listener` con un pattern `publish-subscribe`, catturando gli eventi da qualsiasi parte dell'applicativo.