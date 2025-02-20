[Angular](Angular) supporta 2 tipi di approccio per gestire form interattivi, `template syntax` e direttive.

# Template-driven forms

Fanno affidamento sulle direttive definite nel `FormsModule`:

- `NgModel` permette il two-way binding
- `NgForm` crea un'istanza di `FormGroup` top-level e ne fa il bind con l'elemento `<form>` per tenere traccia i valori aggregati del form e lo stato di validazione. Nel momento in cui `FormModule` viene importato, questa direttiva diventa disponibile su ogni tag `<form>`, non c'è bisogno di un selettore speciale.
- `NgModelGroup` Crea e fa il bind di un'istanza di `FormGroup` ad un elemento del `DOM`.