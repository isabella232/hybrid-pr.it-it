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
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="9f600-103">Modello di app con distribuzione geografica</span><span class="sxs-lookup"><span data-stu-id="9f600-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="9f600-104">Informazioni su come fornire gli endpoint dell'app in più aree e instradare il traffico utente in base alle esigenze di posizione e conformità.</span><span class="sxs-lookup"><span data-stu-id="9f600-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="9f600-105">Contesto e problema</span><span class="sxs-lookup"><span data-stu-id="9f600-105">Context and problem</span></span>

<span data-ttu-id="9f600-106">Le organizzazioni con aree geografiche di ampia portata si sforzano di distribuire in modo sicuro e accurato e di abilitare l'accesso ai dati garantendo al tempo stesso i livelli di sicurezza, conformità e prestazioni necessari per ogni utente, posizione e dispositivo tra i bordi.</span><span class="sxs-lookup"><span data-stu-id="9f600-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="9f600-107">Soluzione</span><span class="sxs-lookup"><span data-stu-id="9f600-107">Solution</span></span>

<span data-ttu-id="9f600-108">Il modello di routing del traffico geografico dell'hub Azure Stack o le app con distribuzione geografica consente di indirizzare il traffico a endpoint specifici in base a diverse metriche.</span><span class="sxs-lookup"><span data-stu-id="9f600-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="9f600-109">La creazione di una gestione traffico con routing basato su geografia e configurazione degli endpoint instrada il traffico agli endpoint in base ai requisiti internazionali, alle normative aziendali e internazionali e alle esigenze dei dati.</span><span class="sxs-lookup"><span data-stu-id="9f600-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Modello con distribuzione geografica](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="9f600-111">Componenti</span><span class="sxs-lookup"><span data-stu-id="9f600-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="9f600-112">All'esterno del cloud</span><span class="sxs-lookup"><span data-stu-id="9f600-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="9f600-113">Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="9f600-113">Traffic Manager</span></span>

<span data-ttu-id="9f600-114">Nel diagramma, gestione traffico si trova all'esterno del cloud pubblico, ma è necessario che sia in grado di coordinare il traffico sia nel Data Center locale che nel cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="9f600-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="9f600-115">Il servizio di bilanciamento instrada il traffico verso le posizioni geografiche.</span><span class="sxs-lookup"><span data-stu-id="9f600-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="9f600-116">Domain Name System (DNS)</span><span class="sxs-lookup"><span data-stu-id="9f600-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="9f600-117">Il nome DNS (Domain Name System) è responsabile della conversione (o risoluzione) del nome di un sito Web o del servizio nel relativo indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="9f600-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="9f600-118">Cloud pubblico</span><span class="sxs-lookup"><span data-stu-id="9f600-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="9f600-119">Endpoint cloud</span><span class="sxs-lookup"><span data-stu-id="9f600-119">Cloud Endpoint</span></span>

<span data-ttu-id="9f600-120">Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="9f600-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="9f600-121">Cloud locali</span><span class="sxs-lookup"><span data-stu-id="9f600-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="9f600-122">Endpoint locale</span><span class="sxs-lookup"><span data-stu-id="9f600-122">Local endpoint</span></span>

<span data-ttu-id="9f600-123">Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="9f600-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="9f600-124">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="9f600-124">Issues and considerations</span></span>

<span data-ttu-id="9f600-125">Prima di decidere come implementare questo modello, considerare quanto segue:</span><span class="sxs-lookup"><span data-stu-id="9f600-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="9f600-126">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="9f600-126">Scalability</span></span>

<span data-ttu-id="9f600-127">Il modello gestisce il routing del traffico geografico anziché il ridimensionamento per soddisfare gli aumenti del traffico.</span><span class="sxs-lookup"><span data-stu-id="9f600-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="9f600-128">Tuttavia, è possibile combinare questo modello con altre soluzioni di Azure e locali.</span><span class="sxs-lookup"><span data-stu-id="9f600-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="9f600-129">Questo modello, ad esempio, può essere usato con il modello di scalabilità tra cloud.</span><span class="sxs-lookup"><span data-stu-id="9f600-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="9f600-130">Disponibilità</span><span class="sxs-lookup"><span data-stu-id="9f600-130">Availability</span></span>

<span data-ttu-id="9f600-131">Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata tramite la configurazione hardware locale e la distribuzione software.</span><span class="sxs-lookup"><span data-stu-id="9f600-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="9f600-132">Gestione</span><span class="sxs-lookup"><span data-stu-id="9f600-132">Manageability</span></span>

<span data-ttu-id="9f600-133">Il modello garantisce una gestione semplice e un'interfaccia familiare tra gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="9f600-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="9f600-134">Quando usare questo modello</span><span class="sxs-lookup"><span data-stu-id="9f600-134">When to use this pattern</span></span>

- <span data-ttu-id="9f600-135">L'organizzazione dispone di rami internazionali che richiedono criteri personalizzati di sicurezza e distribuzione a livello di area.</span><span class="sxs-lookup"><span data-stu-id="9f600-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="9f600-136">Ogni ufficio della mia organizzazione effettua il pull dei dati relativi a dipendenti, aziende e strutture, richiedendo attività di Reporting per normative locali e fuso orario.</span><span class="sxs-lookup"><span data-stu-id="9f600-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="9f600-137">I requisiti a scalabilità elevata possono essere soddisfatti con la scalabilità orizzontale delle app, con più distribuzioni di app eseguite in una singola area e tra aree per gestire requisiti di carico estremi.</span><span class="sxs-lookup"><span data-stu-id="9f600-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="9f600-138">Le app devono essere a disponibilità elevata e rispondere alle richieste dei client anche in caso di interruzioni in una singola area.</span><span class="sxs-lookup"><span data-stu-id="9f600-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9f600-139">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="9f600-139">Next steps</span></span>

<span data-ttu-id="9f600-140">Per ulteriori informazioni sugli argomenti introdotti in questo articolo:</span><span class="sxs-lookup"><span data-stu-id="9f600-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="9f600-141">Vedere [Panoramica di gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview) per altre informazioni sul funzionamento di questo servizio di bilanciamento del carico del traffico basato su DNS.</span><span class="sxs-lookup"><span data-stu-id="9f600-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="9f600-142">Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte per eventuali domande aggiuntive, vedere [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="9f600-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="9f600-143">Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="9f600-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="9f600-144">Quando si è pronti per testare l'esempio di soluzione, continuare con la guida alla distribuzione di soluzioni per le [app con distribuzione geografica](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="9f600-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="9f600-145">Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.</span><span class="sxs-lookup"><span data-stu-id="9f600-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="9f600-146">Si apprenderà come indirizzare il traffico a endpoint specifici, in base a diverse metriche usando il modello di app con distribuzione geografica.</span><span class="sxs-lookup"><span data-stu-id="9f600-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="9f600-147">La creazione di un profilo di gestione traffico con routing basato su geografia e configurazione dell'endpoint garantisce che le informazioni vengano indirizzate agli endpoint in base ai requisiti internazionali, alla regolamentazione aziendale e internazionale e alle esigenze dei dati.</span><span class="sxs-lookup"><span data-stu-id="9f600-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
