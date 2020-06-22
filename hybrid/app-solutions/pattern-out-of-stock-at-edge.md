---
title: Rilevamento scorte esaurite con Azure e Azure Stack Edge
description: Informazioni su come usare Azure e i servizi di Azure Stack Edge per implementare il rilevamento delle scorte.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911188"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Rilevamento scorte esaurito al modello perimetrale

Questo modello illustra come determinare se gli scaffali hanno esaurito le scorte usando un Azure Stack Edge o un dispositivo Azure IoT Edge e telecamere di rete.

## <a name="context-and-problem"></a>Contesto e problema

Le vendite al dettaglio fisico vengono perse perché quando i clienti cercano un elemento, non è presente sullo scaffale. Tuttavia, l'elemento potrebbe essere stato riportato nella parte posteriore dell'archivio e non è stato rifornito. Gli archivi desiderano utilizzare il personale in modo più efficiente e ricevere notifiche automatiche quando gli elementi devono essere riforniti.

## <a name="solution"></a>Soluzione

L'esempio di soluzione USA un dispositivo perimetrale, come un Azure Stack Edge in ogni negozio, che elabora in modo efficiente i dati dalle fotocamere nello Store. Questa progettazione ottimizzata consente agli archivi di inviare solo gli eventi e le immagini rilevanti al cloud. La progettazione consente di risparmiare larghezza di banda, spazio di archiviazione e garantire la privacy dei clienti. Poiché i frame vengono letti da ogni fotocamera, un modello ML elabora l'immagine e restituisce tutte le aree di inventario. L'immagine e le aree esaurite vengono visualizzate in un'app Web locale. Questi dati possono essere inviati a un ambiente Time Series Insight per visualizzare informazioni dettagliate in Power BI.

![Inventario esaurito all'architettura della soluzione perimetrale](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Ecco come funziona la soluzione:

1. Le immagini vengono acquisite da una videocamera di rete tramite HTTP o RTSP.
2. L'immagine viene ridimensionata e inviata al driver di inferenza, che comunica con il modello ML per determinare se sono presenti immagini predefinite.
3. Il modello ML restituisce tutte le aree riservate.
4. Il driver di inferenza carica l'immagine non elaborata in un BLOB (se specificato) e invia i risultati del modello all'hub di Azure e a un processore del rettangolo di delimitazione nel dispositivo.
5. Il processore del rettangolo di delimitazione aggiunge i riquadri all'immagine e memorizza nella cache il percorso dell'immagine in un database in memoria.
6. L'app Web esegue una query per le immagini e le Mostra nell'ordine in cui sono state ricevute.
7. I messaggi dall'hub Internet vengono aggregati in Time Series Insights.
8. Power BI Visualizza un report interattivo degli elementi esauriti nel tempo con i dati provenienti da Time Series Insights.


## <a name="components"></a>Componenti

Questa soluzione USA i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Hardware locale | Videocamera di rete | È necessaria una videocamera di rete, con un feed HTTP o RTSP per fornire le immagini per l'inferenza. |
| Azure | Hub IoT Azure | L' [Hub Azure](/azure/iot-hub/) Internet gestisce il provisioning dei dispositivi e la messaggistica per i dispositivi perimetrali. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) archivia i messaggi dall'hub Internet per la visualizzazione. |
|  | Power BI | [Microsoft Power bi](https://powerbi.microsoft.com/) fornisce report specifici dell'azienda per gli eventi di esaurimento delle scorte. Power BI offre un'interfaccia Dashboard di facile utilizzo per la visualizzazione dell'output di analisi di flusso di Azure. |
| Azure Stack Edge o<br>Dispositivo Azure IoT Edge | Azure IoT Edge | [Azure IOT Edge](/azure/iot-edge/) orchestra il runtime per i contenitori locali e gestisce gli aggiornamenti e la gestione dei dispositivi.|
| | Progetto di Azure cerebrale | In un dispositivo Azure Stack Edge, il [progetto](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) incrociato di controllo Usa matrici di controllo (FPGA) programmabili per accelerare l'inferenza di ml.|

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

La maggior parte dei modelli di apprendimento automatico può essere eseguita solo a un certo numero di frame al secondo, a seconda dell'hardware fornito. Determinare la frequenza di campionamento ottimale dalle fotocamere per assicurarsi che la pipeline ML non venga sottoposta a backup. Tipi diversi di hardware gestiranno diversi numeri di fotocamere e frequenze dei fotogrammi.

### <a name="availability"></a>Disponibilità

È importante considerare cosa può accadere se il dispositivo perimetrale perde la connettività. Considerare i dati che potrebbero andare perduti dal dashboard Time Series Insights e Power BI. La soluzione di esempio fornita non è progettata per essere a disponibilità elevata.

### <a name="manageability"></a>Gestione

Questa soluzione può estendersi a molti dispositivi e posizioni, che potrebbero essere difficili da gestire. I servizi Internet delle cose di Azure possono portare automaticamente nuovi percorsi e dispositivi e mantenerli aggiornati. Devono essere seguite anche le procedure di governance dei dati appropriate.

### <a name="security"></a>Sicurezza

Questo modello gestisce dati potenzialmente sensibili. Verificare che le chiavi vengano ruotate regolarmente e che le autorizzazioni per l'account di archiviazione di Azure e le condivisioni locali siano impostate correttamente.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:
- In questo modello vengono usati più servizi correlati alle cose, tra cui [Azure IOT Edge](/azure/iot-edge/), l' [Hub Azure](/azure/iot-hub/)e l' [Azure Time Series Insights](/azure/time-series-insights/).
- Per altre informazioni su Microsoft Project scondel, vedere [l'annuncio del Blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) ed estrarre il [video di Azure Accelerated Machine Learning con Project onder](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Per altre informazioni sulle procedure consigliate e per ottenere risposte a eventuali domande aggiuntive, vedere [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) .
- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione dei dati a più livelli per la soluzione di analisi](https://aka.ms/edgeinferencingdeploy). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.
