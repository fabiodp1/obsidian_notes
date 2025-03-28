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

# Context

## createContext

```tsx title:todos-context.tsx
interface ITodoContext {
  items: string[];
  addTodo: (text: string) => void,
  removeTodo: (id: string) => void
}

const TodosContext = React.createContext<ITodoContext>({
  items: [],
  addTodo: () => {},
  removeTodo: (id: string) => {}
})
```

## Context Provider

```tsx title:todos-context.tsx
cosnt TodosContextProvider: React.FC = (props) => {
  const [todos, setTodos] = useState<string[]>([]);

  const addTodoHandler = (todoText: string) => {
    ...
  }

  const removeTodoHandler = (id: string) => {
    ...
  }

  const contextValue: ITodoContext = {
    items: todos,
    addTodo: addTodoHandler,
    removeTodo: removeTodoHandler
  }

  return (
    <TodosContext.Provider value={ contextValue }>
      { props.children }
    </TodosContext.Provider>
  )
};

export default TodosContextProvider;
```