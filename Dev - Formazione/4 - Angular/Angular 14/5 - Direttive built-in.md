Le direttive sono classi che aggiungono comportamenti agli elementi dell'applicativo. Vengono usati per gestire form, liste, stili e viste.

I tipi di direttiva in [[Angular]] sono:

- `Components`: con cui si definiscono i componenti
- `Attribute`: cambiano il comportamento e l'aspetto di un elemento, componente o un'altra direttiva.
- `Structural`: per cambiare il layout del [[DOM]] aggiungendo e rimuovendo elementi.

# Attribute

Le direttive attributo più comuni sono:

- `NgClass` aggiunge e rimuove un set di classi [[CSS]]
- `NgStyle` aggiunge e rimuove un set di style [[HTML]]
- `NgModel` aggiunge il [[two-way data binding]] ad un elemento form dell'[[HTML]]

>Molti `NgModules` come `RouterModule` e `FormsModule` definiscono le proprie direttive attributo.

## `NgClass`

può essere usato con un'espressione:

```ts
<!-- toggle the "special" class on/off with a property -->
<div [ngClass]="isSpecial ? 'special' : ''">This div is special</div>
```

 o con un metodo:

```ts
// .ts
currentClasses: Record<string, boolean> = {};
/* . . . */
setCurrentClasses() {
  // CSS classes: added/removed per current state of component properties
  this.currentClasses =  {
    saveable: this.canSave,
    modified: !this.isUnchanged,
    special:  this.isSpecial
  };
}

// .html
<div [ngClass]="currentClasses">This div is initially saveable, unchanged, and special.</div>
```

Nell'esempio sopra il metodo verrà chiamato all'interno di `ngOnInit()`.

## `NgStyle`

Come `NgClass` può essere usato tramite metodo:

```ts
// .ts
currentStyles: Record<string, string> = {};
/* . . . */
setCurrentStyles() {
  // CSS styles: set per current state of component properties
  this.currentStyles = {
    'font-style':  this.canSave      ? 'italic' : 'normal',
    'font-weight': !this.isUnchanged ? 'bold'   : 'normal',
    'font-size':   this.isSpecial    ? '24px'   : '12px'
  };
}

// .html
<div [ngStyle]="currentStyles">
  This div is initially italic, normal weight, and extra large (24px).
</div>
```

Anche qui da utilizzare in `ngOnInit()` o quando vogliamo aggiornare gli stili.

## `NgModel`

Per utilizzarlo bisogna prima di tutto importare `FormModule` fra i moduli:

```ts
// app.module.ts

import { FormsModule } from '@angular/forms'; // <--- JavaScript import from Angular
/* . . . */
@NgModule({
  /* . . . */

  imports: [
    BrowserModule,
    FormsModule // <--- import into the NgModule
  ],
  /* . . . */
})
export class AppModule { }
```

Poi aggiungere `[(ngModel)]` su un elemento form dell'[[HTML]] (es. un campo di input) e settarlo come uguale alla proprietà da bindare:

```ts
<label for="example-ngModel">[(ngModel)]:</label>
<input [(ngModel)]="currentItem.name" id="example-ngModel">
```

È anche possibile personalizzarne il comportamento utilizzandone la forma estesa:

```ts
<input [ngModel]="currentItem.name" (ngModelChange)="setUppercaseName($event)" id="example-uppercase">
```

`NgModel` funziona con elementi che supportano [ControlValueAccessor](https://v14.angular.io/api/forms/ControlValueAccessor). Angular lo fornisce per la maggior parte degli elementi form [[HTML]] base.
Per applicarlo ad elementi non form o componenti di terze parti bisogna scriverne il `value accessor` [DefaultValueAccessor](https://v14.angular.io/api/forms/DefaultValueAccessor).

>Scrivendo un componente non è necessario utilizzare `value accessor` o `NgModel` finché il valore e le prop evento vengono nominate secondo la [two-way binding syntax](https://v14.angular.io/guide/two-way-binding#how-two-way-binding-works) di Angular.

# Structural

## `NgIf` (`NgIfElse`)

```ts
<app-item-detail *ngIf="isActive" [item]="item"></app-item-detail>
```

## `NgFor`

```ts
<div *ngFor="let item of items">{{item.name}}</div>
```

```ts
<div *ngFor="let item of items; let i=index">{{i + 1}} - {{item.name}}</div>
```

Per evitare re-rendering inutili in un for loop, è possibile utilizzare la proprietà `trackBy` di `*ngFor`. Angular farà il re-rendering solo degli elementi che sono cambiati:

1. aggiungere al componente un metodo che ritorni il valore che di cui `NgFor` dovrà tenere traccia attraverso `trackBy`, deve essere qualcosa di univoco come un id:

```ts
trackByItems(index: number, item: Item): number { return item.id; }
```

1. nell'espressione settare il `trackBy` al metodo creato:

```ts
<div *ngFor="let item of items; trackBy: trackByItems">
  ({{item.id}}) {{item.name}}
</div>
```

>Se si ha bisogno di utilizzare gli attributi per mostrare o manipolare degli elementi del [[DOM]] ma non vogliamo che l'elemento [[HTML]] compaia nel [[DOM]] (un po come `<template>` di Vue), possiamo usare `<ng-container>`:

```ts
<p>
  I turned the corner
  <ng-container *ngIf="hero">
    and saw {{hero.name}}. I waved
  </ng-container>
  and continued on my way.
</p>
```

## `NgSwitch`

È un set di 3 direttive:

- `NgSwitch`
- `NgSwitchCase`
- `NgSwitchDefault`

```ts
<div [ngSwitch]="currentItem.feature">
  <app-stout-item    *ngSwitchCase="'stout'"    [item]="currentItem"></app-stout-item>
  <app-device-item   *ngSwitchCase="'slim'"     [item]="currentItem"></app-device-item>
  <app-lost-item     *ngSwitchCase="'vintage'"  [item]="currentItem"></app-lost-item>
  <app-best-item     *ngSwitchCase="'bright'"   [item]="currentItem"></app-best-item>
<!-- . . . -->
  <app-unknown-item  *ngSwitchDefault           [item]="currentItem"></app-unknown-item>
</div>
```