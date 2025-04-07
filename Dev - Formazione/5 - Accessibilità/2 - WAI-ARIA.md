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

- **Roles**
	- definiscono cosa un elemento è e fa. Molti di questi sono chiamati `landmark roles`, che in gran parte riproducono la semantica degli elementi HTML built-in, come `role="navigation"` (`<nav>`), `role="banner"` (`<header>`), `role="complementary"` (`<aside>`) o `role="search"` (`<search>`) e altri.

- **Properties**
	- 
