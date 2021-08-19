---
title: Modello di scalabilità tra cloud in hub di Azure Stack
description: Informazioni su come creare un'app multi-cloud scalabile in Azure e hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 90e0c177b5eaee4d223b4613e0b2ddf385fa799c
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281262"
---
# <a name="cross-cloud-scaling-pattern"></a>Modello di scalabilità tra cloud

Aggiungere automaticamente risorse a un'app esistente per gestire un aumento del carico.

## <a name="context-and-problem"></a>Contesto e problema

L'app non può aumentare la capacità per soddisfare aumenti imprevisti della domanda. Questa mancanza di scalabilità fa in modo che gli utenti non raggiungano l'app durante i periodi di picco di utilizzo. L'app può essere a servizio di un numero fisso di utenti.

Le aziende globali richiedono app basate sul cloud sicure, affidabili e disponibili. Soddisfare gli aumenti della domanda e usare l'infrastruttura giusta per supportare tale domanda è fondamentale. Le aziende hanno difficoltà a bilanciare i costi e la manutenzione con la sicurezza dei dati aziendali, l'archiviazione e la disponibilità in tempo reale.

Potrebbe non essere possibile eseguire l'app nel cloud pubblico. Tuttavia, per l'azienda può non essere economicamente fattibile mantenere la capacità richiesta nell'ambiente locale per gestire i picchi di domanda per l'app. Con questo modello è possibile usare l'elasticità del cloud pubblico con la soluzione locale.

## <a name="solution"></a>Soluzione

Il modello di scalabilità tra cloud estende un'app che si trova in un cloud locale con risorse cloud pubbliche. Il modello viene attivato da un aumento o una diminuzione della domanda e, rispettivamente, aggiunge o rimuove risorse nel cloud. Queste risorse offrono ridondanza, disponibilità rapida e routing conforme a livello geografico.

![Modello di scalabilità tra cloud](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Questo modello si applica solo ai componenti senza stato dell'app.

## <a name="components"></a>Componenti

Il modello di scalabilità tra cloud è costituito dai componenti seguenti.

### <a name="outside-the-cloud"></a>All'esterno del cloud

#### <a name="traffic-manager"></a>Gestione traffico

Nel diagramma si trova all'esterno del gruppo di cloud pubblico, ma dovrebbe essere in grado di coordinare il traffico sia nel data center locale che nel cloud pubblico. Il servizio di bilanciamento del carico offre disponibilità elevata per l'app monitorando gli endpoint e fornendo la ridistribuzione del failover quando necessario.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

Il nome DNS (Domain Name System) è responsabile della conversione (o risoluzione) del nome di un sito Web o del servizio nel relativo indirizzo IP.

### <a name="cloud"></a>Cloud

#### <a name="hosted-build-server"></a>Server di compilazione ospitato

Un ambiente per l'hosting della pipeline di compilazione.

#### <a name="app-resources"></a>Risorse per le app

Le risorse dell'app devono essere in grado di aumentare e aumentare le risorse, ad esempio i set di scalabilità di macchine virtuali e i contenitori.

#### <a name="custom-domain-name"></a>Nome di dominio personalizzato

Usare un nome di dominio personalizzato per il routing delle richieste GLOB.

#### <a name="public-ip-addresses"></a>Indirizzi IP pubblici

Gli indirizzi IP pubblici vengono usati per instradare il traffico in ingresso tramite Gestione traffico all'endpoint delle risorse dell'app cloud pubblico.  

### <a name="local-cloud"></a>Cloud locale

#### <a name="hosted-build-server"></a>Server di compilazione ospitato

Un ambiente per l'hosting della pipeline di compilazione.

#### <a name="app-resources"></a>Risorse per le app

Le risorse dell'app devono poter essere ridimensionate e scalate orizzontalmente, ad esempio i set di scalabilità di macchine virtuali e i contenitori.

#### <a name="custom-domain-name"></a>Nome di dominio personalizzato

Usare un nome di dominio personalizzato per il routing delle richieste GLOB.

#### <a name="public-ip-addresses"></a>Indirizzi IP pubblici

Gli indirizzi IP pubblici vengono usati per instradare il traffico in ingresso tramite Gestione traffico all'endpoint delle risorse dell'app cloud pubblico.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Prima di decidere come implementare questo modello, considerare quanto segue:

### <a name="scalability"></a>Scalabilità

Il componente chiave della scalabilità tra cloud è la possibilità di offrire scalabilità su richiesta. Il ridimensionamento deve avvenire tra l'infrastruttura cloud pubblica e locale e fornire un servizio coerente e affidabile in base alla domanda.

### <a name="availability"></a>Disponibilità

Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata verificando la configurazione hardware locale e la distribuzione del software.

### <a name="manageability"></a>Gestione

Il modello tra cloud garantisce una gestione senza problemi e un'interfaccia familiare tra gli ambienti.

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

Usare questo schema:

- Quando è necessario aumentare la capacità dell'app con richieste impreviste o richieste periodiche.
- Quando non si vogliono investire in risorse che verranno usate solo durante i picchi. Pagare per le informazioni usate.

Questo modello non è consigliato quando:

- La soluzione richiede che gli utenti si connettono tramite Internet.
- L'azienda dispone di normative locali che richiedono che la connessione di origine provenirà da una chiamata in sede.
- Nella rete si verificano colli di bottiglia regolari che limitano le prestazioni del ridimensionamento.
- L'ambiente è disconnesso da Internet e non può raggiungere il cloud pubblico.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sugli argomenti introdotti in questo articolo:

- Vedere la [panoramica Gestione traffico di Azure per](/azure/traffic-manager/traffic-manager-overview) altre informazioni sul funzionamento di questo servizio di bilanciamento del carico del traffico basato su DNS.
- Per [altre informazioni sulle procedure consigliate e](overview-app-design-considerations.md) per ottenere risposte per eventuali domande aggiuntive, vedere Considerazioni sulla progettazione di applicazioni ibride.
- Vedere la [famiglia di prodotti e le soluzioni di Azure Stack](/azure-stack) per altre informazioni sull'intero portfolio di prodotti e soluzioni.

Quando si è pronti per testare l'esempio di soluzione, continuare con la guida alla distribuzione della soluzione di [scalabilità tra cloud.](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling) Questa guida contiene istruzioni dettagliate per la distribuzione e il test dei componenti. Si apprenderà come creare una soluzione tra cloud per fornire un processo attivato manualmente per il passaggio da un'app Web ospitata hub di Azure Stack a un'app Web ospitata in Azure. Si apprenderà anche come usare la scalabilità automatica tramite Gestione traffico, assicurando un'utilità cloud flessibile e scalabile in caso di carico.