[ARIA](https://www.w3.org/TR/using-aria/) (**Accessible Rich Internet Applications**) definisce le specifiche per la creazione di contenuti web accessibili a persone con disabilità.

# Regole

Le specifiche oltre a dare delle indicazioni generali, definiscono anche delle regole di base da seguire:

1. Se è possibile utilizzare un elemento [[HTML]] nativo o attributo con la semantica e comportamento desiderato già `built-in`, piuttosto che utilizzare un elemento generico aggiungendo un ruolo, stato o proprietà ARIA per renderlo accessibile, allora **utilizzalo**.
2. NON cambiare la semantica nativa, a meno che non sia necessario:

```html
<h2 role=tab>haeding tab</h2>            === NO ===
<div role=tab><h2>heading tab</h2></div> === YES ===
```

3. Tutti i controlli interattivi ARIA devono essere utilizzabili via **tastiera**. Ad es. se si sta utilizzando `role=button` l'elemento deve poter ricevere il `focus` e un utente deve poter attivare l'azione associata all'elemento sia tramite il tasto `enter` che `space`.
4. Non utilizzare `role="presentation"` o `aria-hidden="true"` su elementi che possono ricevere il `focus`. Il loro utilizzo risulterà in utenti che fanno il *fucus su nulla*.

```html
<button role=presentation>press me</button>   === NO ===
<button aria-hidden="true">press me</button>  === NO ===

<div aria-hidden="true">
  <button>press me</button>                   === NO ===
</div>
```