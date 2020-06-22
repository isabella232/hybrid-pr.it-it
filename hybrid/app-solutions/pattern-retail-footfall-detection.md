---
title: Modello di rilevamento calpestio con Azure e hub Azure Stack
description: Informazioni su come usare Azure e hub Azure Stack per implementare una soluzione di rilevamento calpestio basata su intelligenza artificiale per l'analisi del traffico del negozio al dettaglio.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911227"
---
# <a name="footfall-detection-pattern"></a>Modello di rilevamento calpestio

Questo modello offre una panoramica per l'implementazione di una soluzione di rilevamento calpestio basata su intelligenza artificiale per l'analisi del traffico dei visitatori nei negozi al dettaglio. La soluzione genera informazioni dettagliate da azioni reali, usando Azure, Hub Azure Stack e il Visione personalizzata AI Dev Kit.

## <a name="context-and-problem"></a>Contesto e problema

Gli archivi di Contoso desiderano ottenere informazioni dettagliate sul modo in cui i clienti ricevono i prodotti correnti in relazione al layout dell'archivio. Non sono in grado di collocare il personale in ogni sezione ed è inefficiente che un team di analisti esamini il metraggio della fotocamera di un intero negozio. Inoltre, nessun negozio ha una larghezza di banda sufficiente per trasmettere video da tutte le relative fotocamere al cloud per l'analisi.

Contoso desidera trovare un metodo non intrusivo e descrittivo per la privacy per determinare i dati demografici dei clienti, la lealtà e le reazioni per archiviare le visualizzazioni e i prodotti.

## <a name="solution"></a>Soluzione

Questo modello di analisi delle vendite al dettaglio usa un approccio a più livelli per l'inferenza al perimetro. Con Visione personalizzata AI Dev Kit vengono inviate solo le immagini con visi umani per l'analisi a un hub Azure Stack privato che esegue Servizi cognitivi di Azure. Resi anonimi, i dati aggregati vengono inviati ad Azure per l'aggregazione in tutti i punti vendita e la visualizzazione in Power BI. La combinazione di Edge e cloud pubblico permette a Contoso di sfruttare la tecnologia di intelligenza artificiale moderna, pur rimanendo in conformità con i criteri aziendali e rispettando la privacy dei clienti.

[![Soluzione modello di rilevamento calpestio](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Ecco un riepilogo del funzionamento della soluzione:

1. Il Visione personalizzata AI Dev Kit ottiene una configurazione dall'hub Internet che installa il runtime IoT Edge e un modello ML.
2. Se il modello vede una persona, acquisisce un'immagine e la carica nell'archivio BLOB dell'hub Azure Stack.
3. Il servizio BLOB attiva una funzione di Azure nell'hub Azure Stack.
4. La funzione di Azure chiama un contenitore con la API Viso per ottenere i dati demografici e di emozioni dall'immagine.
5. I dati vengono resi anonimi e inviati a un cluster di hub eventi di Azure.
6. Il cluster di hub eventi esegue il push dei dati in analisi di flusso.
7. Analisi di flusso aggrega i dati e li inserisce in Power BI.

## <a name="components"></a>Componenti

Questa soluzione USA i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Hardware nell'archivio | [Visione personalizzata AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Consente di filtrare in archivio usando un modello di ML locale che acquisisce solo le immagini di persone per l'analisi. Con provisioning e aggiornamento in modo sicuro tramite l'hub Internet.<br><br>|
| Azure | [Hub eventi di Azure](/azure/event-hubs/) | Hub eventi di Azure offre una piattaforma scalabile per l'inserimento di dati resi anonimi che si integrano in modo accurato con analisi di flusso di Azure. |
|  | [Analisi di flusso di Azure](/azure/stream-analytics/) | Un processo di analisi di flusso di Azure aggrega i dati di resi anonimi e li raggruppa in finestre di 15 secondi per la visualizzazione. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI offre un'interfaccia Dashboard di facile utilizzo per la visualizzazione dell'output di analisi di flusso di Azure. |
| Hub Azure Stack | [Servizio app](/azure-stack/operator/azure-stack-app-service-overview.md) | Il provider di risorse del servizio app fornisce una base per i componenti perimetrali, incluse le funzionalità di hosting e gestione per app Web/API e funzioni. |
| | Cluster del motore di Azure Kubernetes Service [(AKS)](https://github.com/Azure/aks-engine) | Il componente AKS RP con cluster del motore AKS distribuito in Azure Stack Hub fornisce un motore resiliente e scalabile per l'esecuzione del contenitore di API Viso. |
| | [Contenitori di API viso](/azure/cognitive-services/face/face-how-to-install-containers) di servizi cognitivi di Azure| I servizi cognitivi di Azure RP con API Viso contenitori forniscono rilevamento demografico, emotivo e univoco dei visitatori sulla rete privata di contoso. |
| | Archiviazione BLOB | Le immagini acquisite da AI Dev Kit vengono caricate nell'archiviazione BLOB di Azure Stack Hub. |
| | Funzioni di Azure | Una funzione di Azure in esecuzione nell'hub Azure Stack riceve l'input dall'archiviazione BLOB e gestisce le interazioni con l'API Viso. Emette i dati di resi anonimi a un cluster di hub eventi che si trova in Azure.<br><br>|

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

Per consentire la scalabilità di questa soluzione tra più fotocamere e posizioni, è necessario assicurarsi che tutti i componenti siano in grado di gestire l'aumento del carico. Potrebbe essere necessario eseguire azioni come:

- Aumentare il numero di unità di streaming di analisi di flusso.
- Scalare in orizzontale la distribuzione di API Viso.
- Aumentare la velocità effettiva del cluster di hub eventi.
- Per i casi estremi, potrebbe essere necessario eseguire la migrazione da funzioni di Azure a una macchina virtuale.

### <a name="availability"></a>Disponibilità

Poiché questa soluzione è a livelli, è importante considerare la modalità di gestione degli errori di rete o di alimentazione. A seconda delle esigenze aziendali, potrebbe essere necessario implementare un meccanismo per memorizzare nella cache le immagini localmente, quindi procedere all'hub Azure Stack al ritorno della connettività. Se il percorso è sufficientemente grande, la distribuzione di un Data Box Edge con il contenitore di API Viso in tale posizione potrebbe essere un'opzione migliore.

### <a name="manageability"></a>Gestione

Questa soluzione può estendersi a molti dispositivi e posizioni, che potrebbero essere difficili da gestire. I servizi Internet delle cose di [Azure](/azure/iot-fundamentals/) possono essere usati per riunire automaticamente nuove posizioni e dispositivi e mantenerli aggiornati.

### <a name="security"></a>Sicurezza

Questa soluzione acquisisce le immagini dei clienti, prendendo in considerazione la sicurezza. Verificare che tutti gli account di archiviazione siano protetti con i criteri di accesso appropriati e ruotare periodicamente le chiavi. Verificare che gli account di archiviazione e gli hub eventi dispongano di criteri di conservazione che soddisfino le normative sulla privacy aziendale e Assicurarsi anche di livelli di accesso utente. La suddivisione in livelli garantisce che gli utenti abbiano accesso solo ai dati necessari per il proprio ruolo.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Vedere il [modello di dati a livelli](https://aka.ms/tiereddatadeploy), che viene utilizzato dal modello di rilevamento calpestio.
- Per altre informazioni sull'uso della visione personalizzata, vedere [visione personalizzata ai Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) . 

Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione del rilevamento calpestio](solution-deployment-guide-retail-footfall-detection.md). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.
