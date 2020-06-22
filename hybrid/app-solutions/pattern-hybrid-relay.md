---
title: Modello di inoltro ibrido in Azure e hub Azure Stack
description: Usare il modello di inoltro ibrido in Azure e Azure Stack hub per connettersi alle risorse perimetrali protette da firewall.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911146"
---
# <a name="hybrid-relay-pattern"></a>Modello di inoltro ibrido

Informazioni su come connettersi a risorse o dispositivi perimetrali protetti da firewall usando il modello di inoltro ibrido e il relè di Azure.

## <a name="context-and-problem"></a>Contesto e problema

I dispositivi perimetrali sono spesso protetti da un firewall aziendale o da un dispositivo NAT. Sebbene siano sicure, potrebbero non essere in grado di comunicare con il cloud pubblico o i dispositivi perimetrali in altre reti aziendali. Potrebbe essere necessario esporre determinate porte e funzionalità agli utenti nel cloud pubblico in modo sicuro.

## <a name="solution"></a>Soluzione

Il modello di inoltro ibrido usa il relè di Azure per stabilire un tunnel di WebSocket tra due endpoint che non possono comunicare direttamente. I dispositivi che non sono locali ma che devono connettersi a un endpoint locale si connetteranno a un endpoint nel cloud pubblico. Questo endpoint reindirizza il traffico su route predefinite su un canale sicuro. Un endpoint all'interno dell'ambiente locale riceve il traffico e lo instrada alla destinazione corretta.

![architettura della soluzione modello di inoltro ibrido](media/pattern-hybrid-relay/solution-architecture.png)

Ecco come funziona il modello di inoltro ibrido:

1. Un dispositivo si connette alla macchina virtuale (VM) in Azure, su una porta predefinita.
2. Il traffico viene inoltrato al relè di Azure in Azure.
3. La VM nell'hub Azure Stack, che ha già stabilito una connessione di lunga durata al relè di Azure, riceve il traffico e lo inoltra alla destinazione.
4. Il servizio o l'endpoint locale elabora la richiesta.

## <a name="components"></a>Componenti

Questa soluzione USA i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Azure | Macchina virtuale di Azure | Una macchina virtuale di Azure fornisce un endpoint accessibile pubblicamente per la risorsa locale. |
| | Servizio di inoltro di Azure | Un [inoltro di Azure](/azure/azure-relay/) fornisce l'infrastruttura per la gestione del tunnel e della connessione tra la macchina virtuale di Azure e la macchina virtuale dell'hub Azure stack.|
| Hub Azure Stack | Calcolo | Una macchina virtuale Azure Stack Hub fornisce il lato server del tunnel di inoltro ibrido. |
| | Archiviazione | Il cluster del motore AKS distribuito in Azure Stack Hub fornisce un motore resiliente e scalabile per eseguire il API Viso contenitore.|

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

Questo modello consente solo i mapping delle porte 1:1 nel client e nel server. Se, ad esempio, la porta 80 viene sottoferita a tunneling per un servizio nell'endpoint di Azure, non può essere usata per un altro servizio. I mapping delle porte devono essere pianificati di conseguenza. Il relay e le macchine virtuali di Azure devono essere ridimensionati in modo appropriato per gestire il traffico.

### <a name="availability"></a>Disponibilità

Questi tunnel e connessioni non sono ridondanti. Per garantire la disponibilità elevata, è opportuno implementare il codice di controllo degli errori. Un'altra opzione consiste nel disporre di un pool di VM connesse al servizio di inoltro di Azure dietro un servizio di bilanciamento del carico.

### <a name="manageability"></a>Gestione

Questa soluzione può estendersi a molti dispositivi e posizioni, che potrebbero essere difficili da gestire. I servizi Internet delle cose di Azure possono portare automaticamente nuovi percorsi e dispositivi e mantenerli aggiornati.

### <a name="security"></a>Sicurezza

Questo modello, come illustrato, consente l'accesso illimitato a una porta in un dispositivo interno dal perimetro. Prendere in considerazione l'aggiunta di un meccanismo di autenticazione al servizio nel dispositivo interno o davanti all'endpoint di inoltro ibrido.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Questo modello usa il relè di Azure. Per altre informazioni, vedere la [documentazione di inoltro di Azure](/azure/azure-relay/).
- Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte a eventuali domande aggiuntive, vedere [considerazioni sulla progettazione di applicazioni ibride](overview-app-design-considerations.md) .
- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione di soluzioni di inoltro ibrido](https://aka.ms/hybridrelaydeployment). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.