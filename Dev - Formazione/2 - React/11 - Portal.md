In certe situazioni vorremmo che certi elementi del [[DOM]] che dichiariamo all'interno di componenti, venissero renderizzati altrove, ad es. per una modal ha senso che non si trovi come nodo di un componente ma su un layer superiore, ad es. direttamente sotto il body o il nodo root.

È questo tipo di problema che risolve la feature [[Portal]] tramite il metodo [[createPortal]]. Fa parte della libreria [[react-dom]].

```tsx
import { createPortal } from 'react-dom';

export default function MyModal() {
  return createPortal(
    <dialog ref={dialog}>
	{...}
	</dialog>
  ),
  document.getElementById('modal')
}

// index.html
//...
<body>
  <div id="modal"></div>
  <div id="root"></div>
  <!-- .... -->
```