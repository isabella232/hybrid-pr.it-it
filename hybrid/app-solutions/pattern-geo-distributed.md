---
title: Modello di app con distribuzione geografica nell'hub Azure Stack
description: Informazioni sul modello di app con distribuzione geografica per i dispositivi perimetrali intelligenti con Azure e hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910880"
---
# <a name="geo-distributed-app-pattern"></a>Modello di app con distribuzione geografica

Informazioni su come fornire gli endpoint dell'app in più aree e instradare il traffico utente in base alle esigenze di posizione e conformità.

## <a name="context-and-problem"></a>Contesto e problema

Le organizzazioni con aree geografiche di ampia portata si sforzano di distribuire in modo sicuro e accurato e di abilitare l'accesso ai dati garantendo al tempo stesso i livelli di sicurezza, conformità e prestazioni necessari per ogni utente, posizione e dispositivo tra i bordi.

## <a name="solution"></a>Soluzione

Il modello di routing del traffico geografico dell'hub Azure Stack o le app con distribuzione geografica consente di indirizzare il traffico a endpoint specifici in base a diverse metriche. La creazione di una gestione traffico con routing basato su geografia e configurazione degli endpoint instrada il traffico agli endpoint in base ai requisiti internazionali, alle normative aziendali e internazionali e alle esigenze dei dati.

![Modello con distribuzione geografica](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Componenti

### <a name="outside-the-cloud"></a>All'esterno del cloud

#### <a name="traffic-manager"></a>Gestione traffico

Nel diagramma, gestione traffico si trova all'esterno del cloud pubblico, ma è necessario che sia in grado di coordinare il traffico sia nel Data Center locale che nel cloud pubblico. Il servizio di bilanciamento instrada il traffico verso le posizioni geografiche.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

Il nome DNS (Domain Name System) è responsabile della conversione (o risoluzione) del nome di un sito Web o del servizio nel relativo indirizzo IP.

### <a name="public-cloud"></a>Cloud pubblico

#### <a name="cloud-endpoint"></a>Endpoint cloud

Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.  

### <a name="local-clouds"></a>Cloud locali

#### <a name="local-endpoint"></a>Endpoint locale

Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Prima di decidere come implementare questo modello, considerare quanto segue:

### <a name="scalability"></a>Scalabilità

Il modello gestisce il routing del traffico geografico anziché il ridimensionamento per soddisfare gli aumenti del traffico. Tuttavia, è possibile combinare questo modello con altre soluzioni di Azure e locali. Questo modello, ad esempio, può essere usato con il modello di scalabilità tra cloud.

### <a name="availability"></a>Disponibilità

Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata tramite la configurazione hardware locale e la distribuzione software.

### <a name="manageability"></a>Gestione

Il modello garantisce una gestione semplice e un'interfaccia familiare tra gli ambienti.

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

- L'organizzazione dispone di rami internazionali che richiedono criteri personalizzati di sicurezza e distribuzione a livello di area.
- Ogni ufficio della mia organizzazione effettua il pull dei dati relativi a dipendenti, aziende e strutture, richiedendo attività di Reporting per normative locali e fuso orario.
- I requisiti a scalabilità elevata possono essere soddisfatti con la scalabilità orizzontale delle app, con più distribuzioni di app eseguite in una singola area e tra aree per gestire requisiti di carico estremi.
- Le app devono essere a disponibilità elevata e rispondere alle richieste dei client anche in caso di interruzioni in una singola area.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Vedere [Panoramica di gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview) per altre informazioni sul funzionamento di questo servizio di bilanciamento del carico del traffico basato su DNS.
- Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte per eventuali domande aggiuntive, vedere [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) .
- Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per testare l'esempio di soluzione, continuare con la guida alla distribuzione di soluzioni per le [app con distribuzione geografica](solution-deployment-guide-geo-distributed.md). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di. Si apprenderà come indirizzare il traffico a endpoint specifici, in base a diverse metriche usando il modello di app con distribuzione geografica. La creazione di un profilo di gestione traffico con routing basato su geografia e configurazione dell'endpoint garantisce che le informazioni vengano indirizzate agli endpoint in base ai requisiti internazionali, alla regolamentazione aziendale e internazionale e alle esigenze dei dati.
