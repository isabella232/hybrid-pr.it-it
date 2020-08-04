---
title: Modello DevOps nell'hub di Azure Stack
description: Informazioni sul modello DevOps per poter assicurare coerenza tra le distribuzioni in Azure e nell'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: e26056a9507a7467473b009725d4f210d9d59ec8
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477236"
---
# <a name="devops-pattern"></a>Modello DevOps

Eseguire il codice da un'unica posizione e distribuirlo in più destinazioni in ambienti di sviluppo, test e produzione che possono trovarsi nel data center locale, in cloud privati o nel cloud pubblico.

## <a name="context-and-problem"></a>Contesto e problema

La continuità, la sicurezza e l'affidabilità della distribuzione dell'applicazione sono aspetti essenziali per le organizzazioni e i team di sviluppo.

Per le app è spesso necessario effettuare il refactoring del codice perché possa essere eseguito in ogni ambiente di destinazione. Ciò significa che un'app non è completamente portatile. Deve essere aggiornata, testata e convalidata quando viene trasferita in ogni ambiente. Il codice scritto in un ambiente di sviluppo ad esempio deve essere riscritto per poter funzionare in un ambiente di test e scritto nuovamente quando viene infine usato in un ambiente di produzione. Questo codice è anche associato in modo specifico all'host, pertanto il costo e la complessità della gestione dell'app aumentano. Ogni versione dell'app è associata a ogni ambiente. La maggiore complessità e la duplicazione aumentano il rischio in termini di sicurezza e qualità del codice. Il codice non può nemmeno essere prontamente ridistribuito quando si rimuovono/ripristinano gli host non riusciti o si distribuiscono host aggiuntivi per gestire gli aumenti della richiesta.

## <a name="solution"></a>Soluzione

Il modello DevOps consente di compilare, testare e distribuire un'app che viene eseguita in più cloud. Questo modello unisce il processo di integrazione continua e il recapito continuo. Con l'integrazione continua, il codice viene compilato e testato ogni volta che un membro del team esegue il commit di una modifica nel controllo della versione. Il recapito continuo automatizza ogni passaggio da un ambiente di compilazione a uno di produzione. Insieme creano un processo di rilascio che supporta la distribuzione in ambienti diversi. Grazie a questo modello è possibile scrivere una bozza del codice, distribuire lo stesso codice in un ambiente locale, in diversi cloud privati e nei cloud pubblici. Le differenze nell'ambiente richiedono una modifica a un file di configurazione anziché modifiche al codice.

