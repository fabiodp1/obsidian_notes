Ogni componente è formato da:
- Un template [[HTML]]
- Una classe [[TypeScript]] che ne definisce la logica
- Un selettore [[CSS]] che definisce come il componente viene usato in un template
- Opzionale, degli stili [[CSS]] da applicare al template

# Creazione

Per creare un componente bisogna prima rispettare questi requisiti:
- Aver installato l'[Angular CLI](https://v14.angular.io/guide/setup-local#install-the-angular-cli)
- Aver [creato un workspace Angular](https://v14.angular.io/guide/setup-local#create-a-workspace-and-initial-application)

Il miglior modo per creare un componente è utilizzare la CLI lanciando il comando

```terminal
ng generate component <component-name>
```

Di default questo comando andrà a creare:
- Un directory con il nome del componente `<component-name>.component.ts`
- Il file [[TypeScript]] del componente `<component-name>.component.html`
- Il file template [[HTML]] `<component-name>.component.css`
- Un file per il test `<component-name>.component.spec.ts`

>È possibile definire come la CLI farà lo scaffolding del componente utilizzando specifici [comandi](https://v14.angular.io/cli/generate#component-command) È possibile creare i componenti anche manualmente, ma la CLI è consigliata.

# Lifecycle hooks

Per utilizzare il `lifecycle hooks` di [[Angular]] il componente deve implementare la relativa interfaccia:

```ts

```