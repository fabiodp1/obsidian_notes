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

