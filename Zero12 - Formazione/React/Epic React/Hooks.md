In React gli [[hook]] vengono utilizzati per la gestione dello state, i più comuni sono:
- [[useState]]
- [[useEffect]]
- [[useContext]]
- [[useRef]]
- [[useReducer]]
Tutti questi possono essere utilizzati all'interno di un componente React per storare dati (useState) o compiere azioni (side-effects).

Ognuno di questi hook possiede una propria [[API]], alcune ritornano un valore (come useRef e useContext), altre coppie di valori (useState, useReducer) e altri non ritornano niente (useEffect).

# [[useState]]
E' una funzione che accetta un parametro come initial state e ritorna una coppia di valori, ritornando una array con 2 valori (utilizzato con la destrutturazione per dare un nome ai due valori), lo stato e una funzione setter per cambiare lo stato (per convenzione viene nominata iniziando con 'set').