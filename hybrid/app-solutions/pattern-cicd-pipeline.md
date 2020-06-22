---
title: Modello DevOps nell'hub Azure Stack
description: Informazioni sul modello DevOps in modo che sia possibile garantire la coerenza tra le distribuzioni in Azure e in hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 306cc9604a8e919724f9f76b7e5122d534d2d1ae
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911118"
---
# <a name="devops-pattern"></a>Modello DevOps

Eseguire il codice da un'unica posizione e distribuirla in più destinazioni negli ambienti di sviluppo, test e produzione che possono trovarsi nel Data Center locale, nei cloud privati o nel cloud pubblico.

## <a name="context-and-problem"></a>Contesto e problema

La continuità, la sicurezza e l'affidabilità della distribuzione dell'applicazione sono essenziali per le organizzazioni e per i team di sviluppo.

Le app spesso richiedono che il codice sottoposto a refactoring venga eseguito in ogni ambiente di destinazione. Ciò significa che un'app non è completamente portatile. Deve essere aggiornato, testato e convalidato mentre viene spostato in ogni ambiente. Il codice scritto in un ambiente di sviluppo, ad esempio, deve essere riscritto in modo da funzionare in un ambiente di test e riscritto quando si trova infine in un ambiente di produzione. Inoltre, questo codice è associato in modo specifico all'host. Questo aumenta il costo e la complessità della gestione dell'app. Ogni versione dell'app è associata a ogni ambiente. La maggiore complessità e la duplicazione aumentano il rischio di sicurezza e qualità del codice. Inoltre, il codice non può essere prontamente ridistribuito quando si rimuove il ripristino degli host non riusciti o si distribuiscono host aggiuntivi per gestire gli aumenti della richiesta.

## <a name="solution"></a>Soluzione

Il modello DevOps consente di compilare, testare e distribuire un'app in esecuzione su più cloud. Questo modello unisce la pratica dell'integrazione continua e del recapito continuo. Con l'integrazione continua, il codice viene compilato e testato ogni volta che un membro del team eseguirà il commit di una modifica nel controllo della versione. Il recapito continuo automatizza ogni passaggio da una compilazione a un ambiente di produzione. Insieme, questi processi creano un processo di rilascio che supporta la distribuzione in ambienti diversi. Con questo modello è possibile eseguire il drafting del codice e quindi distribuire lo stesso codice in un ambiente locale, in cloud privati diversi e nei cloud pubblici. Le differenze nell'ambiente richiedono una modifica a un file di configurazione anziché modifiche al codice.

