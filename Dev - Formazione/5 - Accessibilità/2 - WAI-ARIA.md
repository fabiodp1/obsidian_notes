Alcune volte mantenere accessibili controlli UI complessi che involvono HTML senza semantica e contenuti dinamici aggiornati tramite JS, è molto difficile. `WAI-ARIA` è una tecnologia che può aiutarci in questo, aggiungendo un altro tipo di semantica che browser e tecnologie assistive possono riconoscere e usare.

---

# WHAT

L'avvento di applicativi sempre più complessi è stato seguito da quello di feature per l'accessibilità.
Per es. l'HTML ha introdotto nuovi tag per aggiungere semantica al markup.
Fra questi anche tentativi di creare elementi come `datepicker` e `range` built-in, ma che avendo un loro styling ben preciso e poco personalizzabili, hanno spinto sempre più sviluppatori e designer ad affidarsi a librerie [JS](JS) per creare elementi più accattivanti e adatti allo scopo.
Ma spesso queste librerie non gestiscono anche la loro accessibilità, risultando in semplici `div` a cui viene applicato dello styling e comportamenti JS.

Il problema è che seppure visualmente e come comportamento questi elementi funzionano, non sono gestiti correttamente dalle tecnologie assistive.

>Per questo motivo nasce `WAI-ARIA` (Web Accessibility Initiative - Accessible Rich Internet  Applications), una specifica creata da `W3C`, che definisce un set di attributi HTML addizionali, da applicare agli elementi  per fornire semantica e migliorarne l'accessibilità.

La specifica definisce 3 principali feature:

- [**Roles**](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles)
	- definiscono cosa un elemento è e fa. Molti di questi sono chiamati `landmark roles`, che in gran parte riproducono la semantica degli elementi HTML built-in, come `role="navigation"` (`<nav>`), `role="banner"` (`<header>`), `role="complementary"` (`<aside>`) o `role="search"` (`<search>`) e altri.

- **Properties**
	- definiscono le proprietà degli elementi, che possono essere usate per dargli semantica o arricchirne il significato. Ad es. `aria-required="true"` specifica che un campo input deve essere riempito per essere valido, mentre `aria-labelledby="label"` permette di mettere un id ad un elemento, per poi referenziarlo come label di qualsiasi cosa vogliamo nella pagina, anche più elementi, non possibile con il semplice `<label for="input">`.

- **States**
	- proprietà speciali che definiscono la condizione attuale degli elementi, come `aria-disabled="true"`, che specifica ad uno screen reader che un campo input è al momento disabilitato. La differenza fra `properties` e `state` è che le prime non cambiano nel tempo, mentre gli state sì, in genere programmaticamente tramite [JS](JS).

>Ciò che è importante sapere sugli attributi `WAI-ARIA` è che non condizionano assolutamente la pagina web, se non l'informazione che viene esposta dall'API di accessibilità del browser. Non modifica ne DOM ne struttura della pagina, nonostante possono essere usati ad es. per la selezione [CSS](CSS).

---

# WHEN

>**NOTA** Usando i corretti elementi [HTML](HTML.md) implicitamente ci vengono forniti i ruoli che sono necessari, per questo vanno *SEMPRE* utilizzate le feature native dell'HTML per fornire le semantiche richieste dagli screen reader per far capire agli utenti cosa sta succedendo.
>`ARIA` interviene quando ciò non è possibile.

Le aree in cui `WAI-ARIA` è utile sono essenzialmente 4:

## Segnaletica / Riferimenti

I valori dell'attributo `role` possono fungere da punti di riferimento che replicano la semantica degli elementi HTML (es. `<nav>`), oppure possono andare oltre e fornire segnali a diverse aree funzionali, come `search`, `tablist`, `tab`, `listbox` ecc.

```html
<search>
  <form>
	<input
	  type="search"
	  name="q"
	  placeholder="Search query"
	  aria-label="Search through site content" />
	<input type="submit" value="Go!" />
  </form>
</search>
```

## Contenuti dinamici

Gli screen reader tendono ad avere difficoltà con i contenuti dinamici. Con `ARIA` possiamo usare `aria-live` per informare gli utenti degli screen reader quando un contenuto viene aggiornato dinamicamente.

```html
<section aria-live="assertive">
  <h1>Random quote</h1>
  <blockquote>
    <p>{{ dynamicContent }}</p>
  </blockquote>
</section>
```

Dal valore dato alla proprietà dipenderà l'urgenza con cui verrà letto il contenuto da parte dello screen reader:

- `off`: il default, l'update non dovrebbe essere annunciato.
- `polite`: deve essere annunciato solo se l'utente è in attesa.
- `assertive`: va annunciato appena possibile.

C'è inoltre da considerare che in questa maniera verrà letta solo la porzione di testo che viene aggiornato, sarebbe utile se venisse letto anche la testata, in modo che l'utente possa ricordare di cosa si sta parlando. Per fare ciò possiamo aggiungere `aria-atomic`:

```html
<section aria-live="assertive" aria-atomic="true">…</section>
```

Questo dirà agli screen reader di leggere l'intero contenuto dell'elemento come un'unica unità atomica, non solo i pezzetti aggiornati.

## Potenziare l'accessibilità via tastiera

Ci sono elementi HTML built-in che hanno accessibilità tramite tastiera in maniera nativa. Quando altri elementi vengono usati con JS per simulare comportamenti simili, l'accessibilità via tastiera e in generale gli screen reader ne soffrono.
Quando non è possibile evitarlo, `WAI-ARIA` fornisce un modo per permettere che venga fatto il focus anche su questi elementi, tramite `tabindex`.

## Accessibilità di controlli non semantici

Quando vengono usati una serie di `<div>` innestati insieme a CSS/JS per creare una feature UI complessa, o quando un controllo nativo viene potenziato usando JS, l'accessibilità ne può soffrire.
In questi casi `ARIA` può aiutare nel fornire ciò che manca con una combinazione di ruoli, come `button`, `listbox`, o `tablist`, e proprietà come `aria-required` o `aria-posinset`, fornendo così maggiori indizi sulle funzionalità.

### Validazione form e alert di errore

```html
<div class="errors" role="alert" aria-relevant="all">
  <ul></ul>
</div>
```

- `role="alert"`: automaticamente trasforma l'elemento in una `live region`, quindi i cambiamenti vengono letti dal reader. Inoltre semanticamente lo identifica come messaggio di allerta, rappresentando una maniera migliore e più accessibile di gestire un alert.
- `aria-relevant="all"`: istruisce lo screen reader per leggere i contenuti della lista di errore nel momento in cui questa cambia. Utile per far sapere all'utente quali errori rimangono, non solo quelli che sono stati aggiunti o rimossi dalla lista.

Possiamo informare i reader del tipo di validazione presente:

```html
<input type="text" name="name" id="name" aria-required="true" />

<input type="number" name="age" id="age" aria-required="true" />
```

