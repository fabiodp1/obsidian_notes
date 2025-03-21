# Interpolation

```ts
<p>{{title}}</p>
<div><img alt="item" src="{{itemImageUrl}}"></div>
```

# Template statements

Per rispondere ad eventi nel template


```ts
<button type="button" (click)="deleteHero()">Delete hero</button>
```

>Da notare che a differenza di altri framework, il metodo passato viene invocato

Per ottenere l'evento basta usare la prop built in `$event`:

```ts
<button type="button" (click)="onSave($event)">Save</button>
<button type="button" *ngFor="let hero of heroes" (click)="deleteHero(hero)">{{hero.name}}</button>
<form #heroForm (ngSubmit)="onSubmit(heroForm)"> ... </form>
```

Nell'esempio sopra *onSave()* prende l'oggetto proprio del template `$event` come argomento, mentre *deleteHero()* prende una `template input variable`, *hero*, e *onSubmit()* prende una `template reference variable` *#heroForm*.

# Binding syntax

Il `data binding` automaticamente tiene aggiornata la pagina sulla base dello state applicativo.
Si fa utilizzando le parentesi quadre `[ ]`.

```ts
<input [disabled]="condition ? true : false"> // property binding
<input [attr.disabled]="condition ? 'disabled' : null"> // attribute binding
```

la differenza fra quelli sopra è che il `property binding` deve avere come valore un boolean, mentre il corrispettivo `attribute binding` si basa sul fatto che il proprio valore sia *null* o meno.

>Normalmente è più facile da leggere e performante usare il `property binding`

## Tipi di data binding

- Dalla sorgente alla view: *Interpolazione*, *Property*, *Attribute*, *Class*, *Style*

```ts
{{expression}} 
[target]="expression"
```

- Dalla view alla sorgente: *Event*

```ts
(target)="statement"
```

- In entrambe le direzioni (two way binding)

```ts
[(target)]="expression"
```

Il `target` di un binding è una proprietà o un evento, che viene circondate dalle `[ ]`, parentesi `( )` o entrambe `[( )]`

- `[ ]` dalla sorgente alla view;

```ts
<img [alt]="hero.name" [src]="heroImageUrl">
<app-hero-detail [hero]="currentHero"></app-hero-detail>
<div [ngClass]="{'special': isSpecial}"></div>
```

- `( )` dalla view alla sorgente

```ts
<button type="button" (click)="onSave()">Save</button>
<app-hero-detail (deleteRequest)="deleteHero()"></app-hero-detail>
<div (myClick)="clicked=$event" clickable>click me</div>
```

- `[( )]` two way binding

```ts
<input [(ngModel)]="name">
```

# Property binding

Permette di dare un valore alle proprietà e direttive [[HTML]]. Usato ad es. per cambiare le prop di un bottone o passare proprietà ad un componente figlio.

```ts
<img alt="item" [src]="itemImageUrl">
```

Le `[ ]` identificano quella proprietà come `target` e indicando ad [[Angular]] che il valore alla destra va valorizzato.

Per settare una proprietà di una direttiva si fa lo stesso:

```ts
<p [ngClass]="classes">[ngClass] binding to the classes property making this blue</p>
```

# Attribute, class e style binding

## Attribute

Serve a settare il valore di attributi per migliorare l'accessibilità o lo style in maniera dinamica, gestendo simultaneamente diverse classi [[CSS]].

La sintassi è simile a quella del `property binding`, ma invece di avere una proprietà fra le parentesi, si precede il nome dell'attributo con il prefisso `attr.`.

```ts
<p [attr.attribute-you-are-targeting]="expression"></p>
```

>Nel momento in cui l'espressione si risolve in *null* o *undefined*, [[Angular]] rimuove completamente l'attributo.

>Un esempio in cui viene utilizzato è per gestire l'attributo `colspan` di un *td*. Questo non è da confondere con `colSpan` che invece è una proprietà.

## Class

- `Bind ad una singola classe`, la calsse viene aggiunta solo se *onSale* è truthy:

```ts
[class.sale]="onSale"
```

- `Bind a più classi`, l'espressione può essere:
	- una stringa di classi
	- un oggetto con i nomi delle classi come chiavi ed espressioni truthy o falsy come valori. In questo caso Angular aggiungera la classe solo se l'espressione è truthy
	- un array di nomi di classi

```ts
[class]="classExpression"
```

## Style

- `Bind a un singolo stile`, come per class:
	- `[style.width]="width"` with deve avere l'unità di misura, es. `100px`
	- `[style.width.px]="width"` non è necessaria l'unità di misura

```ts
// Entrambe corretti
<nav [style.background-color]="expression/value"></nav>
<nav [style.backgroundColor]="expression/value"></nav>
```

- `Bind di più stili`, l'espressione può essere:
	- Una stringa con la lista degli stili es. `"width: 100px; height: 100px; background-color: cornflowerblue;"`
	- Un oggetto avente come key il nome dello stile es. `{width: '100px', height: '100px', backgroundColor: 'cornflowerblue'}`

# Two-way binding

Il `two-way binding` permette ai componenti di condividere dati. Si utilizzano per ascoltare eventi e simultaneamente aggiornare i valori fra padre e figlio.

```ts
<app-sizer [(size)]="fontSizePx"></app-sizer>
```

[Angular - Two-way binding](https://v14.angular.io/guide/two-way-binding#how-two-way-binding-works)