---
title: Modello di rilevamento del piede con Azure e hub di Azure Stack
description: Informazioni su come usare Azure e hub di Azure Stack per implementare una soluzione di rilevamento del piede basata sull'intelligenza artificiale per l'analisi del traffico dei punti vendita al dettaglio.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281279"
---
# <a name="footfall-detection-pattern"></a>Modello di rilevamento del piede

Questo modello offre una panoramica per l'implementazione di una soluzione di rilevamento del piede basata sull'intelligenza artificiale per l'analisi del traffico dei visitatori nei negozi al dettaglio. La soluzione genera informazioni dettagliate dalle azioni reali, usando Azure, hub di Azure Stack e il kit di sviluppo per intelligenza artificiale Visione personalizzata.

## <a name="context-and-problem"></a>Contesto e problema

Contoso Stores vuole ottenere informazioni dettagliate sul modo in cui i clienti ricevono i prodotti correnti in relazione al layout del negozio. Non sono in grado di posizionare il personale in ogni sezione ed è inefficiente avere un team di analisti che rivede il filmato di un intero negozio. Inoltre, nessuno dei negozi ha larghezza di banda sufficiente per trasmettere video da tutte le fotocamere al cloud per l'analisi.

Contoso vuole trovare un modo discreto e rispettoso della privacy per determinare i dati demografici, la fedeltà e le reazioni dei clienti per archiviare display e prodotti.

## <a name="solution"></a>Soluzione

Questo modello di analisi delle vendite al dettaglio usa un approccio a livelli per l'inferenza sul perimetro. Usando l'Visione personalizzata AI Dev Kit, solo le immagini con visi umani vengono inviate per l'analisi a un hub di Azure Stack privato che esegue Servizi cognitivi di Azure. I dati aggregati anonimi vengono inviati ad Azure per l'aggregazione in tutti gli archivi e la visualizzazione in Power BI. La combinazione di edge e cloud pubblico consente a Contoso di sfruttare la moderna tecnologia di intelligenza artificiale pur mantenendo la conformità ai criteri aziendali e rispettando la privacy dei clienti.

[![Soluzione per il modello di rilevamento del piede](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Ecco un riepilogo del funzionamento della soluzione:

1. Il Visione personalizzata AI Dev Kit ottiene una configurazione dall'hub IoT, che installa IoT Edge Runtime e un ML modello.
2. Se il modello vede una persona, scatta una foto e la carica nell'hub di Azure Stack BLOB.
3. Il servizio BLOB attiva una funzione di Azure hub di Azure Stack.
4. La funzione di Azure chiama un contenitore con l'API Viso per ottenere dati demografici ed emozioni dall'immagine.
5. I dati vengono anonimi e inviati a un cluster Hub eventi di Azure dati.
6. Il cluster di Hub eventi inserisce i dati in Analisi di flusso.
7. Analisi di flusso aggrega i dati e lo inserisce Power BI.

## <a name="components"></a>Componenti

Questa soluzione usa i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Hardware in negozio | [Visione personalizzata AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Fornisce filtri nell'archivio usando un modello ML locale che acquisisce solo immagini di persone per l'analisi. Provisioning e aggiornamento sicuro tramite l'hub IoT.<br><br>|
| Azure | [Hub eventi di Azure](/azure/event-hubs/) | Hub eventi di Azure offre una piattaforma scalabile per l'inserimento di dati anonimi che si integra perfettamente con Analisi di flusso di Azure. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Un Analisi di flusso di Azure processo aggrega i dati anonimi e lo raggruppa in finestre di 15 secondi per la visualizzazione. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI un'interfaccia dashboard facile da usare per la visualizzazione dell'output da Analisi di flusso di Azure. |
| Hub di Azure Stack | [Servizio app](/azure-stack/operator/azure-stack-app-service-overview) | Il provider di risorse del servizio app (RP) fornisce una base per i componenti perimetrali, incluse le funzionalità di hosting e gestione per app/API Web e funzioni. |
| | servizio Azure Kubernetes [del motore del servizio Disassodato (AKS)](https://github.com/Azure/aks-engine) | Il componente del servizio AKS con AKS-Engine cluster distribuito in hub di Azure Stack offre un motore scalabile e resiliente per eseguire il contenitore API Viso. |
| | Servizi cognitivi di Azure [contenitori dell'API Viso](/azure/cognitive-services/face/face-how-to-install-containers)| La Servizi cognitivi di Azure RP con contenitori dell'API Viso fornisce dati demografici, emozioni e rilevamento univoco dei visitatori nella rete privata di Contoso. |
| | Archiviazione BLOB | Le immagini acquisite da AI Dev Kit vengono caricate nell'hub di Azure Stack BLOB di Azure. |
| | Funzioni di Azure | Una funzione di Azure in hub di Azure Stack riceve input dall'archivio BLOB e gestisce le interazioni con l'API Viso. Genera dati anonimi in un cluster di Hub eventi che si trova in Azure.<br><br>|

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, considerare i punti seguenti:

### <a name="scalability"></a>Scalabilità

Per consentire a questa soluzione di ridimensionare più fotocamere e posizioni, è necessario assicurarsi che tutti i componenti siano in grado di gestire il carico aumentato. Potrebbe essere necessario eseguire azioni come:

- Aumentare il numero di unità di streaming di Analisi di flusso.
- Aumentare le dimensioni della distribuzione dell'API Viso.
- Aumentare la velocità effettiva del cluster di Hub eventi.
- In casi estremi, può essere necessaria la Funzioni di Azure a una macchina virtuale.

### <a name="availability"></a>Disponibilità

Poiché questa soluzione è a livelli, è importante pensare a come gestire gli errori di rete o di alimentazione. A seconda delle esigenze aziendali, potrebbe essere necessario implementare un meccanismo per memorizzare nella cache le immagini in locale e quindi inoltrare l'hub di Azure Stack quando la connettività viene restituita. Se la posizione è sufficientemente grande, la distribuzione di un Data Box Edge con il contenitore API Viso in tale posizione potrebbe essere un'opzione migliore.

### <a name="manageability"></a>Gestione

Questa soluzione può estendersi a molti dispositivi e posizioni, che potrebbero non essere ingombrante. [I servizi IoT di Azure](/azure/iot-fundamentals/) possono essere usati per portare automaticamente online nuovi dispositivi e località e mantenerli aggiornati.

### <a name="security"></a>Sicurezza

Questa soluzione acquisisce le immagini dei clienti, rendendo la sicurezza una considerazione fondamentale. Assicurarsi che tutti gli account di archiviazione siano protetti con i criteri di accesso adeguati e ruotare regolarmente le chiavi. Assicurarsi che gli account di archiviazione e Hub eventi siano criteri di conservazione che soddisfino le normative sulla privacy aziendali e governative. Assicurarsi anche di livello i livelli di accesso utente. La distribuzione a livelli garantisce che gli utenti hanno accesso solo ai dati necessari per il proprio ruolo.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sugli argomenti introdotti in questo articolo:

- Vedere il [modello di dati a livelli](https://aka.ms/tiereddatadeploy), che viene sfruttato dal modello di rilevamento del piede.
- Per altre [informazioni sull'Visione personalizzata visione personalizzata,](https://azure.github.io/Vision-AI-DevKit-Pages/) vedere il Visione personalizzata AI Dev Kit. 

Quando si è pronti per testare l'esempio di soluzione, continuare con la Guida [alla distribuzione del rilevamento footfall](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection). Questa guida contiene istruzioni dettagliate per la distribuzione e il test dei componenti.