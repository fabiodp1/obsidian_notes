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


# Utilizzo Signal e Computed

## Classe

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

## Template

Per utilizzarli all'interno del template, basta circondare il [[signal]] con `{{ ... }}`:

```ts
@Component({
  selector: 'user-profile',
  template: `<h1>Profile for {{userName()}}</h1>`,
})

export class TodoListItem {
  userName = signal('pro_programmer_123');
}
```

Per utilizzarli come valore degli attributi [[HTML]] invece vengono utilizzate le parentesi quadre:

```ts
@Component({
  /*...*/
  // Set the `disabled` property of the button based on the value of `isValidUserId`.
  template: `<button [disabled]="isValidUserId()">Save changes</button>`,
})

export class UserProfile {
  isValidUserId = signal(false);
}
```

È anche possibile fare il bind delle proprietà del [[DOM]] aggiungendo il prefisso `attr.` al nome della proprietà:

```ts
<!-- Bind the `role` attribute on the `<ul>` element to value of `listRole`. -->
<ul [attr.role]="listRole()">
```

# Interazione utente

È possibile gestire gli eventi come il `click` utilizzando le parentesi tonde:

```ts
@Component({
  /*...*/
  // Add an 'click' event handler that calls the `cancelSubscription` method. 
  template: `<button (click)="cancelSubscription()">Cancel subscription</button>`,
})

export class UserProfile {
  /* ... */
  
  cancelSubscription() { /* Your event handling code goes here. */  }
}
```

Per passare l'evento, basta usare la variabile `built-in` di [[Angular]]:

```ts
@Component({
  /*...*/
  // Add an 'click' event handler that calls the `cancelSubscription` method. 
  template: `<button (click)="cancelSubscription($event)">Cancel subscription</button>`,
})

export class UserProfile {
  /* ... */
  
  cancelSubscription(event: Event) { /* Your event handling code goes here. */  }
}
```

# @if e @for

Come con [[Vue]] possiamo mostrare o nascondere parte del template in maniera condizionale o creare un loop di elementi, utilizzando `@if` e `@for`:

```ts
<h1>User profile</h1>

@if (isAdmin()) {
  <h2>Admin settings</h2>
  <!-- ... -->
} @else {
  <h2>User settings</h2>
  <!-- ... -->  
}
```

```ts
<h1>User profile</h1>

<ul class="user-badge-list">
  @for (badge of badges(); track badge.id) {
    <li class="user-badge">{{badge.name}}</li>
  }
</ul>
```

A differenza di [[Vue]] e [[React]], nel creare un loop di elementi [[Angular]] non utilizza #key per tenere traccia della lista di elementi, ma la keyword #track seguita dal valore da utilizzare.