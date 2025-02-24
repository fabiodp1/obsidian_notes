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

