# Signal

In [[Angular]] la reattività avviene wrappando i valori che rappresentano lo state del componente, tramite il [[signal]]:

```ts
import {signal} from '@angular/core';
// Create a signal with the `signal` function.
const firstName = signal('Morgan');
// Read a signal value by calling it— signals are functions.
console.log(firstName());
// Change the value of this signal by calling its `set` method with a new value.
firstName.set('Jaime');
// You can also use the `update` method to change the value
// based on the previous value.
firstName.update(name => name.toUpperCase());
```

[[Angular]] tiene traccia di quando un signal viene letto o aggiornato, aggiornando se serve il [[DOM]].

# Computed

Un oggetto [[computed]] è un signal che produce il proprio valore sulla base di altri [[signal]].

```ts
import {signal, computed} from '@angular/core';

const firstName = signal('Morgan');
const firstNameCapitalized = computed(() => firstName().toUpperCase());
console.log(firstNameCapitalized()); // MORGAN
```

Si tratta di un oggetto `read-only`, non può essere assegnato o aggiornato. Il suo valore cambia automaticamente al cambiare dei signal da cui dipende.


# Utilizzo

Per utilizzare i [[signal]] e le [[computed]] all'interno della classe di un componente:

```ts
@Component({/* ... */})

export class UserProfile {
  isTrial = signal(false);
  isTrialExpired = signal(false);
  showTrialDuration = computed(() => this.isTrial() && !this.isTrialExpired());
  
  activateTrial() {
    this.isTrial.set(true);
  }
}
```