---
title: Rilevamento delle scorte con Azure e Azure Stack Edge
description: Informazioni su come usare Azure e Azure Stack Edge per implementare il rilevamento di scorte non disponibili.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343876"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Rilevamento delle scorte nel modello perimetrale

Questo modello illustra come determinare se gli scaffali hanno articoli non disponibili usando Azure Stack Edge o Azure IoT Edge di rete e dispositivi.

## <a name="context-and-problem"></a>Contesto e problema

I negozi fisici perdono le vendite perché quando i clienti cercano un articolo, non è presente sullo scaffale. Tuttavia, l'elemento avrebbe potuto essere nella parte posteriore dell'archivio e non essere stato ricosto. I negozi desiderano usare il personale in modo più efficiente e ricevere automaticamente una notifica quando gli elementi devono essere riposiificati.

## <a name="solution"></a>Soluzione

L'esempio di soluzione usa un dispositivo perimetrale, ad esempio un Azure Stack Edge in ogni negozio, che elabora in modo efficiente i dati dalle fotocamere presenti nello store. Questa progettazione ottimizzata consente agli archivi di inviare al cloud solo eventi e immagini pertinenti. La progettazione consente di risparmiare larghezza di banda, spazio di archiviazione e garantisce la privacy dei clienti. Quando i fotogrammi vengono letti da ogni fotocamera, un modello di Machine Learning elabora l'immagine e restituisce eventuali aree non disponibili. L'immagine e le aree non disponibili vengono visualizzate in un'app Web locale. Questi dati possono essere inviati a un ambiente Time Series Insight per visualizzare informazioni dettagliate Power BI.

![Architettura della soluzione out-of-stock at edge](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Ecco come funziona la soluzione:

1. Le immagini vengono acquisite da una fotocamera di rete tramite HTTP o RTSP.
2. L'immagine viene ridimensionata e inviata al driver di inferenza, che comunica con il modello di Machine Learning per determinare se sono presenti immagini non disponibili.
3. Il modello ml restituisce tutte le aree non disponibili.
4. Il driver di inferenza carica l'immagine non elaborata in un BLOB (se specificato) e invia i risultati dal modello a hub IoT di Azure e un processore del rettangolo di selezione nel dispositivo.
5. Il processore del rettangolo di selezione aggiunge i recinti di selezione all'immagine e memorizza nella cache il percorso dell'immagine in un database in memoria.
6. L'app Web esegue una query per le immagini e le mostra nell'ordine di ricezione.
7. I messaggi dell'hub IoT vengono aggregati in Time Series Insights.
8. Power BI visualizza un report interattivo degli articoli non disponibili nel tempo con i dati Time Series Insights.


## <a name="components"></a>Componenti

Questa soluzione usa i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Hardware locale | Fotocamera di rete | È necessaria una fotocamera di rete, con un feed HTTP o RTSP per fornire le immagini per l'inferenza. |
| Azure | Hub IoT di Azure | [hub IoT di Azure](/azure/iot-hub/) gestisce il provisioning dei dispositivi e la messaggistica per i dispositivi perimetrali. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) i messaggi dall'hub IoT per la visualizzazione. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) fornisce report incentrati sull'azienda relativi a eventi di tipo out-of-stock. Power BI un'interfaccia dashboard facile da usare per visualizzare l'output di Analisi di flusso di Azure. |
| Azure Stack Edge o<br>Azure IoT Edge dispositivo | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) orchestra il runtime per i contenitori locali e gestisce la gestione e gli aggiornamenti dei dispositivi.|
| | Progetto Di Azure brainwave | In un Azure Stack Edge, [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) usa Field-Programmable Gate Array (FPGA) per accelerare l'inferenza di Machine Learning.|

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

La maggior parte dei modelli di Machine Learning può essere eseguita solo a un determinato numero di fotogrammi al secondo, a seconda dell'hardware fornito. Determinare la frequenza di campionamento ottimale dalle fotocamere per assicurarsi che la pipeline di Machine Learning non eserciti il backup. Tipi diversi di hardware gestiranno un numero diverso di fotocamere e frequenze dei fotogrammi.

### <a name="availability"></a>Disponibilità

È importante considerare cosa potrebbe accadere se il dispositivo perimetrale perde la connettività. Prendere in considerazione i dati che potrebbero essere persi Time Series Insights e Power BI dashboard. La soluzione di esempio fornita non è progettata per essere a disponibilità elevata.

### <a name="manageability"></a>Gestione

Questa soluzione può estendersi a molti dispositivi e posizioni, che potrebbero non essere più ingombrante. I servizi IoT di Azure possono portare automaticamente online nuovi percorsi e dispositivi e mantenerli aggiornati. È necessario seguire anche le procedure di governance dei dati appropriate.

### <a name="security"></a>Sicurezza

Questo modello gestisce dati potenzialmente sensibili. Assicurarsi che le chiavi vengano ruotate regolarmente e che le autorizzazioni per l'account Archiviazione di Azure e le condivisioni locali siano impostate correttamente.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sugli argomenti introdotti in questo articolo:
- In questo modello vengono usati più servizi correlati a IoT, tra cui Azure IoT Edge [,](/azure/iot-edge/) [hub IoT di Azure](/azure/iot-hub/)e [Azure Time Series Insights](/azure/time-series-insights/).
- Per altre informazioni su Microsoft Project Brainwave, vedere l'annuncio del [blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) ed estrarre il [video Azure Accelerated Machine Learning with Project Brainwave .](https://www.youtube.com/watch?v=DJfMobMjCX0)
- Per [altre informazioni sulle procedure consigliate](overview-app-design-considerations.md) e per ottenere risposte a eventuali domande aggiuntive, vedere Considerazioni sulla progettazione di app ibride.
- Vedere la [famiglia di prodotti e le soluzioni di Azure Stack](/azure-stack) per altre informazioni sull'intero portfolio di prodotti e soluzioni.

Quando si è pronti per testare l'esempio di soluzione, continuare con la guida alla distribuzione della soluzione di [inferenza di Edge ML.](https://aka.ms/edgeinferencingdeploy) Questa guida contiene istruzioni dettagliate per la distribuzione e il test dei componenti.
