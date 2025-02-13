I services sono un modo per riutilizzare codice nell'applicativo utilizzando la [[dependency injection]]. Sono un po come i [[composable]] di [[Vue]].

Similmente ai componenti per dichiarare un service bisogna:
- Utilizzare un decorator `@Injectable` che serve a definire quale parte dell'applicativo può consumare il service attraverso la proprietà `providedIn` (normalmente ha il valore di `root`).
- Dichiarare la classe che contiene la logica del service che sarà resa disponibile.

```ts
import {Injectable} from '@angular/core';

@Injectable({providedIn: 'root'})

export class Calculator {
  add(x: number, y: number) {
    return x + y;
  }
}
```

```ts

```