---
title: Dati a livelli per il modello di analisi con Azure e hub Azure Stack
description: Informazioni su come usare Azure e hub Azure Stack per implementare una soluzione di dati a più livelli nel cloud ibrido.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911244"
---
# <a name="tiered-data-for-analytics-pattern"></a>Dati a livelli per il modello di analisi

Questo modello illustra come usare Azure Stack Hub e Azure per organizzare, analizzare, elaborare, purificare e archiviare i dati in più posizioni locali e nel cloud.

## <a name="context-and-problem"></a>Contesto e problema

Uno dei problemi che si riferiscono alle organizzazioni aziendali nel panorama tecnologico moderno riguarda l'archiviazione, l'elaborazione e l'analisi dei dati protetti. Sono incluse le considerazioni seguenti:

- contenuto dati
- posizione
- requisiti di sicurezza e privacy
- autorizzazioni di accesso
- manutenzione
- warehousing di archiviazione

Azure, in combinazione con Azure Stack Hub, risolve i problemi relativi ai dati e offre soluzioni a basso costo. Questa soluzione è particolarmente indicata tramite una società di produzione distribuita o logistica.

La soluzione è basata sullo scenario seguente:

- Un'organizzazione di produzione multiramo di grandi dimensioni.
- L'archiviazione, l'elaborazione e la distribuzione dei dati rapidi e sicuri tra i percorsi remoti globali e la sede centrale sono obbligatori.
- Attività di dipendenti e macchinari, informazioni sulle funzionalità e dati di Business Reporting che devono rimanere protetti. I dati devono essere distribuiti in modo appropriato e soddisfare i criteri di conformità regionali e le normative del settore.

## <a name="solution"></a>Soluzione

L'uso di ambienti locali e cloud pubblici soddisfa le esigenze delle aziende con più funzionalità. Azure Stack Hub offre una soluzione rapida, sicura e flessibile per la raccolta, l'elaborazione, l'archiviazione e la distribuzione di dati locali e remoti. Questo modello è particolarmente utile quando la sicurezza, la riservatezza, i criteri aziendali e i requisiti normativi possono variare tra località e utenti.

![Modello di dati a livelli per l'architettura della soluzione di analisi](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Componenti

Questo modello usa i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Azure | Archiviazione | Un account di [archiviazione di Azure](/azure/storage/) fornisce un endpoint sterile per l'utilizzo dei dati. Archiviazione di Azure è la soluzione di archiviazione cloud Microsoft per i moderni scenari di archiviazione dati. Archiviazione di Azure offre un archivio oggetti a scalabilità elevata per oggetti dati e un servizio file system per il cloud. Fornisce anche un archivio di messaggistica per la messaggistica affidabile e un archivio NoSQL. |
| Hub Azure Stack | Archiviazione | Per più servizi viene usato un account di [archiviazione Hub Azure stack](/azure-stack/user/azure-stack-storage-overview) :<br><br>- **Archiviazione BLOB** per l'archiviazione di dati non elaborati. L'archiviazione BLOB può archiviare qualsiasi tipo di dati di testo o binari, ad esempio un documento, un file multimediale o un programma di installazione di app. Ogni BLOB è organizzato in un contenitore. I contenitori offrono un modo utile per assegnare i criteri di sicurezza a gruppi di oggetti. Un account di archiviazione può contenere un numero qualsiasi di contenitori e un contenitore può contenere un numero qualsiasi di BLOB, fino al limite di capacità di 500 TB dell'account di archiviazione.<br>- **Archiviazione BLOB** per l'archivio dati. Sono disponibili vantaggi per l'archiviazione a basso costo per l'archiviazione di dati sporadici. Esempi di dati interessanti includono backup, contenuto multimediale, dati scientifici, conformità e dati di archiviazione. In generale, tutti i dati a cui si accede raramente sono considerati archiviazione ad accesso sporadico. Suddivisione in livelli di dati in base ad attributi quali la frequenza di accesso e il periodo di conservazione. I dati dei clienti sono accessibili raramente, ma richiedono latenza e prestazioni simili ai dati sensibili.<br>- **Archiviazione di Accodamento** per l'archiviazione dei dati elaborati. L'archiviazione di Accodamento offre la messaggistica cloud tra i componenti dell'app. Per la progettazione di app per la scalabilità, i componenti delle app sono spesso separati per poterli ridimensionare in modo indipendente. L'archiviazione di Accodamento offre la messaggistica asincrona per la comunicazione tra i componenti dell'app, che siano in esecuzione nel cloud, nel desktop, in un server locale o in un dispositivo mobile. Archiviazione code supporta anche la gestione di attività asincrone e la creazione di flussi di lavoro dei processi. |
| | Funzioni di Azure | Il servizio [funzioni di Azure](/azure/azure-functions/) viene fornito dal [servizio app Azure nel](/azure-stack/operator/azure-stack-app-service-overview) provider di risorse dell'hub Azure stack. Funzioni di Azure consente di eseguire il codice in un ambiente senza server semplice in risposta a una serie di eventi. Scalabilità di funzioni di Azure per soddisfare la domanda senza dover creare una macchina virtuale o pubblicare un'app Web, usando il linguaggio di programmazione che preferisci. Le funzioni vengono usate dalla soluzione per:<br><br>- **Immissione dati**<br>- **Sterilizzazione dei dati.** Le funzioni attivate manualmente possono eseguire l'elaborazione pianificata dei dati, la pulizia e l'archiviazione. Gli esempi possono includere scrub per i clienti notturni e l'elaborazione mensile dei report.|

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

Funzioni di Azure e scalabilità delle soluzioni di archiviazione per soddisfare le esigenze di elaborazione e volume dei dati. Per informazioni sulla scalabilità e destinazioni di Azure, vedere la [documentazione sulla scalabilità di archiviazione di Azure](/azure/storage/common/storage-scalability-targets).

### <a name="availability"></a>Disponibilità

L'archiviazione è la principale considerazione della disponibilità per questo modello. Per la distribuzione e l'elaborazione di volumi di dati di grandi dimensioni è necessaria una connessione tramite collegamenti rapidi.

### <a name="manageability"></a>Gestione

La gestibilità di questa soluzione dipende dagli strumenti di creazione usati e dal coinvolgimento del controllo del codice sorgente.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Vedere la documentazione di [archiviazione di Azure](/azure/storage/) e [funzioni di Azure](/azure/azure-functions/) . Questo modello consente di usare in modo intensivo gli account di archiviazione di Azure e funzioni di Azure in Azure e in hub Azure Stack.
- Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte a domande aggiuntive, vedere [considerazioni sulla progettazione di applicazioni ibride](overview-app-design-considerations.md) .
- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione dei dati a più livelli per la soluzione di analisi](https://aka.ms/tiereddatadeploy). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.
