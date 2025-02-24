Angular implementa 2 strategie per gestire la reattività e quindi rilevare le modifiche a livello di singoli componenti:

- `Default`
- `OnPush`

```ts
export enum ChangeDetectionStrategy {
 OnPush = 0,
 Default = 1,
}
```

Inoltre [Angular](Angular) utilizza diverse strategie per decidere se un componente figlio deve essere controllato quando controlla le modifiche 