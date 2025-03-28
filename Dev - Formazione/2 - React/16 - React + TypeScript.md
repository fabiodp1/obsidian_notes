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

Tramite questo, via `Generic`, possiamo dichiarare le nostre prop:

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

# FormEvent

L'evento di `submit` può essere tipizzato dal type fornito da [React](React.md) `React.FormEvent`:

```tsx title:NewTodo.tsx
...
const submitHandler = (event: React.FormEvent) => {   <===
  event.preventDefault();
  ...
}

return (
  <form onSubmit={ submitHandler }>
    ...
  </form>
)
```

# useRef

```tsx
const todoTextInputRef = useRef<HTMLInputElement>(null);       <===

return (
  <input type="text" ref={ todoTextInputRef } />
)
```

# Function Props

```tsx title:NewTodo.tsx
interface INewTodoProps {
  onAddTodo: (text: string) => void;
}

const NewTodo: React.FC<INewTodoProps> = (props) => {
  ...
  const submitHandler = (event: React.FromEvent) => {
    ...
    props.onAddTodo(enteredText);  
  } 
}
```

# useState

```tsx
const [todos, setTodos] = useState<string[]>([]);
```