![Modello DevOps](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Grazie a un set coerente di strumenti di sviluppo in ambienti locali, cloud privati e cloud pubblici, è possibile implementare una procedura di integrazione continua e recapito continuo. Le app e i servizi distribuiti con il modello DevOps sono intercambiabili e possono essere eseguiti in uno di questi percorsi, sfruttando le funzionalità e le funzionalità del cloud pubblico e locale.

L'uso di una pipeline di versione DevOps consente di:

- Avviare una nuova compilazione in base ai commit del codice in un unico repository.
- Distribuire automaticamente il codice appena compilato nel cloud pubblico per i test di accettazione degli utenti.
- Distribuisci automaticamente in un cloud privato dopo che il codice ha superato il test.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Il modello DevOps è progettato per garantire la coerenza tra le distribuzioni indipendentemente dall'ambiente di destinazione. Tuttavia, le funzionalità variano in ambienti cloud e locali. Tenere presente quanto segue:

- Le funzioni, gli endpoint, i servizi e altre risorse nella distribuzione sono disponibili nei percorsi di distribuzione di destinazione?
- Gli elementi di configurazione sono archiviati in percorsi accessibili tra cloud?
- I parametri di distribuzione funzioneranno in tutti gli ambienti di destinazione?
- Le proprietà specifiche delle risorse sono disponibili in tutti i cloud di destinazione?

Per altre informazioni, vedere [sviluppare modelli di Azure Resource Manager per la coerenza del cloud](https://docs.microsoft.com/azure/azure-resource-manager/templates-cloud-consistency).

Inoltre, tenere presenti i punti seguenti quando si decide come implementare questo modello:

### <a name="scalability"></a>Scalabilità

I sistemi di automazione della distribuzione sono il punto di controllo chiave nei modelli di DevOps. Le implementazioni possono variare. La selezione delle dimensioni del server corrette dipende dalle dimensioni del carico di lavoro previsto. La scalabilità delle VM è più costosa rispetto ai contenitori. Per usare contenitori per il ridimensionamento, tuttavia, è necessario che il processo di compilazione venga eseguito con contenitori.

### <a name="availability"></a>Disponibilità

La disponibilità nel contesto di DevPattern significa essere in grado di ripristinare tutte le informazioni sullo stato associate al flusso di lavoro, ad esempio i risultati dei test, le dipendenze del codice o altri artefatti. Per valutare i requisiti di disponibilità, considerare due metriche comuni:

- L'obiettivo del tempo di ripristino (RTO) specifica per quanto tempo è possibile passare senza un sistema.

- L'obiettivo del punto di ripristino (RPO) indica la quantità di dati che è possibile permettersi di perdere se un'alterazione del servizio influiscono sul sistema.

In pratica, RTO e RPO implicano ridondanza e backup. Nel cloud globale di Azure, la disponibilità non è una questione di recupero hardware, che fa parte di Azure, ma piuttosto di garantire la manutenzione dello stato dei sistemi DevOps. In Azure Stack Hub, il ripristino dell'hardware può essere una considerazione.

Un altro aspetto importante della progettazione del sistema usato per l'automazione della distribuzione è il controllo dell'accesso e la gestione corretta dei diritti necessari per distribuire i servizi negli ambienti cloud. Quali sono i diritti necessari per creare, eliminare o modificare le distribuzioni? Un set di diritti, ad esempio, è in genere necessario per creare un gruppo di risorse in Azure e un altro per distribuire i servizi nel gruppo di risorse.

### <a name="manageability"></a>Gestione

La progettazione di qualsiasi sistema basato sul modello DevOps deve prendere in considerazione l'automazione, la registrazione e gli avvisi per ogni servizio nel portfolio. Usare i servizi condivisi, un team di applicazioni o entrambi e tenere traccia anche dei criteri di sicurezza e della governance.

Distribuire ambienti di produzione e ambienti di sviluppo/test in gruppi di risorse separati in Azure o Azure Stack Hub. È quindi possibile monitorare le risorse di ogni ambiente ed eseguire il rollup dei costi di fatturazione in base al gruppo di risorse. È anche possibile eliminare un intero set di risorse, operazione molto utile nelle distribuzioni di prova.

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

Usare questo modello se:

- È possibile sviluppare codice in un ambiente che soddisfi le esigenze degli sviluppatori e distribuirlo in un ambiente specifico della soluzione, in cui potrebbe essere difficile sviluppare un nuovo codice.
- È possibile usare il codice e gli strumenti che gli sviluppatori desiderano, purché siano in grado di seguire il processo di integrazione continua e recapito continuo nel modello DevOps.

Questo modello non è consigliato:

- Se non è possibile automatizzare l'infrastruttura, il provisioning delle risorse, la configurazione, l'identità e le attività di sicurezza.
- Se i team non hanno accesso alle risorse del cloud ibrido per implementare un approccio continuo di integrazione/sviluppo continuo (CI/CD).

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Per altre informazioni su Azure DevOps e sugli strumenti correlati, tra cui Azure Repos e Azure Pipelines, vedere la [documentazione di Azure DevOps](/azure/devops) .
- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione della soluzione ci/CD ibrido DevOps](https://aka.ms/hybriddevopsdeploy). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di. Si apprenderà come distribuire un'app in Azure e Azure Stack hub usando una pipeline di integrazione continua/distribuzione continua (CI/CD) ibrida.
