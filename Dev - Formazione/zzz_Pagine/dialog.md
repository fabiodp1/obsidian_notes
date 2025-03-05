È un elemento [[HTML]] native, che in genere viene utilizzato per mostrare [[modal]].
Può essere aperto tramite l'attributo `open` ma ===settandolo direttamente a true, si perderà la feature del backdrop del dialog===.
Per questo è meglio aprirlo programmaticamente tramite `.showModal()`.

>Bisogna fare attenzione a gestire la digitazione del tasto `Esc` che di default chiude il dialog aperto, ma se questo non viene riflesso sullo state applicativo, potrebbe creare problemi alla presentation. Per questo possiamo usare l'evento nativo `onClose` emesso dal browser, per poterlo gestire.