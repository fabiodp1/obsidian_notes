## WHAT is a Cloud computing service

E’ un servizio che mi permette di deployare il mio applicativo (o in generale risorsa IT) online, mettendomi a disposizione le risorse (numero di server, memoria ecc.) senza che io quindi debba avere una struttura hardware in casa, con un modello di pagamento pay-as-you-go (pago in base a quanto l’utilizzo). Si pagano solo le istanze EC2 attive.

## WHY should I need it

In un’applicazione on-premises se pago per avere un’infrastruttura che possa servire molte utenze, non è detto che sarà sfruttata al massimo, quindi rischio di pagare più di quello che necessito.

Con i servizi in cloud vengono create e distrutte nuove istanze del mio applicativo all’occorrenza, in questa maniera pago solo quello che è necessario.

## WHEN it is really needed

## HOW it works

## Module 1

### Concepts

**[[Amazon EC2]]**: Amazon Elastic Compute Cloud, corrisponde a quello che sarebbe per il cloud amazon il **Server** cioè il Server Virtuale.

## Module 2

### Concepts

**[[Hypervisor]]**: E’ un software che si occupa della creazione e gestione delle VM. Può essere di tipo 1 (bare metal) e viene installato direttamente sull’hardaware fisico, senza bisogno di un sistema operativo sottostante (tipico dei datacenter aziendali). Oppure può essere sotto forma di applicazione installabile su un sistema operativo (es. VirtualBox).

**Multitenancy**: la condivisione di risorse hardware fra macchine virtuali. L’hypervisor è il responsabile della sua gestione.

**CaaS**: Compute as a Service, il modello adottato da [[AWS]] e i servizi cloud in generale

**[[Elastic Load Balancing]] (ELB):** [[Load balancer]](applicazione) che si occupa di indirizzare le richieste sulle istanze meno indaffarate, in modo da bilanciare il carico.

**[[Tightly coupled architecture]]:** quando le applicazioni comunicano direttamente fra di loro. Il chiamante attende la risposta per proseguire con gli altri processi. Poco efficiente perché se uno dei componenti nella catena di messaggio fallisce o ha problemi, crea problemi a tutta la catena.

**[[Loosely coupled architecture]]:** molto più efficiente e sicura della tightly, poiché un single failure non causerà failure a cascata. Se un elemento della catena fallisce, viene isolato. In genere viene utilizzato un **buffer** che raccoglie i messaggi (message queue).

**[[Server-less]]**: un'infrastruttura server-less implica che non è possibile vedere o accedere all'infrastruttura sottostante il server, tutta la gestione, dalla scalabilità, disponibilità e gestione dell'applicativo, vengono autogestiti dal servizio.
### Famiglie di istanze

Ogni tipo di istanza di EC2 viene raggruppata in una famiglia di istanze

- **Generale purpose:** a questa famiglia appartengono dei tipi di ec2 bilanciati nelle risorse, multipurpose. Utili per:
    - **web server**
    - **code repository**.
- **Compute optimized:** Ideale per task con necessità di computazione elevate, come:
    - **gaming servers**
    - **HPC**
    - **modellazione scientifica**.
- **Memory optimized:** come per compute optimized ma focalizzata su task che richiedono memoria.
- **Accelerated computing:** utile per:
    - **Calcoli sui Floating-point number**
    - **Processamento grafico**
    - **Data pattern matching**
    - **Acceleratori hardware**
- **Storage optimized:** se sono necessarie elevate performance per dati locali

### Messaging and Queuing

Amazon AWS fornisce diversi tipi buffer (queue) per la gestione dei messaggi, **tipico dei microservices**:

- **SQS:** è una coda per operazioni asincrone, ma gli elementi della cosa vengono presi in carico da un solo elemento, una volta soddisfatti non sono accessibili ad altri, permette di:
    - mandare messaggi
    - salvare messaggi
    - ricevere messaggi
    - funziona fra componenti software
    - con qualsiasi volume di dati
    - il payload dei messaggi viene protetto
- **SNS (Simple Notification Service) topic:** permette di inviare messaggi a diversi tipi di client, un canale per inviare messaggi a tutti i subscribers di un determinato topic. I subscribers possono essere altri endpoint, http web hooks, AWS lambda function ecc. Può anche essere usato per notificare gli enduser attraverso notifiche push mobile, sms e email.

#### Quando usare Amazon [[SQS]] o Amazon [[SNS]]?

- **Usare SQS** quando:
    - Hai bisogno di una coda point-to-point e vuoi garantire che ogni messaggio venga elaborato una sola volta.
    - Vuoi elaborare messaggi in ordine e hai bisogno di un controllo rigoroso su chi riceve i messaggi (ad esempio, in una pipeline di elaborazione dati).
- **Usare SNS** quando:
    - Vuoi trasmettere lo stesso messaggio a più destinatari contemporaneamente.
    - Vuoi realizzare un pattern di notifica, dove uno o più servizi devono ricevere notifiche di un evento in tempo reale.
### Servizi Server-less
Aws fornisce diverse opzioni di computamento [[server-less]], una di queste è [[AWS Lambda]]:
Il codice viene caricato in una funzione lambda, viene configurato un trigger e quando questo viene attivato da una richiesta, il codice viene avviato all'interno di un ambiente messo a disposizione da AWS.

### Container service
Se abbiamo necessità di avviare il nostro applicativo in forma di container, AWS mette a disposizione Amazon Elastic Container Service ([[Amazon ECS]]) o Amazon Elastic Kubernets Service ([[Amazon EKS]]). Entrambe sono dei [[Container Orchestration Tools]].
Utilizzando Docker container sugli EC2 è come utilizzare normalmente i container docker avendo come host gli EC2. Per gestire l'avvio, chiusura o monitoraggio di un [[cluster]] di container, necessitiamo dei Container Orchestrator, come quelli messi a disposizione da AWS.

Sia gli ECS che EKS possono girare su EC2, ma se non abbiamo necessità di accedere o configurare un determinato sistema operativo, o non vogliamo gestire gli EC2, possiamo utilizzare [[AWS Fargate]], una server-less compute platform.

#### Più semplicemente:
Quando utilizzare [[EC2]]:
- Bisogno di ospitare applicazioni tradizionali
- Necessitiamo un accesso totale dell''OS
Quando utilizzare [[AWS Lambda]]:
- Quando vogliamo ospitare piccoli script e funzioni
- Abbiamo un'applicazione service-oriented
- Abbiamo un'applicazione Event driven
- Non necessitiamo di gestire l'ambiente sottostante
