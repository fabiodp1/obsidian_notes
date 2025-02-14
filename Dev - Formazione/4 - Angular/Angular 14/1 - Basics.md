# Architettura

l'architettura di Angular si basa su certi concetti fondamentali. I mattoni di Angular sono i componenti che vengono organizzati in [[NgModules]].
Questi li raccolgono il relativo codice in set funzionali.

>Un applictivo in [[Angular]] viene definita da un set di [[NgModules]]. Un applicativo ha SEMPRE almeno un modulo di root che ne abilita l'avvio e tipicamente possiede altri `feature modules`.

- I componenti definiscono le `view`
- I componenti utilizzano `service`, che forniscono specifiche funzionalità