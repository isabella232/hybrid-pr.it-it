---
title: Considerazioni per la progettazione di app ibride in Azure e nell'hub di Azure Stack
description: Considerazioni sulla progettazione per la creazione di un'app ibrida per il cloud intelligente e la rete perimetrale intelligente, inclusi il posizionamento, la scalabilità, la disponibilità e la resilienza.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8b975c7b99807490d446f557e84b6e0eabf34649
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852491"
---
# <a name="hybrid-app-design-considerations"></a>Considerazioni per la progettazione di app ibride

Microsoft Azure è l'unico cloud ibrido coerente. Consente di riutilizzare gli investimenti per lo sviluppo e abilita app che possono estendersi su Azure globale, sui cloud sovrani di Azure e Azure Stack, che è un'estensione di Azure nel data center. Le app che si estendono su cloud sono anche chiamate *app ibride*.

La [*Guida all'architettura delle applicazioni in Azure*](/azure/architecture/guide) descrive un approccio strutturato alla progettazione di app scalabili, resilienti e a disponibilità elevata. Le considerazioni illustrate nella [*Guida all'architettura delle applicazioni in Azure*](/azure/architecture/guide) si applicano ugualmente alle app progettate per un singolo cloud e per le app che si estendono su cloud.

Questo articolo approfondisce i [*Concetti fondamentali della qualità del software*](/azure/architecture/guide/pillars) illustrati nella [*Guida all'architettura*](/azure/architecture/guide/) [*delle applicazioni in Azure*,](/azure/architecture/guide/) concentrandosi specificamente sulla progettazione di app ibride. Viene aggiunto anche un concetto fondamentale sul *posizionamento* poiché le app ibride non sono esclusive di un unico cloud o data center locale.

Gli scenari ibridi variano molto a seconda delle risorse disponibili per lo sviluppo e comprendono considerazioni su geografia, sicurezza, accesso a Internet e altre considerazioni. Sebbene questa guida non possa includere considerazioni specifiche, può offrire alcune linee guida e procedure consigliate da seguire. La progettazione, la configurazione, la distribuzione e la gestione dell'architettura di un'app ibrida implicano molte considerazioni per la progettazione che potrebbero non essere note.

Questo documento mira ad aggregare le possibili domande che possono sorgere quando si implementano app ibride e offre considerazioni (concetti fondamentali) e procedure consigliate per l'uso di app ibride. Rispondendo a queste domande durante la fase di progettazione, si eviteranno i problemi che potrebbero verificarsi nell'ambiente di produzione.

Essenzialmente, si tratta di domande che è necessario porsi prima di creare un'app ibrida. Prima di iniziare, è necessario eseguire questi passaggi:

- Identificare e valutare i componenti dell'app.
- Valutare i componenti dell'app rispetto ai concetti fondamentali.

## <a name="evaluate-the-app-components"></a>Valutare i componenti dell'app

Ogni componente di un'app ha un ruolo specifico all'interno dell'app e deve essere esaminato con tutte le considerazioni per la progettazione. I requisiti e le funzionalità di ogni componente devono tenere presente queste considerazioni per determinare l'architettura dell'app.

Scomporre l'app nei relativi componenti studiandone l'architettura e individuandone gli elementi costitutivi. I componenti possono includere anche altre app con cui l'app interagisce. Quando si identificano i componenti, valutare le operazioni ibride desiderate in base alle relative caratteristiche ponendo le domande seguenti:

- Qual è la funzione del componente?
- Quali sono le interdipendenze tra i componenti?

Ad esempio, un'app può avere un front-end e un back-end definiti come due componenti. In uno scenario ibrido, il front-end si trova in un cloud mentre il back-end si trova in un altro cloud. L'app offre canali di comunicazione tra il front-end e l'utente e anche tra il front-end e il back-end.

Un componente di un'app è definito da moduli e scenari diversi. L'attività più importante è la loro identificazione e la loro posizione nel cloud o in locale.

I componenti comuni dell'app da includere nell'inventario sono elencati nella tabella 1.

### <a name="table-1-common-app-components"></a>Tabella 1. Componenti comuni dell'app

| **Componente** | **Linee guida per le app ibride** |
| ---- | ---- |
| Connessioni client | L'app (su qualsiasi dispositivo) può accedere agli utenti, da un punto di ingresso singolo, in diversi modi:<br>-   Un modello client-server che richiede all'utente di installare un client per l'uso con l'app. Un'app basata su server a cui si accede da un browser.<br>- Le connessioni client possono includere notifiche in caso di interruzione della connessione o avvisi quando possono essere applicati addebiti per il roaming. |
| Authentication  | L'autenticazione può essere necessaria per un utente che si connette all'app o da un componente che si connette a un altro componente. |
| API  | È possibile concedere agli sviluppatori l'accesso a livello di codice all'app con set di API e librerie di classi e offrire un'interfaccia di connessione basata su standard Internet. È anche possibile usare le API per scomporre un'app in unità logiche operative in modo indipendente. |
| Servizi  | È possibile usare i servizi concisi per offrire le funzionalità per un'app. Un servizio può essere il motore in cui viene eseguita l'app. |
| Code | È possibile usare le code per organizzare lo stato dei cicli di vita e gli stati dei componenti dell'app. Queste code possono offrire funzionalità di messaggistica, notifiche e buffering alle entità di sottoscrizione. |
| Archiviazione dei dati | Un'app può essere senza stato o con stato. Le app con stato necessitano di archiviazione dei dati che può essere offerta da numerosi formati e volumi. |
| Memorizzazione dei dati nella cache  | Un componente di caching dei dati nella progettazione può risolvere in modo strategico i problemi di latenza e svolgere un ruolo nell'attivazione del bursting del cloud. |
| Inserimento di dati | I dati possono essere inviati a un'app in molti modi, dai valori inviati dall'utente in un modulo Web a un flusso di dati continuamente elevato. |
| Elaborazione dati | Le attività di elaborazione dei dati, ad esempio report, analisi, esportazioni batch e trasformazione dei dati, possono essere elaborate nell'origine o scaricate in un componente separato usando una copia dei dati. |

## <a name="assess-app-components-for-pillars"></a>Valutare i componenti dell'app rispetto ai concetti fondamentali

Per ogni componente, valutarne le caratteristiche in base a ogni concetto fondamentale. Quando si valuta ogni componente in base a tutti i concetti fondamentali, possono sorgere domande non considerate in precedenza che influiscono sulla progettazione dell'app ibrida. Agire in base a queste considerazioni potrebbe aggiungere valore per l'ottimizzazione dell'app. La tabella 2 include una descrizione di ogni concetto fondamentale in relazione alle app ibride.

### <a name="table-2-pillars"></a>Tabella 2. Concetti fondamentali

| **Concetto fondamentale** | **Descrizione** |
| ----------- | --------------------------------------------------------- |
| Posizione  | Posizionamento strategico dei componenti nelle app ibride. |
| Scalabilità  | La capacità di un sistema di gestire carichi maggiori. |
| Disponibilità  | La percentuale di tempo per cui un'app ibrida funziona ed è in esecuzione. |
| Resilienza | Possibilità di ripristino di un'app ibrida. |
| Gestione | Processi operativi che mantengono un sistema in esecuzione in produzione. |
| Security | Protezione delle app ibride e dei dati dalle minacce. |

## <a name="placement"></a>Posizione

Un'app ibrida implica una considerazione sul posizionamento, ad esempio per il data center.

Il posizionamento è l'attività importante di inserimento dei componenti in modo che possano servire al meglio un'app ibrida. Per definizione, le app ibride si estendono in più posizioni, ad esempio da locale al cloud e tra cloud diversi. È possibile posizionare i componenti dell'app nei cloud in due modi:

- **App ibride verticali**  
    I componenti dell'app vengono distribuiti in più posizioni. Ogni singolo componente può avere più istanze che si trovano in una sola posizione.

- **App ibride orizzontali**  
    I componenti dell'app vengono distribuiti in più posizioni. Ogni singolo componente può avere più istanze che si estendono su più posizioni.

    In alcuni componenti la posizione è nota, mentre in altri posizione e posizionamento non sono noti. Questa virtuosità può essere raggiunta con un livello di astrazione. Questo livello, con un framework di app moderno come i microservizi, può definire il modo in cui l'app viene gestita dal posizionamento dei componenti dell'app che operano sui nodi nei cloud.

### <a name="placement-checklist"></a>Elenco di controllo del posizionamento

**Verificare le posizioni obbligatorie.** Assicurarsi che l'app o i relativi componenti richiedano l'esecuzione in un cloud specifico o la certificazione per un cloud specifico. Questo può includere i requisiti di sovranità dell'azienda o previsti dalla legge. Determinare inoltre se sono necessarie operazioni locali per una posizione specifica o per le impostazioni locali.

**Verificare le dipendenze di connettività.** Le posizioni obbligatorie e altri fattori possono determinare le dipendenze di connettività tra i componenti. Quando si posizionano i componenti, determinare la connettività e la sicurezza ottimali per la comunicazione tra di essi. Le opzioni includono [*VPN*,](/azure/vpn-gateway/) [*ExpressRoute*](/azure/expressroute/) e [*Connessioni ibride*.](/azure/app-service/app-service-hybrid-connections)

**Valutare le funzionalità delle piattaforme.** Per ogni componente dell'app, verificare se è disponibile nel cloud il provider di risorse richiesto per il componente dell'app e se la larghezza di banda può soddisfare i requisiti di velocità effettiva e latenza previsti.

**Pianificare la portabilità.** Usare i framework di app moderni, ad esempio contenitori o microservizi, per pianificare le operazioni di trasferimento e per impedire le dipendenze del servizio.

**Determinare i requisiti di sovranità dei dati.** Le app ibride sono progettate per l'isolamento dei dati, ad esempio in un data center locale. Esaminare il posizionamento delle risorse per soddisfare correttamente questo requisito.

**Pianificare la latenza.** Le operazioni tra cloud possono introdurre una distanza fisica tra i componenti dell'app. Verificare i requisiti per soddisfare qualsiasi latenza.

**Controllare i flussi di traffico.** Gestire i picchi di utilizzo e le comunicazioni appropriate e sicure per i dati di informazioni personali quando l'accesso viene eseguito dal front-end in un cloud pubblico.

## <a name="scalability"></a>Scalabilità

La scalabilità è la capacità di un sistema di gestire un aumento del carico in un'app, che può variare nel tempo poiché altri fattori e forze influiscono sul numero di destinatari, oltre alle dimensioni e all'ambito dell'app.

Per informazioni di base su questo concetto fondamentale, vedere [*Scalabilità*](/azure/architecture/guide/pillars#scalability) nei cinque concetti fondamentali dell'eccellenza dell'architettura.

Un approccio di ridimensionamento orizzontale per le app ibride consente di aggiungere più istanze per soddisfare la domanda e quindi di disabilitarle nei periodi in cui il carico è inferiore.

Negli scenari ibridi l'aumento delle istanze dei singoli componenti richiede considerazioni aggiuntive quando i componenti vengono distribuiti nei cloud. Il ridimensionamento di una parte dell'app può richiedere il ridimensionamento di un'altra parte. Ad esempio, se il numero di connessioni client aumenta ma i servizi Web dell'app non vengono aumentati in modo appropriato, il carico sul database potrebbe saturare l'app.

Alcuni componenti dell'app possono essere aumentati in modo lineare, mentre altri hanno dipendenze di ridimensionamento e possono essere limitati nella scalabilità. Ad esempio, un tunnel VPN che offre la connettività ibrida per le posizioni dei componenti dell'app presenta un limite per la larghezza di banda e la latenza a cui può essere ridimensionato. Come vengono ridimensionati i componenti dell'app per assicurarsi che questi requisiti siano soddisfatti?

### <a name="scalability-checklist"></a>Elenco di controllo per la scalabilità

**Verificare le soglie di ridimensionamento.** Per gestire le varie dipendenze nell'app, determinare la misura in cui i componenti dell'app in cloud diversi possono essere ridimensionati in modo indipendente l'uno dall'altro, rispettando al tempo stesso i requisiti per l'esecuzione dell'app. Le app ibride devono spesso ridimensionare aree specifiche nell'app per gestire una funzionalità quando interagisce e influisce sulla parte rimanente dell'app. Ad esempio. il superamento di un numero di istanze front-end può richiedere il ridimensionamento del back-end.

**Definire le pianificazioni di ridimensionamento.** Poiché la maggior parte delle app ha periodi di maggiore attività, è necessario aggregare le ore di picco nelle pianificazioni per coordinare un ridimensionamento ottimale.

**Usare un sistema di monitoraggio centralizzato.** Sebbene le funzionalità di monitoraggio della piattaforma possano offrire scalabilità automatica, le app ibride richiedono un sistema di monitoraggio centralizzato che aggrega l'integrità del sistema e il carico. Un sistema di monitoraggio centralizzato può avviare il ridimensionamento di una risorsa in una posizione e il ridimensionamento a seconda della risorsa in un'altra posizione. Inoltre, un sistema di monitoraggio centrale può tenere traccia dei cloud che scalano automaticamente le risorse e dei cloud che non eseguono questa operazione.

**Usare le funzionalità di scalabilità automatica disponibili.** Se le funzionalità di scalabilità automatica fanno parte dell'architettura, la scalabilità automatica viene implementata impostando soglie che definiscono quando è necessario aumentare o ridurre le istanze di un componente dell'app. Un esempio di scalabilità automatica è una connessione client che viene scalata automaticamente in un cloud per gestire una maggiore capacità, ma comporta anche la scalabilità di altre dipendenze dell'app, distribuite in cloud diversi. È necessario verificare le funzionalità di scalabilità automatica di questi componenti dipendenti.

Se la scalabilità automatica non è disponibile, prendere in considerazione l'implementazione di script e altre risorse per il ridimensionamento manuale, attivati dalle soglie nel sistema di monitoraggio centralizzato.

**Determinare il carico previsto in base alla posizione.** Le app ibride che gestiscono le richieste client possono basarsi principalmente su un'unica posizione. Quando il carico delle richieste client supera una soglia, è possibile aggiungere altre risorse in una posizione diversa per distribuire il carico delle richieste in ingresso. Verificare che le connessioni client siano in grado di gestire i carichi aumentati e determinare anche eventuali procedure automatizzate per le connessioni client per gestire il carico.

## <a name="availability"></a>Disponibilità

La disponibilità è il tempo in cui il sistema funziona ed è in esecuzione. La disponibilità viene misurata come percentuale del tempo di attività. Gli errori dell'app, i problemi dell'infrastruttura e il carico del sistema possono ridurre la disponibilità.

Per informazioni di base su questo concetto fondamentale, vedere [*Disponibilità*](/azure/architecture/framework/) nei cinque concetti fondamentali dell'eccellenza dell'architettura.

### <a name="availability-checklist"></a>Elenco di controllo della disponibilità

**Offrire ridondanza per la connettività.** Le app ibride richiedono la connettività tra i cloud in cui è distribuita l'app. Poiché sono disponibili più tecnologie per la connettività ibrida, oltre alla scelta della tecnologia principale, usare un'altra tecnologia per offrire ridondanza con le funzionalità di failover automatico in caso di errore della tecnologia principale.

**Classificare i domini di errore.** Per le app a tolleranza di errore sono necessari più domini di errore. I domini di errore consentono di isolare il punto di errore, ad esempio se si verifica un errore in un singolo disco rigido in locale, se un commutatore Top-of-Rack diventa inattivo o se il data center completo non è disponibile. In un'app ibrida una posizione può essere classificata come dominio di errore. Con requisiti di disponibilità maggiori, maggiore sarà la necessità di valutare il modo in cui un singolo dominio di errore deve essere classificato.

**Classificare i domini di aggiornamento.** I domini di aggiornamento vengono usati per garantire che le istanze dei componenti dell'app siano disponibili, mentre altre istanze dello stesso componente vengono gestite con aggiornamenti o aggiornamenti delle funzionalità. Come per i domini di errore, i domini di aggiornamento possono essere classificati in base al posizionamento nelle diverse posizioni. È necessario determinare se un componente dell'app è in grado di supportare l'aggiornamento in una posizione prima di essere aggiornato in un'altra posizione o se sono necessarie altre configurazioni di dominio. Una singola posizione può avere più domini di aggiornamento.

**Tenere traccia delle istanze e della disponibilità.** I componenti dell'app a disponibilità elevata possono essere disponibili tramite il bilanciamento del carico e la replica di dati sincrona. È necessario determinare il numero di istanze che possono essere offline prima che il servizio venga interrotto.

**Implementare la riparazione automatica.** Nel caso in cui un problema provochi un'interruzione della disponibilità dell'app, un rilevamento da parte di un sistema di monitoraggio potrebbe avviare attività di riparazione automatica nell'app, ad esempio eliminando l'istanza con errore ed eseguendone di nuovo la distribuzione. Probabilmente è necessaria una soluzione di monitoraggio centrale, integrata con una pipeline con integrazione continua e recapito continuo (CI/CD) ibrida. L'app è integrata con un sistema di monitoraggio per identificare i problemi che potrebbero richiedere la ridistribuzione di un componente dell'app. Il sistema di monitoraggio può anche attivare CI/CD ibride per ridistribuire il componente dell'app e potenzialmente qualsiasi altro componente dipendente nella stessa posizione o in altre posizioni.

**Gestire i contratti di servizio.** La disponibilità è essenziale per tutti i contratti per garantire la connettività ai servizi e alle app offerte ai clienti. Ogni posizione su cui si basa l'app ibrida potrebbe avere un contratto di servizio specifico. Questi diversi contratti di servizio possono avere effetto sul contratto di servizio globale dell'app ibrida.

## <a name="resiliency"></a>Resilienza

La resilienza è la capacità di un'app ibrida e di un sistema di correggere gli errori e continuare a funzionare. L'obiettivo della resilienza consiste nel ripristinare uno stato completamente funzionale dell'app dopo un errore. Le strategie di resilienza includono soluzioni come backup, replica e ripristino di emergenza.

Per informazioni di base su questo concetto fondamentale, vedere [*Resilienza*](/azure/architecture/guide/pillars#resiliency) nei cinque concetti fondamentali dell'eccellenza dell'architettura.

### <a name="resiliency-checklist"></a>Elenco di controllo per la resilienza

**Individuare le dipendenze di ripristino di emergenza.** Il ripristino di emergenza in un cloud potrebbe richiedere modifiche ai componenti dell'app in un altro cloud. Se viene eseguito il failover di uno o più componenti di un cloud in un'altra posizione, all'interno dello stesso cloud o in un altro cloud, è necessario che le modifiche vengano comunicate ai componenti dipendenti. Sono incluse anche le dipendenze di connettività. La resilienza richiede un piano di ripristino app completamente testato per ogni cloud.

**Stabilire un flusso di ripristino.** Una progettazione efficace del flusso di ripristino ha valutato i componenti dell'app per la loro capacità di gestire i buffer, i tentativi, rieseguire il trasferimento di dati non riuscito e, se necessario, eseguire il fallback in un servizio o in un flusso di lavoro diverso. È necessario determinare il meccanismo di backup da usare, la relativa procedura di ripristino e la frequenza con cui viene testato. È anche necessario determinare la frequenza dei backup incrementali e completi.

**Testare i recuperi parziali.** Un ripristino parziale di una parte dell'app può offrire agli utenti la garanzia che tutti i dati non siano disponibili. Questa parte del piano deve garantire che un ripristino parziale non abbia alcun effetto collaterale, ad esempio un servizio di backup e ripristino che interagisce con l'app per arrestarla normalmente prima di eseguire il backup.

**Determinare gli istigatori del ripristino di emergenza e assegnare la responsabilità.** Un piano di ripristino deve descrivere gli utenti e i ruoli che possono avviare azioni di backup e ripristino e gli elementi di cui è possibile eseguire il backup e il ripristino.

**Confrontare le soglie di riparazione automatica con il ripristino di emergenza.** Determinare le funzionalità di riparazione automatica di un'app per l'avvio del ripristino automatico e il tempo necessario affinché la riparazione automatica di un'app venga considerata completata o non riuscita. Determinare le soglie per ogni cloud.

**Verificare la disponibilità delle funzionalità di resilienza.** Determinare la disponibilità delle funzionalità di resilienza per ogni posizione. Se una posizione non offre le funzionalità necessarie, è consigliabile integrare la posizione in un servizio centralizzato che offre le funzionalità di resilienza.

**Determinare i tempi di inattività.** Determinare il tempo di inattività previsto per la manutenzione dell'app completa e dei componenti dell'app.

**Documentare le procedure per la risoluzione dei problemi.** Definire le procedure di risoluzione dei problemi per ridistribuire le risorse e i componenti dell'app.

## <a name="manageability"></a>Gestione

Le considerazioni sulla gestione delle app ibride sono fondamentali per la progettazione dell'architettura. Un'app ibrida ben gestita offre un'infrastruttura come codice che abilita l'integrazione di codice dell'app coerente in una pipeline di sviluppo comune. Implementando test a livello di sistema e singoli coerenti delle modifiche apportate all'infrastruttura, è possibile garantire una distribuzione integrata se le modifiche superano i test, consentendone l'unione nel codice sorgente.

Per informazioni di base su questo concetto fondamentale, vedere [*DevOps*](/azure/architecture/framework/#devops) nei cinque concetti fondamentali dell'eccellenza dell'architettura.

### <a name="manageability-checklist"></a>Elenco di controllo della gestibilità

**Implementare il monitoraggio.** Usare un sistema di monitoraggio centralizzato dei componenti dell'app distribuiti nei cloud per offrire una vista aggregata dell'integrità e delle prestazioni. Questo sistema include il monitoraggio dei componenti dell'app e delle funzionalità della piattaforma correlate.

Determinare le parti dell'app che richiedono il monitoraggio.

**Coordinare i criteri.** Ogni posizione in cui si trova un'app ibrida può avere criteri specifici relativi ai tipi di risorse consentiti, alle convenzioni di denominazione, ai tag e altri criteri.

**Definire e usare i ruoli.** L'amministratore del database dovrà determinare le autorizzazioni necessarie per le diverse persone che necessitano dell'accesso alle risorse dell'app, ad esempio il proprietario di un'app, l'amministratore di un database e un utente finale. Queste autorizzazioni devono essere configurate nelle risorse e all'interno dell'app. Un sistema di controllo degli accessi in base al ruolo (RBAC) consente di impostare queste autorizzazioni per le risorse dell'app. Questi diritti di accesso sono complessi quando tutte le risorse vengono distribuite in un singolo cloud, ma richiedono ancora più attenzione quando le risorse vengono distribuite in più cloud. Le autorizzazioni per le risorse impostate in un cloud non si applicano alle risorse impostate in un altro cloud.

**Usare pipeline CI/CD.** Una pipeline con integrazione continua e sviluppo continuo (CI/CD) può offrire un processo coerente per la creazione e la distribuzione di app in più cloud e per offrire la garanzia di qualità per l'infrastruttura e l'app. Questa pipeline consente di testare l'infrastruttura e l'app in un cloud e di distribuirle in un altro cloud. La pipeline consente anche di distribuire alcuni componenti dell'app ibrida in un cloud e altri componenti in un altro cloud, costituendo essenzialmente la base per la distribuzione di app ibride. Un sistema CI/CD è essenziale per la gestione delle dipendenze tra i componenti dell'app durante l'installazione, ad esempio dell'app Web che necessita di una stringa di connessione al database.

**Gestire il ciclo di vita.** Poiché le risorse di un'app ibrida possono trovarsi in più posizioni, è necessario aggregare le funzionalità di gestione del ciclo di vita di ogni singola posizione in una singola unità di gestione del ciclo di vita. Prendere in considerazione il modo in cui vengono create, aggiornate ed eliminate.

**Esaminare le strategie di risoluzione dei problemi.** La risoluzione dei problemi di un'app ibrida interessa più componenti dell'app rispetto alla stessa app in esecuzione in un cloud singolo. Oltre alla connettività tra i cloud, l'app viene eseguita su due piattaforme anziché una. Un'attività importante nella risoluzione dei problemi delle app ibride consiste nell'esaminare il monitoraggio dell'integrità e delle prestazioni aggregato dei componenti dell'app.

## <a name="security"></a>Security

La sicurezza è una delle considerazioni principali per qualsiasi app cloud e diventa ancora più critica per le app cloud ibrido.

Per informazioni di base su questo concetto fondamentale, vedere [*Sicurezza*](/azure/architecture/guide/pillars#security) nei cinque concetti fondamentali dell'eccellenza dell'architettura.

### <a name="security-checklist"></a>Elenco di controllo relativo alla sicurezza

**Presupporre una violazione.** Se una parte dell'app è compromessa, assicurarsi che esistano soluzioni per ridurre al minimo la diffusione della violazione, non solo all'interno della stessa posizione, ma anche in posizioni diverse.

**Monitorare l'accesso alla rete consentito.** Determinare i criteri di accesso alla rete per l'app, ad esempio accedere solo all'app da una subnet specifica e consentire solo le porte e i protocolli minimi tra i componenti richiesti per il corretto funzionamento dell'app.

**Utilizzare un'autenticazione affidabile.** Uno schema di autenticazione affidabile è essenziale per la sicurezza dell'app. Si consiglia di usare un provider di identità federato che offre funzionalità di Single Sign-On e usa uno o più degli schemi seguenti: accesso tramite nome utente e password, chiavi pubbliche e private, autenticazione a due fattori o a più fattori e gruppi di sicurezza attendibili. Determinare le risorse appropriate per archiviare dati sensibili e altri segreti per l'autenticazione dell'app, oltre ai tipi di certificato e ai relativi requisiti.

**Usare la crittografia.** Identificare le aree dell'app che usano la crittografia, ad esempio per l'archiviazione dei dati o la comunicazione client e l'accesso.

**Usare canali protetti.** Un canale protetto nei cloud è fondamentale per offrire controlli di sicurezza e autenticazione, protezione in tempo reale, quarantena e altri servizi nei cloud.

**Definire e usare i ruoli.** Implementare i ruoli per le configurazioni delle risorse e l'accesso a identità singola nei cloud. Determinare i requisiti di controllo degli accessi in base al ruolo (RBAC) per l'app e le risorse della piattaforma.

**Controllare il sistema.** Il monitoraggio del sistema consente di registrare e aggregare i dati sia dai componenti dell'app che dalle operazioni correlate della piattaforma cloud.

## <a name="summary"></a>Summary

Questo articolo offre un elenco di controllo di elementi importanti da considerare durante la creazione e la progettazione delle app ibride. Esaminando questi concetti fondamentali prima di distribuire l'app, è possibile evitare di porsi queste domande nelle interruzioni di produzione e potenzialmente richiedere una revisione del progetto.

Può sembrare un'attività preliminare che richiede molto tempo, ma risulta sicuramente utile se si progetta l'app tenendo presente questi concetti fondamentali.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni, vedere le seguenti risorse:

- [Cloud ibrido](https://azure.microsoft.com/overview/hybrid-cloud/)
- [App cloud ibrido](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Sviluppare i modelli di Azure Resource Manager per la coerenza cloud](/azure/azure-resource-manager/templates/templates-cloud-consistency)