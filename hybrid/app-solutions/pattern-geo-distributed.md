---
title: Modello di app con distribuzione geografica in hub di Azure Stack
description: Informazioni sul modello di app con distribuzione geografica per il perimetro intelligente usando Azure e hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281228"
---
# <a name="geo-distributed-app-pattern"></a>Modello di app con distribuzione geografica

Informazioni su come fornire endpoint dell'app in più aree e indirizzare il traffico utente in base alla posizione e alle esigenze di conformità.

## <a name="context-and-problem"></a>Contesto e problema

Le organizzazioni con aree geografiche di vasta portata si impegnano per distribuire e abilitare l'accesso ai dati in modo sicuro e accurato garantendo al tempo stesso i livelli di sicurezza, conformità e prestazioni necessari per utente, posizione e dispositivo oltre i confini.

## <a name="solution"></a>Soluzione

Il hub di Azure Stack di routing del traffico geografico, o app con distribuzione geografica, consente di indirizzare il traffico a endpoint specifici in base a diverse metriche. La creazione di un Gestione traffico con routing basato su aree geografiche e la configurazione degli endpoint indirizza il traffico agli endpoint in base ai requisiti a livello di area, alle normative aziendali e internazionali e alle esigenze dei dati.

![Modello con distribuzione geografica](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Componenti

### <a name="outside-the-cloud"></a>All'esterno del cloud

#### <a name="traffic-manager"></a>Gestione traffico

Nel diagramma, Gestione traffico si trova all'esterno del cloud pubblico, ma deve essere in grado di coordinare il traffico sia nel data center locale che nel cloud pubblico. Il servizio di bilanciamento instrada il traffico alle posizioni geografiche.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

Il nome DNS (Domain Name System) è responsabile della conversione (o risoluzione) del nome di un sito Web o del servizio nel relativo indirizzo IP.

### <a name="public-cloud"></a>Cloud pubblico

#### <a name="cloud-endpoint"></a>Endpoint cloud

Gli indirizzi IP pubblici vengono usati per instradare il traffico in ingresso tramite Gestione traffico all'endpoint delle risorse dell'app cloud pubblica.  

### <a name="local-clouds"></a>Cloud locali

#### <a name="local-endpoint"></a>Endpoint locale

Gli indirizzi IP pubblici vengono usati per instradare il traffico in ingresso tramite Gestione traffico all'endpoint delle risorse dell'app cloud pubblica.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Prima di decidere come implementare questo modello, considerare quanto segue:

### <a name="scalability"></a>Scalabilità

Il modello gestisce il routing del traffico geografico anziché il ridimensionamento per soddisfare gli aumenti del traffico. Tuttavia, è possibile combinare questo modello con altre soluzioni di Azure e locali. Ad esempio, questo modello può essere usato con il modello di scalabilità tra cloud.

### <a name="availability"></a>Disponibilità

Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata verificando la configurazione hardware locale e la distribuzione del software.

### <a name="manageability"></a>Gestione

Il modello garantisce una gestione semplice e un'interfaccia familiare tra gli ambienti.

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

- L'organizzazione ha filiali internazionali che richiedono criteri di distribuzione e sicurezza a livello di regione personalizzati.
- Ogni ufficio dell'organizzazione estrae i dati relativi a dipendenti, aziende e strutture, richiedendo attività di creazione di report in base alle normative locali e al fuso orario.
- I requisiti di scalabilità elevata possono essere soddisfatti scalando orizzontalmente le app, con più distribuzioni di app effettuate all'interno di una singola area e tra aree per gestire requisiti di carico estremi.
- Le app devono essere a disponibilità elevata e reattive alle richieste client anche in caso di interruzioni in una singola area.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sugli argomenti introdotti in questo articolo:

- Vedere la [Gestione traffico di Azure panoramica per](/azure/traffic-manager/traffic-manager-overview) altre informazioni sul funzionamento di questo servizio di bilanciamento del carico del traffico basato su DNS.
- Per [altre informazioni sulle procedure consigliate](overview-app-design-considerations.md) e per ottenere risposte per eventuali domande aggiuntive, vedere Considerazioni sulla progettazione di app ibride.
- Vedere la [famiglia di prodotti e le soluzioni di Azure Stack](/azure-stack) per altre informazioni sull'intero portfolio di prodotti e soluzioni.

Quando si è pronti per testare l'esempio di soluzione, continuare con la guida alla distribuzione della soluzione [di app con distribuzione geografica.](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed) Questa guida contiene istruzioni dettagliate per la distribuzione e il test dei componenti. Si apprenderà come indirizzare il traffico a endpoint specifici, in base a varie metriche usando il modello di app con distribuzione geografica. La creazione di un profilo di Gestione traffico con routing geografico e configurazione degli endpoint garantisce che le informazioni vengano indirizzate agli endpoint in base ai requisiti internazionali, alle normative aziendali e internazionali e alle esigenze a livello di dati.