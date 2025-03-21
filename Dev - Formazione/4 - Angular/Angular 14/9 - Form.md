[Angular](Angular) supporta 2 tipi di approccio per gestire form interattivi, `template syntax` e direttive.

# Template-driven forms

Fanno affidamento sulle direttive definite nel `FormsModule`:

- `NgModel` permette il two-way binding
- `NgForm` crea un'istanza di `FormGroup` top-level e ne fa il bind con l'elemento `<form>` per tenere traccia i valori aggregati del form e lo stato di validazione. Nel momento in cui `FormModule` viene importato, questa direttiva diventa disponibile su ogni tag `<form>`, non c'è bisogno di un selettore speciale.
- `NgModelGroup` Crea e fa il bind di un'istanza di `FormGroup` ad un elemento del `DOM`.

# Form status

Con l'import di `FormsModule` Angular automaticamente crea e collega la direttiva `NgForm` al tag `<form>` (`NgForm` ha come selettore 'form' quindi automaticamente si attacca agli elementi form).

Per accedere a `NgForm` e il form status, bisogna dichiarare una `template reference variable`:

```ts
<form #heroForm="ngForm">
```

Fatto questo la template variable `heroForm` adesso è una reference all'istanza della direttiva `NgForm` che governa il form per intero.

## Nominare gli elementi

Utilizzando il `[(ngModel)]` va definito l'attributo `name`, [[Angular]] lo userà per registrare l'elemento con la direttiva `NgForm` attaccato all'elemento `<form>`:

```ts
<div class="form-group">
  <label for="name">Name</label>
  <input type="text" class="form-control" id="name"
         required
         [(ngModel)]="model.name" name="name">
</div>
```

## Control states

Aggiungendo la direttiva `NgModel` ad un controllo, aggiunge a questo delle classi [CSS](CSS) che ne definiscono lo stato. Queste possono essere utilizzate per cambiare lo stile di un controllo in base al suo stato.

queste sono le classi (ogni coppia ha i due valori opposti fra loro):

- `ng-touched` / `ng-untouched` se il control è stato visitato
- `ng-dirty` / `ng-pristine` se il valore del controllo è stato cambiato
- `ng-valid` / `ng-invalid` se il valore è valido

>la classe `ng-submitted` invece viene applicata al submit e solo all'elemento `form`, non agli elementi contenuti.

```ts
<input … class="form-control ng-untouched ng-pristine ng-valid" …>
```

In questa maniera è possibile applicare degli stili in base alla classe aggiunta:

```ts
// forms.css 

.ng-valid[required], .ng-valid.required  {
  border-left: 5px solid #42A948; /* green */
}

.ng-invalid:not(form)  {
  border-left: 5px solid #a94442; /* red */
}
```

## Mostrare messaggi di errore

Un modo per gestire i messaggi di errore è utilizzare le `template reference variable` per poter accedere ai controlli dei campi dall'interno del template:

```ts
<label for="name">Name</label>
<input type="text" class="form-control" id="name"
       required
       [(ngModel)]="model.name" name="name"
       #name="ngModel">
<div [hidden]="name.valid || name.pristine"
     class="alert alert-danger">
  Name is required
</div>
```

>la `template reference variable` `#name` viene settata a `ngModel` perché quello è il valore della proprietà `NgModel.exportAs`. Questa proprietà dice ad Angular come collegare la reference variable a una direttiva.

## Resettare il form

Per resettare il form basta chiamare nel template il metodo `reset()` della `reference variable` `heroForm`:

```ts
<button type="button" class="btn btn-default" (click)="newHero(); heroForm.reset()">New Hero</button>
```

## Submit del form

Per gestire l'evento di submit basta fare il bind dell'evento `ngSubmit`:

```ts
<form (ngSubmit)="onSubmit()" #heroForm="ngForm">
```

L'evento viene lanciato al click del bottone con `type="submit"` e possiamo usare la `reference variable` per gestire l'abilitazione del click:

```ts
<button type="submit" class="btn btn-success" [disabled]="!heroForm.form.valid">Submit</button>
```