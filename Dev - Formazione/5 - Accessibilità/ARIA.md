`ARIA` (Accessible Rich Internet Applications) definisce le specifiche per la creazione di contenuti web accessibili a persone con disabilità.

# Regole

Le specifiche oltre a dare delle indicazioni generali, definiscono anche delle regole di base da seguire:

1. Se è possibile utilizzare un elemento [[HTML]] nativo o attributo con la semantica e comportamento desiderato già `built-in`, piuttosto che utilizzare un elemento generico aggiungendo un ruolo, stato o proprietà ARIA per renderlo accessibile, allora **utilizzalo**.
2. NON cambiare la semantica nativa, a meno che non sia necessario:

```html
<h2 role=tab>haeding tab</h2>            === NO ===
<div role=tab><h2>heading tab</h2></div> === YES ===
```

