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
<input [disabled]="condition ? true : false">
<input [attr.disabled]="condition ? 'disabled' : null">
```

la differenza fra quelli sopra è che il `property binding` deve avere come valore un boolean, mentre il corrispettivo `attribute binding` si basa sul fatto che il proprio valore sia *null* o meno.