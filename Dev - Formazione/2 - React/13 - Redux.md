# WHAT

>[[Redux]] è un sistema di gestione dello stato trasversalmente ai componenti e il resto dell'app.

Permette quindi di gestire lo [state](state), quindi dati che al cambiamento dovrebbero coinvolgere la [UI](UI).
Possiamo suddividere gli `state` in 3 tipologie:

- **`Local State`**
	- Lo stato che appartiene ad un **singolo componente**, in genere gestito usando `useState` e `useReducer`.
	
- **`Cross-component State`**
	- State che coinvolge **diversi componenti**, ad es. per l'apertura o chiusura di un componente Modal. Anche qui si tratta di state creato con `useState` o `useReducer` e passato attraverso `prop drilling`.
	
- **`App-wide state`**
	- State che coinvolge **tutto l'applicativo**, ad es. il theme scelto e lo stato di autenticazione di un utente. Anche questo richiede `prop drilling`

>Gli ultimi due possono essere più complessi da gestire attraverso il `prop drilling`, rispetto al local state, per questo spesso viene usato il `Context`. [Redux](Redux) aiuta a risolvere lo stesso tipo di problema.

---

# WHY

Se [Redux](Redux) è semplicemente un'alternativa ai [5 - Context](5%20-%20Context.md), perché dovremmo usarlo?

## Contro del Context

Parliamo di potenziali 'contro', perché potrebbe essere che nella nostra app non rappresentino un problema.

- In certi casi potremmo avere **elevata complessità di setup e gestione**
	- Soprattutto in grandi applicazioni, potremmo avere multipli `Context` per la gestione di diversi tipi di state. Ovviamente potremmo anche crearne uno unico, ma diventerebbe un `Provider` molto grande e difficile da gestire, da manutenere e avrebbe anche state che non hanno relazione diretta fra loro.
- Problemi di **performance** per cambiamenti frequenti allo stato
	- È consigliato soprattutto per state che vengono aggiornati poco frequentemente, ad es. cambio theme e autenticazione, ma no se i dati cambiano frequentemente. **Non può in tutti gli scenari essere utilizzato** come `store`.

Librerie come `Redux` nascono per risolvere queste criticità.

---

# HOW

[[Redux]] nasce per poter creare uno `Central Data Store`, avere tutti dati (state) centralizzati in **un unico** store.
Questo potrebbe sembrare difficile da mantenere, come per il Context, ma in realtà non abbiamo bisogno ogni volta di gestire direttamente l'intero store,

I componenti che hanno bisogno di utilizzare lo store, fanno una `Subscription` ad esso e ogni volta che lo state all'interno dello store cambia, il componente verrà notificato per reagire di conseguenza.

>I componenti non manipolano **===MAI===** lo stato dello store in maniera diretta, per fare ciò utilizzano (attraverso le `Action`) i **Reducer**.

I `Reducer` sono delle funzioni che si occupano di modificare lo state dello store e vengono attivati da delle funzione di `Dispatch` usati dai componenti, le `Action`.
Le `Action` sono degli oggetti che descrivono le azioni che i `Reducer` devono compiere. 
Quindi il componente chiama l'`Action`, [[React]] la passa al `Reducer` che genererà il nuovo `state` e tutti i componenti che hanno fatto la `Subscription` a quello state verranno informati.