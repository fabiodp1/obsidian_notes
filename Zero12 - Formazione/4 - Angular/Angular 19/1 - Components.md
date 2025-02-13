# Definizione

In [[Angular]] i componenti sono delle classi che contengono la logica del componente, e un [[decorator]] che contiene altri dati come il template, css, e il selettore css che definisce il componente.

```ts
// user-profile.ts
@Component({
  selector: 'user-profile',
  template: `
    <h1>User profile</h1>
    <p>This is the user profile page</p>
  `,
  styles: `h1 { font-size: 3em; } `,
})
export class UserProfile { /* Your component code goes here */ }
```


>Sia template che css possono essere importati da altri file tramite `templateUrl` e `styleUrl`:

```ts
// user-profile.ts
@Component({
  selector: 'user-profile',
  templateUrl: 'user-profile.html',
  styleUrl: 'user-profile.css',
})
export class UserProfile {
  // Component behavior is defined in here
}
```

# Utilizzo

È possibile utilizzare i componenti nel template dopo averli importati all'interno del decoratore:

```ts
// user-profile.ts
import {ProfilePhoto} from 'profile-photo.ts';
@Component({
  selector: 'user-profile',
  imports: [ProfilePhoto],
  template: `
    <h1>User profile</h1>
    <profile-photo />
    <p>This is the user profile page</p>
  `,
})
export class UserProfile {
  // Component behavior is defined in here
}
```