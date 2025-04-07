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

Le aree in cui `WAI-ARIA` è utile sono essenzialmente 4:

## Segnaletica / Riferimenti

I valori dell'attributo `role` possono fungere da punti di riferimento che replicano la semantica degli elementi HTML (es. `<nav>`), oppure possono andare oltre e fornire segnali a diverse aree funzionali, come `search`, `tablist`, `tab`, `listbox` ecc.

## Contenuti dinamici

Gli screen reader tendono ad avere difficoltà con i contenuti dinamici. Con `ARIA` possiamo usare `aria-live` per informare gli utenti degli screen reader quando un contenuto viene aggiornato dinamicamente.

## Potenziare l'accessibilità via tastiera

