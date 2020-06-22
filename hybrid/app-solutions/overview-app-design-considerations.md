---
title: Considerazioni sulla progettazione di app ibride in Azure e Azure Stack Hub
description: Informazioni sulle considerazioni di progettazione per la creazione di un'app ibrida per il cloud intelligente e i dispositivi perimetrali intelligenti, tra cui posizionamento, scalabilità, disponibilità e resilienza.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4fd52f76baad8059e130adfc01cdd0152b40a510
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911115"
---
# <a name="hybrid-app-design-considerations"></a>Considerazioni sulla progettazione di app ibride

Microsoft Azure è l'unico Cloud ibrido coerente. Consente di riutilizzare gli investimenti per lo sviluppo e di abilitare app che possono estendersi su Azure globale, sui cloud sovrani di Azure e Azure Stack, che è un'estensione di Azure nel Data Center. Le app che si estendono su cloud sono anche denominate *app ibride*.

Nella [*Guida all'architettura applicazione Azure*](https://docs.microsoft.com/azure/architecture/guide) viene descritto un approccio strutturato per la progettazione di applicazioni scalabili, resilienti e a disponibilità elevata. Le considerazioni descritte nella [*Guida all'architettura di applicazione Azure*](https://docs.microsoft.com/azure/architecture/guide) si applicano ugualmente alle app progettate per un singolo Cloud e per le app che si estendono su cloud.

In questo articolo vengono illustrati i [*pilastri della qualità del software*](https://docs.microsoft.com/azure/architecture/guide/pillars) illustrati nella [ *Guida all'architettura*](https://docs.microsoft.com/azure/architecture/guide/) di [*applicazione Azure*](https://docs.microsoft.com/azure/architecture/guide/) , con particolare attenzione alla progettazione di app ibride. Inoltre, aggiungiamo un pilastro di *posizionamento* come app ibride non esclusive per un cloud o un Data Center locale.

Gli scenari ibridi variano molto con le risorse disponibili per lo sviluppo e comprendono considerazioni quali geografia, sicurezza, accesso a Internet e altre considerazioni. Sebbene questa guida non possa enumerare le considerazioni specifiche, può fornire alcune linee guida e procedure consigliate da seguire. La progettazione, la configurazione, la distribuzione e la gestione di un'architettura di app ibrida implicano molte considerazioni sulla progettazione che potrebbero non essere note.

Questo documento mira ad aggregare le possibili domande che possono verificarsi quando si implementano app ibride e fornisce considerazioni (questi pilastri) e procedure consigliate per l'uso. Affrontando queste domande durante la fase di progettazione, si eviteranno i problemi che potrebbero causare nell'ambiente di produzione.

Essenzialmente, si tratta di domande che è necessario considerare prima di creare un'app ibrida. Per iniziare, è necessario eseguire le operazioni seguenti:

- Identificare e valutare i componenti dell'app.
- Valutare i componenti dell'app rispetto ai pilastri.

## <a name="evaluate-the-app-components"></a>Valutare i componenti dell'app

Ogni componente di un'app ha un ruolo specifico all'interno dell'app di dimensioni maggiori e deve essere esaminato con tutte le considerazioni di progettazione. I requisiti e le funzionalità di ogni componente devono essere mappati a queste considerazioni per determinare l'architettura dell'app.

Scomporre l'app nei relativi componenti studiando l'architettura dell'app e determinando la sua consistenza. I componenti possono includere anche altre app con cui l'app interagisce. Quando si identificano i componenti, valutare le operazioni ibride desiderate in base alle relative caratteristiche ponendo le domande seguenti:

- Qual è lo scopo del componente?
- Quali sono le interdipendenze tra i componenti?

Ad esempio, un'app può avere un front-end e un back-end definiti come due componenti. In uno scenario ibrido, il front-end si trova in un cloud e il back-end si trova nell'altra. L'app fornisce canali di comunicazione tra il front-end e l'utente e anche tra il front-end e il back-end.

Un componente dell'app è definito da molti formati e scenari. L'attività più importante è la loro identificazione e la loro posizione nel cloud o in locale.

I componenti comuni dell'app da includere nell'inventario sono elencati nella tabella 1.

### <a name="table-1-common-app-components"></a>Tabella 1. Componenti comuni delle app

| **Componente** | **Linee guida sulle app ibride** |
| ---- | ---- |
| Connessioni client | L'app (su qualsiasi dispositivo) può accedere agli utenti in diversi modi, da un punto di ingresso singolo, inclusi i modi seguenti:<br>: Modello client-server che richiede all'utente di installare un client per l'uso con l'app. App basata su server a cui si accede da un browser.<br>-Le connessioni client possono includere notifiche in caso di interruzione della connessione o di avvisi quando possono essere applicati addebiti per il roaming. |
| Authentication  | L'autenticazione può essere necessaria per un utente che si connette all'app o da un componente che si connette a un altro. |
| API  | È possibile fornire agli sviluppatori l'accesso a livello di codice all'app con set di API e librerie di classi e fornire un'interfaccia di connessione basata su standard Internet. È anche possibile usare le API per scomporre un'app in unità logiche operative in modo indipendente. |
| Servizi  | È possibile usare i servizi concisi per fornire le funzionalità per un'app. Un servizio può essere il motore in cui viene eseguita l'app. |
| Code | È possibile usare le code per organizzare lo stato dei cicli di vita e degli Stati dei componenti dell'app. Queste code possono fornire funzionalità di messaggistica, notifiche e buffering alle entità di sottoscrizione. |
| Archiviazione dei dati | Un'app può essere con o senza stato. Le app con stato necessitano di archiviazione dei dati che può essere soddisfatta da numerosi formati e volumi. |
| Memorizzazione dei dati nella cache  | Un componente di caching dei dati nella progettazione può risolvere in modo strategico i problemi di latenza e svolgere un ruolo nell'attivazione del cloud. |
| Inserimento di dati | I dati possono essere inviati a un'app in molti modi, a partire da valori inviati dall'utente in un Web Form per un flusso di dati continuamente elevato. |
| Elaborazione dati | Le attività di elaborazione dei dati, ad esempio report, analisi, esportazioni batch e trasformazione dei dati, possono essere elaborate nell'origine o scaricate in un componente separato utilizzando una copia dei dati. |

## <a name="assess-app-components-for-pillars"></a>Valutare i componenti dell'app per i pilastri

Per ogni componente, valutarne le caratteristiche per ogni colonna. Quando si valuta ogni componente con tutti i pilastri, le domande che potrebbero non essere state considerate possono diventare note che influiscono sulla progettazione dell'app ibrida. Agire su queste considerazioni potrebbe aggiungere valore per ottimizzare l'app. La tabella 2 fornisce una descrizione di ogni pilastro in relazione alle app ibride.

### <a name="table-2-pillars"></a>Tabella 2. Pilastri

| **Concetto fondamentale** | **Descrizione** |
| ----------- | --------------------------------------------------------- |
| Selezione host  | Posizionamento strategico dei componenti nelle app ibride. |
| Scalabilità  | La capacità di un sistema di gestire carichi maggiori. |
| Disponibilità  | La percentuale di tempo in cui un'app ibrida è funzionante e funzionante. |
| Resilienza | Possibilità per il ripristino di un'app ibrida. |
| Gestione | Processi operativi che mantengono un sistema in esecuzione in produzione. |
| Sicurezza | Protezione delle app ibride e dei dati dalle minacce. |

## <a name="placement"></a>Selezione host

Un'app ibrida ha una considerazione di posizionamento, ad esempio per il Data Center.

La selezione host è l'attività importante del posizionamento dei componenti in modo che possano servire al meglio un'app ibrida. Per definizione, le app ibride si estendono in posizioni come da locale al cloud e tra cloud diversi. È possibile inserire i componenti dell'app nei cloud in due modi:

- **App ibride verticali**  
    I componenti dell'app vengono distribuiti tra percorsi diversi. Ogni singolo componente può disporre di più istanze situate solo in un'unica posizione.

- **App ibride orizzontali**  
    I componenti dell'app vengono distribuiti tra percorsi diversi. Ogni singolo componente può disporre di più istanze che si estendono su più percorsi.

    Alcuni componenti possono essere consapevoli della loro posizione, mentre altri non hanno alcuna conoscenza della loro posizione e posizionamento. Questa virtuosità può essere realizzata con un livello di astrazione. Questo livello, con un Framework di app moderno come i microservizi, può definire il modo in cui l'app viene gestita dal posizionamento dei componenti dell'app che operano sui nodi tra cloud.

### <a name="placement-checklist"></a>Elenco di controllo selezione host

**Verificare le località obbligatorie.** Assicurarsi che l'app o i relativi componenti siano necessari per operare in o richiedere la certificazione per un cloud specifico. Questo può includere i requisiti di sovranità dell'azienda o dettati dalla legge. Inoltre, determinare se sono necessarie operazioni locali per una località o impostazioni locali specifiche.

**Verificare le dipendenze di connettività.** Le posizioni obbligatorie e altri fattori possono determinare le dipendenze di connettività tra i componenti. Quando si inseriscono i componenti, determinare la connettività e la sicurezza ottimali per la comunicazione tra di essi. Le opzioni includono [ *VPN*,](https://docs.microsoft.com/azure/vpn-gateway/) [ *ExpressRoute*](https://docs.microsoft.com/azure/expressroute/) e [ *connessioni ibride*.](https://docs.microsoft.com/azure/app-service/app-service-hybrid-connections)

**Valutazione delle funzionalità della piattaforma.** Per ogni componente dell'app, vedere se il provider di risorse richiesto per il componente dell'app è disponibile nel cloud e se la larghezza di banda può soddisfare i requisiti di velocità effettiva e latenza previsti.

**Pianificare la portabilità.** Usare i Framework di app moderni, ad esempio contenitori o microservizi, per pianificare le operazioni di trasferimento e per impedire le dipendenze del servizio.

**Determinare i requisiti di sovranità dei dati.** Le app ibride sono incentrate sull'isolamento dei dati, ad esempio in un Data Center locale. Esaminare il posizionamento delle risorse per ottimizzare il successo per soddisfare questo requisito.

**Pianificare la latenza.** Le operazioni tra cloud possono introdurre una distanza fisica tra i componenti dell'app. Verificare i requisiti per soddisfare qualsiasi latenza.

**Controllare i flussi di traffico.** Gestire i picchi di utilizzo e le comunicazioni appropriate e sicure per i dati di informazioni personali, quando si accede dal front-end in un cloud pubblico.

## <a name="scalability"></a>Scalabilità

La scalabilità è la capacità di un sistema di gestire un aumento del carico in un'app, che può variare nel tempo in quanto altri fattori e forze influiscono sulle dimensioni dei destinatari, oltre alle dimensioni e all'ambito dell'app.

Per informazioni di base su questo pilastro, vedere [*scalabilità*](https://docs.microsoft.com/azure/architecture/guide/pillars#scalability) nei cinque pilastri di eccellenza dell'architettura.

Un approccio di scalabilità orizzontale per le app ibride consente di aggiungere più istanze per soddisfare la domanda e quindi di disabilitarle durante i periodi più tranquilli.

Negli scenari ibridi, la scalabilità orizzontale dei singoli componenti richiede considerazioni aggiuntive quando i componenti vengono distribuiti tra cloud. Il ridimensionamento di una parte dell'app può richiedere il ridimensionamento di un altro. Se ad esempio il numero di connessioni client aumenta ma i servizi Web dell'app non vengono ridimensionati in modo appropriato, il carico sul database potrebbe saturare l'app.

Alcuni componenti dell'app possono essere scalati in orizzontale in modo lineare, mentre altri hanno dipendenze di ridimensionamento e possono essere limitati alla portata di scalabilità. Ad esempio, un tunnel VPN che fornisce la connettività ibrida per i percorsi dei componenti dell'app presenta un limite per la larghezza di banda e la latenza a cui può essere ridimensionato. Come vengono ridimensionati i componenti dell'app per assicurarsi che questi requisiti siano soddisfatti?

### <a name="scalability-checklist"></a>Elenco di controllo per la scalabilità

**Verificare le soglie di scalabilità.** Per gestire le varie dipendenze nell'app, determinare la scalabilità in cui i componenti dell'app in cloud diversi possono essere ridimensionati in modo indipendente l'uno dall'altro, rispettando al tempo stesso i requisiti per l'esecuzione dell'app. Le app ibride devono spesso ridimensionare aree specifiche nell'app per gestire una funzionalità quando interagisce e influiscono sul resto dell'app. Il superamento di un numero di istanze front-end può ad esempio richiedere la scalabilità del back-end.

**Definire pianificazioni di scalabilità.** Per la maggior parte delle app sono previsti periodi di occupato, quindi è necessario aggregare le ore di picco nelle pianificazioni per coordinare la scalabilità ottimale.

**Utilizzare un sistema di monitoraggio centralizzato.** Le funzionalità di monitoraggio della piattaforma possono fornire scalabilità automatica, ma le app ibride richiedono un sistema di monitoraggio centralizzato che aggrega l'integrità del sistema e il carico. Un sistema di monitoraggio centralizzato può avviare il ridimensionamento di una risorsa in una posizione e la scalabilità a seconda della risorsa in un'altra posizione. Inoltre, un sistema di monitoraggio centrale può tenere traccia dei cloud che scalano automaticamente le risorse e dei cloud.

**Sfruttare le funzionalità di scalabilità automatica (come disponibile).** Se le funzionalità di scalabilità automatica fanno parte dell'architettura, viene implementata la scalabilità automatica impostando soglie che definiscono quando un componente dell'app deve essere scalato in verticale, in uscita o in entrata. Un esempio di scalabilità automatica è una connessione client con scalabilità automatica in un cloud per gestire una maggiore capacità, ma comporta anche la scalabilità di altre dipendenze dell'app, distribuite tra cloud diversi. È necessario verificare le funzionalità di scalabilità automatica di questi componenti dipendenti.

Se la scalabilità automatica non è disponibile, prendere in considerazione l'implementazione di script e altre risorse per adattare il ridimensionamento manuale, attivato dalle soglie nel sistema di monitoraggio centralizzato.

**Determinare il carico previsto per località.** Le app ibride che gestiscono le richieste client possono basarsi principalmente su un'unica posizione. Quando il carico delle richieste client supera una soglia, è possibile aggiungere altre risorse in un percorso diverso per distribuire il carico delle richieste in ingresso. Verificare che le connessioni client siano in grado di gestire i carichi aumentati e determinare anche eventuali procedure automatizzate per le connessioni client per gestire il carico.

## <a name="availability"></a>Disponibilità

La disponibilità è il momento in cui un sistema è funzionante e funzionante. La disponibilità viene misurata come percentuale del tempo di esecuzione. Gli errori dell'app, i problemi dell'infrastruttura e il carico del sistema possono ridurre la disponibilità.

Per informazioni di base su questo pilastro, vedere [*disponibilità*](/azure/architecture/framework/) nei cinque pilastri di eccellenza dell'architettura.

### <a name="availability-checklist"></a>Elenco di controllo della disponibilità

**Fornire ridondanza per la connettività.** Le app ibride richiedono la connettività tra i cloud in cui è distribuita l'app. È possibile scegliere tra tecnologie per la connettività ibrida, quindi, oltre alla scelta della tecnologia principale, usare un'altra tecnologia per fornire ridondanza con le funzionalità di failover automatico in caso di errore della tecnologia principale.

**Classificazione domini di errore.** Per le app a tolleranza di errore sono necessari più domini di errore. I domini di errore consentono di isolare il punto di errore, ad esempio se un singolo disco rigido ha esito negativo in locale, se un interruttore Top-of-rack diventa inattivo o se il data center completo non è disponibile. In un'app ibrida una località può essere classificata come un dominio di errore. Con i requisiti di disponibilità maggiori, più è necessario valutare il modo in cui un singolo dominio di errore deve essere classificato.

**Classificazione domini di aggiornamento.** I domini di aggiornamento vengono usati per garantire che le istanze dei componenti dell'app siano disponibili, mentre altre istanze dello stesso componente vengono gestite con aggiornamenti o aggiornamenti delle funzionalità. Come per i domini di errore, i domini di aggiornamento possono essere classificati in base alla posizione tra le diverse posizioni. È necessario determinare se un componente dell'app è in grado di supportare l'aggiornamento in un'unica posizione prima di essere aggiornato in un altro percorso o se sono necessarie altre configurazioni di dominio. Una singola posizione può avere più domini di aggiornamento.

**Tenere traccia delle istanze e della disponibilità.** I componenti dell'app a disponibilità elevata possono essere disponibili tramite il bilanciamento del carico e la replica di dati sincrona. È necessario determinare il numero di istanze che è possibile offline prima che il servizio venga interrotto.

**Implementare la riparazione automatica.** Nel caso in cui un problema provochi un'interruzione della disponibilità dell'app, un rilevamento da parte di un sistema di monitoraggio potrebbe avviare attività di correzione automatica nell'app, ad esempio svuotare l'istanza non riuscita e ridistribuirla. Probabilmente è necessaria una soluzione di monitoraggio centrale, integrata con una pipeline di integrazione continua e distribuzione continua (CI/CD) ibrida. L'app è integrata con un sistema di monitoraggio per identificare i problemi che potrebbero richiedere la ridistribuzione di un componente dell'app. Il sistema di monitoraggio può anche attivare CI/CD ibridi per ridistribuire il componente dell'app e potenzialmente qualsiasi altro componente dipendente nella stessa posizione o in altre posizioni.

**Mantenere i contratti di servizio (SLA).** La disponibilità è essenziale per tutti i contratti per garantire la connettività ai servizi e alle app che si hanno con i clienti. Ogni località su cui si basa l'app ibrida potrebbe avere un contratto di contratto. Questi diversi contratti di contratto possono influenzare il contratto di contratto globale dell'app ibrida.

## <a name="resiliency"></a>Resilienza

La resilienza è la capacità di un sistema e di un'app ibrida di ripristinare gli errori e continuare a funzionare. L'obiettivo della resilienza consiste nel riportare l'app a uno stato completamente funzionante dopo che si è verificato un errore. Le strategie di resilienza includono soluzioni come backup, replica e ripristino di emergenza.

Per informazioni di base su questo pilastro, vedere [*resilienza*](https://docs.microsoft.com/azure/architecture/guide/pillars#resiliency) nei cinque pilastri dell'eccellenza dell'architettura.

### <a name="resiliency-checklist"></a>Elenco di controllo per la resilienza

**Individuare le dipendenze di ripristino di emergenza.** Il ripristino di emergenza in un cloud potrebbe richiedere modifiche ai componenti dell'app in un altro cloud. Se viene eseguito il failover di uno o più componenti di un cloud in un'altra posizione, all'interno dello stesso cloud o in un altro cloud, è necessario che i componenti dipendenti siano consapevoli di queste modifiche. Sono incluse anche le dipendenze di connettività. La resilienza richiede un piano di ripristino app completamente testato per ogni cloud.

**Stabilire un flusso di ripristino.** Una progettazione efficace del flusso di ripristino ha valutato i componenti dell'app per la loro capacità di gestire i buffer, i tentativi, ritentare il trasferimento dei dati non riusciti e, se necessario, eseguire il fallback a un servizio o a un flusso di lavoro diverso. È necessario determinare il meccanismo di backup da utilizzare, la relativa procedura di ripristino e la frequenza con cui viene testato. È anche necessario determinare la frequenza per i backup incrementali e completi.

**Testare i recuperi parziali.** Un ripristino parziale per una parte dell'app può fornire agli utenti la garanzia che tutti non sono disponibili. Questa parte del piano deve garantire che un ripristino parziale non abbia alcun effetto collaterale, ad esempio un servizio di backup e ripristino che interagisce con l'app per arrestarlo normalmente prima di eseguire il backup.

**Determinare gli istigatori del ripristino di emergenza e assegnare responsabilità.** Un piano di ripristino deve descrivere chi e quali ruoli possono avviare azioni di backup e ripristino oltre a ciò di cui è possibile eseguire il backup e il ripristino.

**Confrontare le soglie di correzione automatica con il ripristino di emergenza.** Determinare le funzionalità di riparazione automatica di un'app per l'avvio automatico del ripristino e il tempo necessario affinché la riparazione automatica di un'app venga considerata un errore o un esito positivo. Determinare le soglie per ogni cloud.

**Verificare la disponibilità delle funzionalità di resilienza.** Determinare la disponibilità delle funzionalità e delle funzionalità di resilienza per ogni località. Se una località non fornisce le funzionalità necessarie, è consigliabile integrare tale percorso in un servizio centralizzato che fornisca le funzionalità di resilienza.

**Determinare i tempi di inattività.** Determinare il tempo di inattività previsto per la manutenzione dell'app nel suo complesso e come componenti dell'app.

**Procedure per la risoluzione dei problemi del documento.** Definire le procedure di risoluzione dei problemi per ridistribuire le risorse e i componenti dell'app.

## <a name="manageability"></a>Gestione

Le considerazioni sulla gestione delle app ibride sono fondamentali per la progettazione dell'architettura. Un'app ibrida ben gestita fornisce un'infrastruttura come codice che consente l'integrazione di codice dell'app coerente in una pipeline di sviluppo comune. Implementando test coerenti a livello di sistema e singoli di modifiche apportate all'infrastruttura, è possibile garantire una distribuzione integrata se le modifiche superano i test, consentendo l'Unione nel codice sorgente.

Per informazioni di base su questo pilastro, vedere [*DevOps*](/azure/architecture/framework/#devops) nei cinque pilastri dell'eccellenza dell'architettura.

### <a name="manageability-checklist"></a>Elenco di controllo della gestibilità

**Implementare il monitoraggio.** Usare un sistema di monitoraggio centralizzato dei componenti dell'app distribuiti tra cloud per offrire una visualizzazione aggregata delle prestazioni e dell'integrità. Questo sistema include il monitoraggio dei componenti dell'app e delle funzionalità della piattaforma correlate.

Determinare le parti dell'app che richiedono il monitoraggio.

**Criteri di coordinazione.** Ogni posizione che un'app ibrida si estende può avere un proprio criterio che copre i tipi di risorse consentiti, le convenzioni di denominazione, i tag e altri criteri.

**Definire e usare i ruoli.** In qualità di amministratore di database, è necessario determinare le autorizzazioni necessarie per diverse persone, ad esempio un proprietario dell'app, un amministratore di database e un utente finale, che devono accedere alle risorse dell'app. Queste autorizzazioni devono essere configurate nelle risorse e all'interno dell'app. Un sistema di controllo degli accessi in base al ruolo consente di impostare queste autorizzazioni per le risorse dell'app. Questi diritti di accesso sono impegnativi quando tutte le risorse vengono distribuite in un singolo Cloud, ma richiedono ancora più attenzione quando le risorse vengono distribuite tra cloud. Le autorizzazioni per le risorse impostate in un cloud non si applicano alle risorse impostate in un altro cloud.

**Usare pipeline CI/CD.** Una pipeline di integrazione continua e sviluppo continuo (CI/CD) può offrire un processo coerente per la creazione e la distribuzione di app che si estendono su più cloud e per fornire la garanzia di qualità per l'infrastruttura e l'app. Questa pipeline consente di testare l'infrastruttura e l'app in un cloud e distribuirla in un altro cloud. La pipeline consente anche di distribuire alcuni componenti dell'app ibrida in un cloud e in altri componenti in un altro cloud, costituendo essenzialmente la base per la distribuzione di app ibride. Un sistema CI/CD è essenziale per la gestione delle dipendenze tra i componenti dell'app durante l'installazione, ad esempio l'app Web che necessita di una stringa di connessione al database.

**Gestire il ciclo di vita.** Poiché le risorse di un'app ibrida possono estendersi, è necessario aggregare tutte le funzionalità di gestione del ciclo di vita di ogni singola posizione in una singola unità di gestione del ciclo di vita. Prendere in considerazione il modo in cui vengono creati, aggiornati ed eliminati.

**Esaminare le strategie di risoluzione dei problemi.** La risoluzione dei problemi di un'app ibrida prevede più componenti dell'app rispetto alla stessa app in esecuzione in un unico Cloud. Oltre alla connettività tra i cloud, l'app viene eseguita su due piattaforme anziché una. Un'attività importante nella risoluzione dei problemi delle app ibride consiste nell'esaminare il monitoraggio dell'integrità e delle prestazioni aggregato dei componenti dell'app.

## <a name="security"></a>Sicurezza

La sicurezza è una delle considerazioni principali per qualsiasi app cloud e diventa ancora più critica per le app cloud ibride.

Per informazioni di base su questo pilastro, vedere [*sicurezza*](https://docs.microsoft.com/azure/architecture/guide/pillars#security) nei cinque pilastri dell'eccellenza dell'architettura.

### <a name="security-checklist"></a>Elenco di controllo relativo alla sicurezza

**Si presuppone una violazione.** Se una parte dell'app è compromessa, assicurarsi che esistano soluzioni per ridurre al minimo la diffusione della violazione, non solo all'interno della stessa posizione, ma anche in posizioni diverse.

**Monitoraggio dell'accesso alla rete consentito.** Determinare i criteri di accesso alla rete per l'app, ad esempio accedere solo all'app da una subnet specifica e consentire solo le porte e i protocolli minimi tra i componenti richiesti per il corretto funzionamento dell'app.

**Utilizzare un'autenticazione affidabile.** Uno schema di autenticazione affidabile è essenziale per la sicurezza dell'app. Si consiglia di usare un provider di identità federato che fornisce funzionalità di Single Sign-On e utilizza uno o più degli schemi seguenti: accesso tramite nome utente e password, chiavi pubbliche e private, autenticazione a due fattori o a più fattori e gruppi di sicurezza attendibili. Determinare le risorse appropriate per archiviare dati sensibili e altri segreti per l'autenticazione dell'app, oltre ai tipi di certificato e ai relativi requisiti.

**Utilizzare la crittografia.** Identificare le aree dell'app che usano la crittografia, ad esempio per l'archiviazione dei dati o la comunicazione client e l'accesso.

**Usare canali protetti.** Un canale sicuro tra i cloud è fondamentale per fornire controlli di sicurezza e autenticazione, protezione in tempo reale, quarantena e altri servizi tra cloud.

**Definire e usare i ruoli.** Implementare i ruoli per le configurazioni delle risorse e l'accesso a identità singola tra cloud. Determinare i requisiti di controllo degli accessi in base al ruolo (RBAC) per l'app e le relative risorse della piattaforma.

**Controllare il sistema.** Il monitoraggio del sistema consente di registrare e aggregare i dati sia dai componenti dell'app che dalle operazioni correlate della piattaforma cloud.

## <a name="summary"></a>Summary

Questo articolo fornisce un elenco di controllo di elementi importanti da considerare durante la creazione e la progettazione delle app ibride. Esaminando questi pilastri prima di distribuire l'app, è possibile evitare l'esecuzione di queste domande nelle interruzioni di produzione e potenzialmente richiedere di rivedere il progetto.

Può sembrare un'attività che richiede molto tempo in anticipo, ma è possibile ottenere facilmente il ritorno sugli investimenti se si progetta l'app in base a questi pilastri.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni, vedere le seguenti risorse:

- [Cloud ibrido](https://azure.microsoft.com/overview/hybrid-cloud/)
- [App cloud ibride](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [I modelli di Azure Resource Manager possono essere sviluppati per la coerenza cloud](https://aka.ms/consistency)
