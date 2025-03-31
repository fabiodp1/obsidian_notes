[ARIA](https://www.w3.org/TR/using-aria/) (**Accessible Rich Internet Applications**) definisce le specifiche per la creazione di contenuti web accessibili a persone con disabilità.

Le specifiche oltre a dare delle indicazioni generali, definiscono anche delle regole di base da seguire.

Vedi anche [QUI](https://www.w3.org/WAI/ARIA/apg/patterns/) e .

---

# Regole d'utilizzo

## Prima Regola

>Se è possibile utilizzare un elemento [[HTML]] nativo o attributo con la semantica e comportamento desiderato già `built-in`, piuttosto che utilizzare un elemento generico aggiungendo un ruolo, stato o proprietà ARIA per renderlo accessibile, allora **utilizzalo**.

## Seconda Regola

>NON cambiare la semantica nativa, a meno che non sia necessario:

```html
<h2 role=tab>haeding tab</h2>            === NO ===
<div role=tab><h2>heading tab</h2></div> === YES ===
```

## Terza Regola

>Tutti i controlli interattivi ARIA devono essere utilizzabili via **tastiera**. Ad es. se si sta utilizzando `role=button` l'elemento deve poter ricevere il `focus` e un utente deve poter attivare l'azione associata all'elemento sia tramite il tasto `enter` che `space`.

## Quarta Regola

>Non utilizzare `role="presentation"` o `aria-hidden="true"` su elementi che possono ricevere il `focus`. Il loro utilizzo risulterà in utenti che fanno il *fucus su nulla*.

È preferibile utilizzare `display:none` o `visibility:hidden`, poiché faranno in modo che non si possa fare `focus` sull'elemento e inoltre verrà rimosso dall'albero dell'accessibilità, quindi non sarà più necessario utilizzare `aria-hidden`.

```html
<button role=presentation>press me</button>   === NO ===
<button aria-hidden="true">press me</button>  === NO ===

<div aria-hidden="true">
  <button>press me</button>                   === NO ===
</div>

<button hidden>press me</button>              === YES ===

<div hidden>
  <button>press me</button>                   === YES ===
</div>
```

## Quinta Regola

>Tutti gli elementi interattivi devono avere un **nome accessibile**. Questo avrà un nome accessibile solo quando la proprietà dell'`Accessibility API` `accName` avrà un valore.

Ad esempio nel seguente esempio l'`input type=text` ha una label visibile, ma nessun nome accessibile:

```html
user name <input type="text">

or

<span>user name</span> <input type="text">
```

![](Pasted%20image%2020250331120823.png)

Invece nell'esempio seguente possiede una label visibile E un nome accessibile, poiché l'elemento `input` possiede una label e l'elemento `<label>` viene utilizzato correttamente, associandolo correttamente all'`input`:

```html
<!-- Note: use of for/id or wrapping label around text
and control methods will result in an accessible name -->

<input type="text" aria-label="User Name">

or

<span id="p1">user name</span> <input type="text" aria-labelledby="p1">
```

![](Pasted%20image%2020250331121152.png)

>**NOTA**: l'esempio sopra è per i widget ARIA. Per normali `input` HTML è preferibile seguire la prima regola ARIA e utilizzare l'elemento `label` con un attributo `for` per associare label e `input`.

### `label` ed elementi etichettabili

L'elemento label non può essere utilizzato per fornire un nome accessibile ai `custom controls`, a meno che la label non referenzi un elemento HTML etichettabile:

```html
<!-- HTML input element with combox role -->

<label>
  user name <input type="text" role="combobox">   === YES ===
</label>
```

Un elemento `div` a prescindere dal role che gli viene assegnato, non è un elemento etichettabile:

````html
<!-- HTML div element with combox role -->

<label>
 user name <div  role="combobox"></div>   === NO ===
</label>
```