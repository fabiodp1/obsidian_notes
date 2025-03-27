# Component Props

Il modo corretto di definire il type di un `Functional Component` in React è utilizzando il type `React.FC`.

>In questa maniera rendiamo chiaro che stiamo definendo un `functional component` e verranno riconosciute e suggerite tutte le prop standard come `children` ecc.

```tsx title:Todos.tsx
import React from 'react';

const Todos: React.FC = (props) => {
  return (
    <ul>
      { props.children }
    </ul>
  )
}
```

A questo tramite `Generic` possiamo aggiungere le nostre prop:

```tsx title:Todos.tsx
interface IMyType {
  name: string;
}

const Todos: React.FC<IMyType> = (props) => {    <===
  return (
    <h3>{ props.name }</h3>                      <===
    <ul>
      { props.children }
    </ul>
  )
}
```