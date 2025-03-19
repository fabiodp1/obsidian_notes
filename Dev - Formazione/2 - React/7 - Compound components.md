I [[compound components]] sono un modo in [[React]] per organizzare i componenti in modo da renderli flessibili e componibili fra loro.
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