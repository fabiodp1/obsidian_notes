# Strategie

Angular implementa 2 strategie per gestire la reattività e quindi rilevare le modifiche a livello di singoli componenti:

- `Default`
- `OnPush`

```ts
export enum ChangeDetectionStrategy {
 OnPush = 0,
 Default = 1,
}
```

Inoltre [Angular](Angular) utilizza diverse strategie per decidere se un componente figlio deve essere controllato quando controlla le modifiche del componente padre.
Quando il padre viene controllato, tutte le direttive figlie sono coinvolte e influenzate dalla strategia definita per quel componente.

>Una volta definita, una strategia non può essere cambiata a runtime.

La strategia di `Default`, internamente denominata `CheckAlways`, implica il rilevamento automatico e regolare delle modifiche di un componente a meno che la view non sia esplicitamente scollegata.

Nella strategia `OnPush` invece, internamente denominata `CheckOnce`, il rilevamento delle modifiche viene saltato a meno che il componente non sia contrassegnato come `Dirty`.

>Angular implementa meccanismi automatici per contrassegnare un componente com `Dirty`. Se necessario può essere fatto anche manualmente utilizzando il metodo `markForCheck` esposto da `ChangeDetectorRef`.

## In depth

Quando [Angular](Angular) crea un'istanza di un componente, imposta un flag corrispondente nell'istanza `LView`, che rappresenta la vista del componente.
Questo flag può essere `CeckAlways` o `Dirty`:

![](Pasted%20image%2020250224122217.png)

Questo vuol dire che tutte le istanze `LView` create per quel componente avrà uno di questi flag impostati.
Nel caso di strategia `OnPush`, il flag `Dirty` verrà disattivato automaticamente dopo il primo passaggio di rilevamento delle modifiche.