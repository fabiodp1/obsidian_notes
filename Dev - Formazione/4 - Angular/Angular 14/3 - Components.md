Ogni componente è formato da:
- Un template [[HTML]]
- Una classe [[TypeScript]] che ne definisce la logica
- Un selettore [[CSS]] che definisce come il componente viene usato in un template
- Opzionale, degli stili [[CSS]] da applicare al template

# Creazione

Per creare un componente bisogna prima rispettare questi requisiti:
- Aver installato l'[Angular CLI](https://v14.angular.io/guide/setup-local#install-the-angular-cli)
- Aver [creato un workspace Angular](https://v14.angular.io/guide/setup-local#create-a-workspace-and-initial-application)

Il miglior modo per creare un componente è utilizzare la CLI lanciando il comando

```terminal
ng generate component <component-name>
```

Di default questo comando andrà a creare:
- Un directory con il nome del componente `<component-name>.component.ts`
- Il file [[TypeScript]] del componente `<component-name>.component.html`
- Il file template [[HTML]] `<component-name>.component.css`
- Un file per il test `<component-name>.component.spec.ts`

>È possibile definire come la CLI farà lo scaffolding del componente utilizzando specifici [comandi](https://v14.angular.io/cli/generate#component-command) È possibile creare i componenti anche manualmente, ma la CLI è consigliata.

# Lifecycle hooks

Per utilizzare il `lifecycle hooks` di [[Angular]] il componente o direttiva deve implementare la relativa interfaccia:

```ts
@Component({selector: 'my-component'}) // può essere una direttiva
export class MyComponent implements OnInit {
  constructor()) { }

  // implement OnInit's `ngOnInit` method
  ngOnInit() {
    console.log('OnInit');
  }
}
```

Gli hook vengono eseguiti in questo ordine:

1. `ngOnChanges()` risponde quando Angular setta o resetta i *data-bound input properties*. Viene chiamato prima di *ngOnInit()* (se il componente è legato ad input) e ogni volta che un data-bound input prop cambia (prop passate dai padri).
2. `ngOnInit()` chiamato una sola volta dopo il primo *ngOnChanges()*.
3. `ngDoCheck()` chiamato immediatamente dopo ogni *ngOnChnges()*.
4. `ngAfterContentInit()`
5. `ngAfterContentChecked()`
6. `ngAfterViewInit()` lanciato dopo che Angular ha inizializzato la view del componente.
7. `ngAfterViewChecked()` lanciato dopo *ngAfterViewInit()* e ad ogni *ngAfterContentChecked()*.
8. `ngOnDestroy()` appena prima che Angular distrugga la direttiva o componente

# Passare dati da Parent a Child

Un componente figlio per dichiarare una prop utilizza il [[decorator]] `@Input()`:

```ts
import { Component, Input } from '@angular/core';

import { Hero } from './hero';

@Component({
  selector: 'app-hero-child',
  template: `
    <h3>{{hero.name}} says:</h3>
    <p>I, {{hero.name}}, am at your service, {{masterName}}.</p>
  `
})
export class HeroChildComponent {
  @Input() hero!: Hero;
  @Input('master') masterName = ''; // Alias
}
```

```ts
import { Component } from '@angular/core';
import { HEROES } from './hero';

@Component({
  selector: 'app-hero-parent',
  template: `
    <h2>{{master}} controls {{heroes.length}} heroes</h2>

    <app-hero-child
      *ngFor="let hero of heroes"
      [hero]="hero"
      <!-- Alias -->
      [master]="master">
    </app-hero-child>
  `
})
export class HeroParentComponent {
  heroes = HEROES;
  master = 'Master';
}
```

>Per tenere traccia nel figlio dei cambiamenti della prop si possono utilizzare diversi metodi, fra questi:

- Utilizzare un `setter`:

```ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-name-child',
  template: '<h3>"{{name}}"</h3>'
})
export class NameChildComponent {
  @Input()
  get name(): string { return this._name; }
  set name(name: string) {
    this._name = (name && name.trim()) || '<no name set>';
  }
  private _name = '';
}
```

- Utilizzare `ngOnChanges()`:

```ts
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-version-child',
  template: `
    <h3>Version {{major}}.{{minor}}</h3>
    <h4>Change log:</h4>
    <ul>
      <li *ngFor="let change of changeLog">{{change}}</li>
    </ul>
  `
})
export class VersionChildComponent implements OnChanges {
  @Input() major = 0;
  @Input() minor = 0;
  changeLog: string[] = [];

  ngOnChanges(changes: SimpleChanges) {
    const log: string[] = [];
    for (const propName in changes) {
      const changedProp = changes[propName];
      const to = JSON.stringify(changedProp.currentValue);
      if (changedProp.isFirstChange()) {
        log.push(`Initial value of ${propName} set to ${to}`);
      } else {
        const from = JSON.stringify(changedProp.previousValue);
        log.push(`${propName} changed from ${from} to ${to}`);
      }
    }
    this.changeLog.push(log.join(', '));
  }
}
```

# Ascoltare gli eventi del child

Per fare ciò il child espone una proprietà `EventEmitter`  on cui emette un [[custom event]].
La proprietà che si occuperà dell'emissione dell'evento è un `output property`, utilizzando il [[decorator]] `@Output()`

```ts
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-voter',
  template: `
    <button type="button" (click)="vote(true)">Agree</button>
    <button type="button" (click)="vote(false)">Disagree</button>
  `
})
export class VoterComponent {
  // ...
  @Output() voted = new EventEmitter<boolean>();

  vote(agreed: boolean) {
    this.voted.emit(agreed);
  }
}
```

Il padre per poterlo ascoltare dovrà:

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-vote-taker',
  template: `
    <app-voter
      *ngFor="let voter of voters"
      [name]="voter"
      (voted)="onVoted($event)">
    </app-voter>
  `
})
export class VoteTakerComponent {

