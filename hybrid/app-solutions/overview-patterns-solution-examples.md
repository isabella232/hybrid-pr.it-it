---
title: Modelli ibridi ed esempi di soluzioni per Azure e hub di Azure Stack
description: Panoramica di modelli ibridi ed esempi di soluzioni per l'apprendimento e la creazione di soluzioni ibride in Azure e hub di Azure Stack.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343859"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Modelli ed esempi di soluzioni ibride per Azure e Azure Stack

Microsoft offre azure e Azure Stack prodotti e soluzioni come un unico ecosistema di Azure coerente. La Microsoft Azure Stack è un'estensione di Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Cloud ibrido e app ibride

Azure Stack offre la flessibilità cloud computing ambiente locale e perimetrale abilitando un *cloud ibrido.* hub di Azure Stack, Azure Stack HCI e Azure Stack Edge azure dal cloud ai data center sovrani, alle succursali, sul campo e oltre. Con questo set eterogeneo di funzionalità, è possibile:

- Riutilizzare il codice ed eseguire app native del cloud in modo coerente in Azure e negli ambienti locali.
- Eseguire carichi di lavoro virtualizzati tradizionali con connessioni facoltative ai servizi di Azure.
- Trasferire i dati nel cloud o mantenerla nel data center sovrano per mantenere la conformità.
- Eseguire carichi di lavoro di Machine Learning con accelerazione hardware, in contenitori o virtualizzati, il tutto nella rete perimetrale intelligente.

Le app che si estendono su cloud sono anche chiamate *app ibride*. È possibile creare app cloud ibride in Azure e distribuirle nel data center connesso o disconnesso ovunque ci si trovi.

Gli scenari di app ibride variano notevolmente con le risorse disponibili per lo sviluppo. Vengono inoltre riportate alcune considerazioni, ad esempio geografia, sicurezza, accesso a Internet e altro ancora. Anche se i modelli di soluzione e gli esempi descritti qui potrebbero non soddisfare tutti i requisiti, forniscono linee guida ed esempi da esplorare e riutilizzare durante l'implementazione di soluzioni ibride.

## <a name="solution-patterns"></a>Modelli di soluzione

I modelli di soluzione cull generalized repeatable design guidance,from real world customer scenarios and experiences. Un modello è astratto e può essere applicabile a diversi tipi di scenari o settori verticali. Ogni modello documenta il contesto e il problema e fornisce una panoramica di un esempio di soluzione. L'esempio di soluzione è inteso come possibile implementazione del modello.

Esistono due tipi di articoli sui modelli:

- Modello singolo: fornisce indicazioni di progettazione per un singolo scenario per utilizzo generico.
- Multi-pattern: fornisce indicazioni di progettazione in cui viene usata l'applicazione di più modelli. Questo modello è spesso necessario per risolvere scenari più complessi o problemi specifici del settore.

## <a name="solution-deployment-guides"></a>Guide alla distribuzione di soluzioni

Le guide dettagliate alla distribuzione sono utili per la distribuzione di un esempio di soluzione. La guida può anche fare riferimento a un esempio di codice complementare, archiviato nel repository di esempio [delle](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)soluzioni GitHub.

## <a name="next-steps"></a>Passaggi successivi

- Vedere la [famiglia di prodotti e le soluzioni di Azure Stack](/azure-stack) per altre informazioni sull'intero portfolio di prodotti e soluzioni.
- Esplorare le sezioni "Modelli" e "Guide alla distribuzione delle soluzioni" del sommario per altre informazioni su ognuno di essi.
- Leggere le [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) per esaminare i pilastri della qualità del software per la progettazione, la distribuzione e il funzionamento di app ibride.
- [Configurare un ambiente di sviluppo in Azure Stack](/azure-stack/user/azure-stack-dev-start) e distribuire la prima [app](/azure-stack/user/azure-stack-dev-start-deploy-app) in Azure Stack.
