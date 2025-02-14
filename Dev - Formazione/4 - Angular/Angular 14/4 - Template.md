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
- `( )` dalla view alla sorgente
- `[( )]` two way binding