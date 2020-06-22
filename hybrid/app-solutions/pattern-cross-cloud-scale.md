---
title: Modello di scalabilità tra cloud nell'hub Azure Stack
description: Informazioni su come creare un'app multicloud scalabile in Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911132"
---
# <a name="cross-cloud-scaling-pattern"></a>Modello di scalabilità tra cloud

Aggiungere automaticamente risorse a un'app esistente per supportare un aumento del carico.

## <a name="context-and-problem"></a>Contesto e problema

L'app non può aumentare la capacità per soddisfare un aumento imprevisto della richiesta. Questa mancanza di scalabilità comporta che gli utenti non raggiungano l'app durante i picchi di utilizzo. L'app può servire un numero fisso di utenti.

Le aziende globali richiedono app sicure, affidabili e disponibili basate sul cloud. La riunione aumenta la domanda e l'uso dell'infrastruttura corretta per supportare tale richiesta è fondamentale. Le aziende faticano a bilanciare i costi e la manutenzione grazie alla sicurezza dei dati aziendali, all'archiviazione e alla disponibilità in tempo reale.

Potrebbe non essere possibile eseguire l'app nel cloud pubblico. Tuttavia, potrebbe non essere economicamente fattibile per l'azienda mantenere la capacità richiesta nell'ambiente locale per gestire i picchi di domanda per l'app. Con questo modello, è possibile usare l'elasticità del cloud pubblico con la soluzione locale.

## <a name="solution"></a>Soluzione

Il modello di scalabilità tra cloud estende un'app che si trova in un cloud locale con risorse di cloud pubblico. Il modello viene attivato da un aumento o una diminuzione della domanda e, rispettivamente, aggiunge o rimuove le risorse nel cloud. Queste risorse offrono ridondanza, disponibilità rapida e routing conforme a geografica.

![Modello di scalabilità tra cloud](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Questo modello si applica solo ai componenti senza stato dell'app.

## <a name="components"></a>Componenti

Il modello di scalabilità tra cloud è costituito dai componenti seguenti.

### <a name="outside-the-cloud"></a>All'esterno del cloud

#### <a name="traffic-manager"></a>Gestione traffico

Nel diagramma, si trova all'esterno del gruppo di cloud pubblico, ma è necessario che sia in grado di coordinare il traffico sia nel Data Center locale che nel cloud pubblico. Il servizio di bilanciamento garantisce la disponibilità elevata per l'app monitorando gli endpoint e fornendo la ridistribuzione del failover quando necessario.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

Il nome DNS (Domain Name System) è responsabile della conversione (o risoluzione) del nome di un sito Web o del servizio nel relativo indirizzo IP.

### <a name="cloud"></a>Cloud

#### <a name="hosted-build-server"></a>Server di compilazione ospitato

Ambiente per l'hosting della pipeline di compilazione.

#### <a name="app-resources"></a>Risorse per le app

Le risorse dell'app devono essere in grado di eseguire la scalabilità orizzontale e orizzontale, ad esempio i set di scalabilità di macchine virtuali e i contenitori.

#### <a name="custom-domain-name"></a>Nome di dominio personalizzato

Usare un nome di dominio personalizzato per il routing delle richieste glob.

#### <a name="public-ip-addresses"></a>Indirizzi IP pubblici

Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.  

### <a name="local-cloud"></a>Cloud locale

#### <a name="hosted-build-server"></a>Server di compilazione ospitato

Ambiente per l'hosting della pipeline di compilazione.

#### <a name="app-resources"></a>Risorse per le app

Le risorse dell'app richiedono la possibilità di scalare in orizzontale e in orizzontale, come i set di scalabilità di macchine virtuali e i contenitori.

#### <a name="custom-domain-name"></a>Nome di dominio personalizzato

Usare un nome di dominio personalizzato per il routing delle richieste glob.

#### <a name="public-ip-addresses"></a>Indirizzi IP pubblici

Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Prima di decidere come implementare questo modello, considerare quanto segue:

### <a name="scalability"></a>Scalabilità

Il componente chiave del ridimensionamento tra cloud è la possibilità di fornire scalabilità su richiesta. La scalabilità deve essere eseguita tra l'infrastruttura cloud pubblica e quella locale e fornire un servizio coerente e affidabile in base alla richiesta.

### <a name="availability"></a>Disponibilità

Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata tramite la configurazione hardware locale e la distribuzione software.

### <a name="manageability"></a>Gestione

Il modello tra cloud garantisce una gestione semplice e un'interfaccia familiare tra gli ambienti.

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

Usare questo schema:

- Quando è necessario aumentare la capacità dell'app con richieste impreviste o richieste periodiche nella richiesta.
- Quando non si vuole investire in risorse che verranno usate solo durante i picchi. Pagare per le risorse usate.

Questo modello non è consigliato nei casi seguenti:

- Per la soluzione è necessario che gli utenti si connettano tramite Internet.
- L'azienda dispone di normative locali che richiedono che la connessione di origine provenga da una chiamata al sito.
- La rete presenta colli di bottiglia regolari che limitano le prestazioni del ridimensionamento.
- L'ambiente è disconnesso da Internet e non può raggiungere il cloud pubblico.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Vedere [Panoramica di gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview) per altre informazioni sul funzionamento di questo servizio di bilanciamento del carico del traffico basato su DNS.
- Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte per eventuali domande aggiuntive, vedere [considerazioni sulla progettazione di applicazioni ibride](overview-app-design-considerations.md) .
- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione di soluzioni di scalabilità tra cloud](solution-deployment-guide-cross-cloud-scaling.md). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di. Si apprenderà come creare una soluzione tra cloud per fornire un processo attivato manualmente per passare da un'app Web ospitata nell'hub Azure Stack a un'app Web ospitata in Azure. Si apprenderà anche come usare la scalabilità automatica tramite Gestione traffico, garantendo un'utilità cloud flessibile e scalabile quando sono sotto carico.
