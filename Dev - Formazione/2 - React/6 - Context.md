Una volta che abbiamo tutto il nostro server state cache in [React Query](5%20-%20Cache%20management%20-%20React%20Query.md), non rimane molto altro `state` rimasto in giro per l'applicazione che non possa essere gestito tramite composition e lifting state.

Comunque in alcuni casi abbiamo la necessità di avere certi UI state accessibili globalmente via context. Ad esempio se l'utente è loggato, o per le notifiche toast ecc.

>Vedi [[useContext]]

[useContext – React](https://react.dev/reference/react/useContext)