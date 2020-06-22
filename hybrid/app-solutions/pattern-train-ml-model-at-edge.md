---
title: Eseguire il training del modello di Machine Learning al modello perimetrale
description: Informazioni su come eseguire il training del modello di Machine Learning al perimetro con Azure e hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911101"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>Eseguire il training del modello di Machine Learning al modello perimetrale

Generare modelli di Machine Learning (ML) portatili da dati che esistono solo in locale.

## <a name="context-and-problem"></a>Contesto e problema

Molte organizzazioni vogliono sbloccare informazioni approfondite dai dati locali o legacy usando gli strumenti che i loro data scientist conoscono. [Azure Machine Learning](/azure/machine-learning/) offre strumenti nativi del cloud per eseguire il training, l'ottimizzazione e la distribuzione di modelli di apprendimento avanzato e ml.  

Tuttavia, alcuni dati sono troppo grandi per l'invio al cloud o non possono essere inviati al cloud per motivi normativi. Con questo modello, i data scientist possono usare Azure Machine Learning per eseguire il training di modelli usando dati e calcolo locali.

## <a name="solution"></a>Soluzione

Il training al modello perimetrale usa una macchina virtuale (VM) in esecuzione nell'hub Azure Stack. La macchina virtuale viene registrata come destinazione di calcolo in Azure ML, consentendo l'accesso ai dati disponibili solo in locale. In questo caso i dati vengono archiviati nell'archivio BLOB di Azure Stack Hub.

Una volta eseguito il training del modello, questo viene registrato con Azure ML, in contenitori e aggiunto a un Container Registry di Azure per la distribuzione. Per questa iterazione del modello, la macchina virtuale di training dell'hub Azure Stack deve essere raggiungibile tramite la rete Internet pubblica.

[![Training del modello ML nell'architettura perimetrale](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

Ecco come funziona il modello:

1. La VM dell'hub Azure Stack viene distribuita e registrata come destinazione di calcolo con Azure ML.
2. In Azure ML viene creato un esperimento che usa la macchina virtuale Azure Stack Hub come destinazione di calcolo.
3. Una volta eseguito il training del modello, questo viene registrato e in contenitori.
4. Il modello può ora essere distribuito in percorsi locali o nel cloud.

## <a name="components"></a>Componenti

Questa soluzione USA i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure Machine Learning](/azure/machine-learning/) orchestra il training del modello ml. |
| | Registro Azure Container | Azure ML inserisce il modello in un contenitore e lo archivia in un [container Registry di Azure](/azure/container-registry/) per la distribuzione.|
| Hub Azure Stack | Servizio app | [Azure stack Hub con il servizio app](/azure-stack/operator/azure-stack-app-service-overview) fornisce la base per i componenti al perimetro. |
| | Calcolo | Per eseguire il training del modello ML viene usata una macchina virtuale Azure Stack Hub che esegue Ubuntu con Docker. |
| | Archiviazione | I dati privati possono essere ospitati nell'archivio BLOB dell'hub Azure Stack. |

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

Per consentire la scalabilità di questa soluzione, è necessario creare una macchina virtuale di dimensioni appropriate nell'hub Azure Stack per il training.

### <a name="availability"></a>Disponibilità

Assicurarsi che gli script di training e la macchina virtuale dell'hub Azure Stack dispongano dell'accesso ai dati locali usati per il training.

### <a name="manageability"></a>Gestione

Assicurarsi che i modelli e gli esperimenti siano registrati in modo appropriato, con versione e con tag per evitare confusione durante la distribuzione del modello.

### <a name="security"></a>Sicurezza

Questo modello consente ad Azure ML di accedere ai dati sensibili in locale. Assicurarsi che l'account usato per SSH nella macchina virtuale Azure Stack Hub disponga di una password complessa e che gli script di training non mantengano o caricano i dati nel cloud.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Vedere la [documentazione di Azure Machine Learning](/azure/machine-learning) per una panoramica di ml e argomenti correlati.
- Vedere [container Registry di Azure](/azure/container-registry/) per informazioni su come creare, archiviare e gestire immagini per le distribuzioni di contenitori.
- Per altre informazioni sul provider di risorse e su come eseguire la distribuzione, vedere [servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-overview) .
- Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte alle domande aggiuntive, vedere [considerazioni sulla progettazione di applicazioni ibride](overview-app-design-considerations.md) .
- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per testare l'esempio di soluzione, continuare con il [modello Train ml nella Guida alla distribuzione di Edge](https://aka.ms/edgetrainingdeploy). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.
