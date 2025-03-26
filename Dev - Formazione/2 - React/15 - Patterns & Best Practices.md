[Your React App Isn’t Great — Here’s Why. | by Pierre G. | Dec, 2024 | Level Up Coding](https://levelup.gitconnected.com/your-react-app-isnt-great-here-s-why-5eb61b3f110b)

# Code splitting
[Code-Splitting – React](https://legacy.reactjs.org/docs/code-splitting.html)

Per ottimizzare le performance è importante che fra componenti le callback vengano passate utilizzando l'hook `useCallback`, in modo da evitare re-render inutili.
Inoltre se il nostro componente è un context provider, è buona pratica passare come value la versione memoized:

```ts
function AuthProvider(props) {
  const {
    data: user,
    error,
    ...
  } = useAsync()

  const login = React.useCallback(
    form => auth.login(form).then(user => setData(user)),
    [setData],
  )

  const register = React.useCallback(
    form => auth.register(form).then(user => setData(user)),
    [setData],
  )

  const logout = React.useCallback(() => {
    auth.logout()
    setData(null)
  }, [setData])

  const value = React.useMemo(() => ({user, login, logout, register}), [
    login,
    logout,
    register,
    user,
  ])

  if (isLoading || isIdle) {
    return <FullPageSpinner />
  }

  if (isError) {
    return <FullPageErrorFallback error={error} />
  }

  if (isSuccess) {
    return <AuthContext.Provider value={value} {...props} />
  }

  throw new Error(`Unhandled status: ${status}`)
}
```

---

# Compound components

I [[compound components]] sono un modo in [[React]] per organizzare i componenti in modo da renderli flessibili e componibili fra loro.
Non sono pensati per essere `standalone` ma per funzionare uniti fra loro.

Ad es. considerando questo componente:

```ts
<LoginFormModal
  onSubmit={handleSubmit}
  modalTitle="Modal title"
  modalLabelText="Modal label (for screen readers)"
  submitButton={<button>Submit form</button>}
  openButton={<button>Open Modal</button>}
/>
```

Sembra abbastanza semplice, ma come possiamo fare per renderlo flessibile, ad es. se vogliamo riutilizzare la Modal per mostrare altro, o se vogliamo spostare il pulsante per il close in basso, dovremmo passare un'altra prop che gli dica di mostrarlo in basso. 
Questi problemi sono più facilmente e elegantemente risolvibili con i **compound component**.

```jsx
<Modal>
  <ModalOpenButton>
    <button>Open Modal</button>
  </ModalOpenButton>
  <ModalContents aria-label="Modal label (for screen readers)">
    <ModalDismissButton>
      <button>Close Modal</button>
    </ModalDismissButton>
    <h3>Modal title</h3>
    <div>Some great contents of the modal</div>
  </ModalContents>
</Modal>
```

Ecco come è possibile organizzare i componenti per farli diventare un compound:

```ts
const ModalContext = React.createContext()

const useModal = () => {
  const context = React.useContext(ModalContext)

  if (!context) throw new Error('To use the modal context you need a provider')
  return context
}

const Modal = props => {
  const [isOpen, setIsOpen] = React.useState(false)
  return <ModalContext.Provider value={{isOpen, setIsOpen}} {...props} />
}

const ModalDismissButton = ({children}) => {
  const {setIsOpen} = useModal()
  return React.cloneElement(children, {onClick: () => setIsOpen(false)})
}

const ModalOpenButton = ({children}) => {
  const {setIsOpen} = useModal()
  return React.cloneElement(children, {onClick: () => setIsOpen(true)})
}

const ModalContents = ({children, ...props}) => {
  const {isOpen, setIsOpen} = useModal()

  return (
    <Dialog isOpen={isOpen} onDismiss={() => setIsOpen(false)} {...props}>
      {children}
    </Dialog>
  )
}

export {ModalContents, ModalDismissButton, ModalOpenButton, Modal, useModal}
```

>NOTA [[cloneElement]] prende l'elemento che gli viene passato e ne restituisce il clone, con la possibilità di assegnargli dei valori predefiniti alle prop.