---
title: Modelli ibridi ed esempi di soluzioni per Azure e hub Azure Stack
description: Panoramica di modelli ibridi ed esempi di soluzioni per l'apprendimento e la creazione di soluzioni ibride in Azure e Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ab0eb885e7b0fefaca8991522712652f979d8712
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910919"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>Modelli ibridi ed esempi di soluzioni per Azure e Azure Stack

Microsoft offre prodotti e soluzioni di Azure e Azure Stack come un ecosistema di Azure coerente. La famiglia di Microsoft Azure Stack è un'estensione di Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Cloud ibrido e app ibride

Azure Stack offre la flessibilità di cloud computing all'ambiente locale e al perimetro abilitando un *cloud ibrido*. Azure Stack Hub, Azure Stack HCI e Azure Stack Edge estendono Azure dal cloud nei data center sovrani, nelle succursali, nel campo e oltre. Con questo set di funzionalità diversificato, è possibile:

- Riutilizzare il codice ed eseguire app native del cloud in modo coerente in Azure e negli ambienti locali.
- Eseguire carichi di lavoro virtualizzati tradizionali con connessioni facoltative ai servizi di Azure.
- Trasferire i dati nel cloud o mantenerli nel Data Center sovrano per mantenere la conformità.
- Esegui carichi di lavoro di apprendimento automatico, in contenitori o virtualizzati con accelerazione hardware, tutto sul perimetro intelligente.

Le app che si estendono su cloud sono anche denominate *app ibride*. Puoi creare app cloud ibride in Azure e distribuirle nel tuo Data Center connesso o disconnesso situato ovunque ti trovi.

Gli scenari di app ibride variano significativamente con le risorse disponibili per lo sviluppo. Si estendono anche considerazioni quali geografia, sicurezza, accesso a Internet e altri. Anche se i modelli e le soluzioni descritti qui potrebbero non soddisfare tutti i requisiti, forniscono linee guida ed esempi per esplorare e riutilizzare durante l'implementazione di soluzioni ibride.

## <a name="design-patterns"></a>Modelli di progettazione

Modelli di progettazione abbattere le indicazioni generali di progettazione ripetibili, da scenari e esperienze reali dei clienti. Un modello è astratto, in modo che possa essere applicato a diversi tipi di scenari o settori verticali. Ogni modello documenta il contesto e il problema e fornisce una panoramica di un esempio di soluzione. L'esempio di soluzione è concepito come un'implementazione possibile del modello.

Esistono due tipi di articoli del modello:

- Modello singolo: fornisce indicazioni di progettazione per un singolo scenario per utilizzo generico.
- Multi-pattern: fornisce indicazioni di progettazione in cui viene usata l'applicazione di più modelli. Questo modello è spesso necessario per la risoluzione di scenari più complessi o problemi specifici del settore.

## <a name="solution-deployment-guides"></a>Guide alla distribuzione della soluzione

Guida dettagliata alla distribuzione di un esempio di soluzione. La guida può anche fare riferimento a un esempio di codice complementare, archiviato nel [repository di esempio delle soluzioni](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)github.

## <a name="next-steps"></a>Passaggi successivi

- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .
- Per ulteriori informazioni su ogni argomento, vedere le sezioni "Patterns" e "guide alla distribuzione della soluzione" del sommario.
- Leggere le [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) per esaminare i pilastri della qualità del software per la progettazione, la distribuzione e il funzionamento di app ibride.
- [Configura un ambiente di sviluppo in Azure stack](/azure-stack/user/azure-stack-dev-start.md) e [Distribuisci la tua prima app](/azure-stack/user/azure-stack-dev-start-deploy-app.md) in Azure stack.