  onVoted(agreed: boolean) {
    // ...
  }
}
```

# Accedere a prop e metodi child

Il padre per accedere alle proprietà e metodi del figlio dovrà creare una variabile di reference del template:

```ts
// === CHILD ===

@Component({
  selector: 'app-countdown-timer',
  // ...
})
export class CountdownTimerComponent {
  start() { //... }
  stop()  { //... }
}
```

```ts
// === FATHER ===

@Component({
  selector: 'app-countdown-parent-lv',
  template: `
    <button type="button" (click)="timer.start()">Start</button>
    <button type="button" (click)="timer.stop()">Stop</button>

    <!-- template reference variable #timer -->
    <app-countdown-timer #timer></app-countdown-timer>
  `,
})
export class CountdownLocalVarParentComponent { }
```

Questo approccio è abbastanza diretto, ma non permette va fatto tutto sul template del parent, il padre di per se non può utilizzare la variabile nella sua classe.

Per collegare le due classi e quindi fare in modo che la classe del padre possa accedere alla classe del figlio, va fatto l'`inject` del figlio nel padre com `ViewChild`:

```ts
@Component({
  selector: 'app-countdown-parent-vc',
  template: `
    <button type="button" (click)="start()">Start</button>
    <button type="button" (click)="stop()">Stop</button>

	<!-- template variable not needed -->
    <app-countdown-timer></app-countdown-timer>
  `,
  styleUrls: ['../assets/demo.css']
})
export class CountdownViewChildParentComponent implements AfterViewInit {

  @ViewChild(CountdownTimerComponent)
  private timerComponent!: CountdownTimerComponent;

  seconds() { return 0; }

  ngAfterViewInit() {
    // Redefine `seconds()` to get from the `CountdownTimerComponent.seconds` ...
    // but wait a tick first to avoid one-time devMode
    // unidirectional-data-flow-violation error
    setTimeout(() => this.seconds = () => this.timerComponent.seconds, 0);
  }

  start() { this.timerComponent.start(); }
  stop() { this.timerComponent.stop(); }
}
```

>Il [[lifecycle hook]] `ngAfterViewInit()` è importante qui perchè il componente figlio non è disponibile fino a quando [[Angular]] non ha mostrato la vista padre, quindi inizialmente mostra 0.
>Poi Angular chiama `ngAfterViewInit` quando ormai è troppo tardi per aggiornare la vista del padre. Il data-flow unidirezionale di Angular previene l'aggiornarsi della view padre nello stesso ciclo, quindi l'app dovrà aspettare un turno prima di poter mostrare i secondi (nell'esempio).
>Viene usato `setTimeout()` per aspettare un `tick` e dopo viene modificato il metodo `seconds()` in modo che prenda i valori futuri dal componente figlio.