![Modello DevOps](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Grazie a un set coerente di strumenti di sviluppo in ambienti locali, cloud privati e cloud pubblici, è possibile implementare l'integrazione continua e il recapito continuo. Le app e i servizi che vengono distribuiti con il modello DevOps sono intercambiabili e possono essere eseguiti in una di queste posizioni, sfruttando le funzioni e le funzionalità di cloud pubblico e ambiente locale.

Usando una pipeline di versione DevOps è possibile eseguire le operazioni seguenti:

- Avviare una nuova compilazione in base ai commit del codice in un unico repository.
- Distribuire automaticamente il codice appena compilato nel cloud pubblico per il test di accettazione utente.
- Eseguire automaticamente la distribuzione in un cloud privato dopo che il codice ha superato il test.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Il modello DevOps è progettato per assicurare coerenza tra le distribuzioni indipendentemente dall'ambiente di destinazione. Le funzionalità sono tuttavia diverse negli ambienti cloud e locali. Tenere presente quanto segue:

- Le funzioni, gli endpoint, i servizi e le altre risorse della distribuzione sono disponibili nelle posizioni di distribuzione di destinazione?
- Gli artefatti della compilazione sono archiviati in posizioni accessibili in diversi cloud?
- I parametri di distribuzione funzioneranno in tutti gli ambienti di destinazione?
- Le proprietà specifiche delle risorse sono disponibili in tutti i cloud di destinazione?

Per altre informazioni, vedere [Sviluppare i modelli di Azure Resource Manager per la coerenza cloud](/azure/azure-resource-manager/templates-cloud-consistency).

Prima di decidere come implementare questo modello, è opportuno considerare anche quanto segue:

### <a name="scalability"></a>Scalabilità

I sistemi di automazione della distribuzione sono il punto di controllo principale nei modelli DevOps. Le implementazioni possono variare. La selezione delle dimensioni corrette del server dipende dalle dimensioni del carico di lavoro previsto. La scalabilità delle VM è più costosa rispetto ai contenitori. Per usare contenitori per il ridimensionamento, tuttavia, è necessario che il processo di compilazione venga eseguito con contenitori.

### <a name="availability"></a>Disponibilità

Nel contesto del modello DevPattern disponibilità significa poter ripristinare tutte le informazioni relative allo stato associate al flusso di lavoro, ad esempio i risultati dei test, le dipendenze del codice o altri artefatti. Per valutare i requisiti di disponibilità, considerare due metriche comuni:

- RTO (Recovery Time Objective, obiettivo del tempo di ripristino) specifica per quanto tempo è possibile non usare un sistema.

- RPO (Recovery Point Objective, obiettivo del punto di ripristino) indica quanti dati è possibile permettersi di perdere se un'interruzione del servizio interessa il sistema.

In pratica, RTO e RPO implicano ridondanza e backup. Nel cloud globale di Azure la disponibilità non riguarda il ripristino dell'hardware, che fa parte di Azure, ma piuttosto il mantenimento dello stato dei sistemi DevOps. Nell'hub di Azure Stack il ripristino dell'hardware può essere un punto da considerare.

Altri aspetti importanti della progettazione del sistema usato per l'automazione della distribuzione sono il controllo di accesso e la gestione corretta dei diritti necessari per distribuire i servizi negli ambienti cloud. Quali sono i diritti necessari per creare, eliminare o modificare le distribuzioni? In genere è ad esempio necessario un set di diritti per creare un gruppo di risorse in Azure e un altro set per distribuire i servizi nel gruppo di risorse.

### <a name="manageability"></a>Gestione

La progettazione di qualsiasi sistema basato sul modello DevOps deve prendere in considerazione l'automazione, la registrazione e gli avvisi per ogni servizio nel portfolio. Usare i servizi condivisi, un team dell'applicazione o entrambi e tenere traccia dei criteri di sicurezza e della governance.

Distribuire gli ambienti di produzione e gli ambienti di sviluppo/test in gruppi di risorse separati in Azure o nell'hub di Azure Stack. A questo punto è possibile monitorare le risorse di ogni ambiente ed eseguire il rollup dei costi di fatturazione in base al gruppo di risorse. È anche possibile eliminare un intero set di risorse, operazione molto utile nelle distribuzioni di prova.

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

Usare questo modello se:

- È possibile sviluppare il codice in un ambiente che soddisfa le esigenze degli sviluppatori e distribuirlo in un ambiente specifico della soluzione in cui potrebbe essere difficile sviluppare un nuovo codice.
- È possibile usare il codice e gli strumenti che gli sviluppatori desiderano, purché possano seguire il processo di integrazione continua e recapito continuo nel modello DevOps.

Questo modello non è consigliato:

- Se non è possibile automatizzare l'infrastruttura, il provisioning delle risorse, la configurazione, l'identità e le attività di sicurezza.
- Se i team non hanno accesso alle risorse del cloud ibrido per implementare un approccio di integrazione continua/sviluppo continuo (CI/CD).

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sugli argomenti introdotti in questo articolo:

- Vedere la [documentazione di Azure DevOps](/azure/devops) per altre informazioni su Azure DevOps e sugli strumenti correlati, tra cui Azure Repos e Azure Pipelines.
- Vedere la [famiglia di prodotti e le soluzioni di Azure Stack](/azure-stack) per altre informazioni sull'intero portfolio di prodotti e soluzioni.

Prima di testare l'esempio della soluzione, leggere la [guida alla distribuzione della soluzione CI/CD ibrida DevOps](https://aka.ms/hybriddevopsdeploy). Questa guida contiene istruzioni dettagliate per la distribuzione e il test dei componenti. Offre informazioni su come distribuire un'app in Azure e nell'hub di Azure Stack usando una pipeline di integrazione continua e recapito continuo (CI/CD) ibrida.
