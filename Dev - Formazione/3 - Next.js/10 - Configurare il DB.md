Uno dei modi più semplici per creare un DB e deployare la propria app è utilizzare [[Vercel]].
Una volta deployata l'app è possibile creare uno store tramite interfaccia.
Una volta creato:
1. Andare sul tab `.env.local` e cliccare su **Show secret** e copiare lo snippet (assicurarsi di rivelarli prima di copiarli)
2. Incollare nel file `.env` del proprio applicativo
3. Assicurarsi che il file `.env` sia fra i file ignorati nel [[.gitignore]]
4. lanciare `pnpm i @vercel/postgres` per installare l'sdk di vercel [[Postgres]]