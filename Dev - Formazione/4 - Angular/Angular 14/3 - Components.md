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

Un componente figlio per dichiarare una prop utilizza il [[decorator]] `@Input`:

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
La proprietà che si occuperà dell'emissione dell'evento è un `output property`, utilizzando il [[decorator]]
