# Architettura

l'architettura di Angular si basa su certi concetti fondamentali. I mattoni di Angular sono i componenti che vengono organizzati in `NgModules`.
Questi li raccolgono il relativo codice in set funzionali.

>Un applicativo in Angular viene definito da un set di `NgModules`. Un applicativo ha SEMPRE almeno un modulo di root che ne abilita l'avvio e tipicamente possiede altri `feature modules`.

- I componenti definiscono le `view`
- I componenti utilizzano `service`, che forniscono specifiche funzionalità non direttamente legate alle view. I service provider possono essere `injected` nel componente come dipendenza, rendendo il codice modulare, riutilizzabile ed efficiente.

I moduli, i componenti e i servizi sono classi che utilizzano `decorators`. Questi ne indicano il tipo e forniscono metadati che dicono ad Angular come vanno utilizzati.

- I metadati per una classe componente fanno in modo che questo venga associato ad un template che definisce una `view`.
- I metadati per una classe service forniscono le info di cui ha bisogno Angular per renderlo disponibile agli altri componenti attraverso la [[dependency injection]].

Un applicativo normalmente possiede diverse view arrangiate in maniera gerarchica. Angular fornisce un service `Router` che permette la navigazione fra view. 