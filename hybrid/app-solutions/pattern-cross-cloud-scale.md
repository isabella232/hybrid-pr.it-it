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
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="89076-103">Modello di scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="89076-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="89076-104">Aggiungere automaticamente risorse a un'app esistente per supportare un aumento del carico.</span><span class="sxs-lookup"><span data-stu-id="89076-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="89076-105">Contesto e problema</span><span class="sxs-lookup"><span data-stu-id="89076-105">Context and problem</span></span>

<span data-ttu-id="89076-106">L'app non può aumentare la capacità per soddisfare un aumento imprevisto della richiesta.</span><span class="sxs-lookup"><span data-stu-id="89076-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="89076-107">Questa mancanza di scalabilità comporta che gli utenti non raggiungano l'app durante i picchi di utilizzo.</span><span class="sxs-lookup"><span data-stu-id="89076-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="89076-108">L'app può servire un numero fisso di utenti.</span><span class="sxs-lookup"><span data-stu-id="89076-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="89076-109">Le aziende globali richiedono app sicure, affidabili e disponibili basate sul cloud.</span><span class="sxs-lookup"><span data-stu-id="89076-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="89076-110">La riunione aumenta la domanda e l'uso dell'infrastruttura corretta per supportare tale richiesta è fondamentale.</span><span class="sxs-lookup"><span data-stu-id="89076-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="89076-111">Le aziende faticano a bilanciare i costi e la manutenzione grazie alla sicurezza dei dati aziendali, all'archiviazione e alla disponibilità in tempo reale.</span><span class="sxs-lookup"><span data-stu-id="89076-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="89076-112">Potrebbe non essere possibile eseguire l'app nel cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="89076-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="89076-113">Tuttavia, potrebbe non essere economicamente fattibile per l'azienda mantenere la capacità richiesta nell'ambiente locale per gestire i picchi di domanda per l'app.</span><span class="sxs-lookup"><span data-stu-id="89076-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="89076-114">Con questo modello, è possibile usare l'elasticità del cloud pubblico con la soluzione locale.</span><span class="sxs-lookup"><span data-stu-id="89076-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="89076-115">Soluzione</span><span class="sxs-lookup"><span data-stu-id="89076-115">Solution</span></span>

<span data-ttu-id="89076-116">Il modello di scalabilità tra cloud estende un'app che si trova in un cloud locale con risorse di cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="89076-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="89076-117">Il modello viene attivato da un aumento o una diminuzione della domanda e, rispettivamente, aggiunge o rimuove le risorse nel cloud.</span><span class="sxs-lookup"><span data-stu-id="89076-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="89076-118">Queste risorse offrono ridondanza, disponibilità rapida e routing conforme a geografica.</span><span class="sxs-lookup"><span data-stu-id="89076-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Modello di scalabilità tra cloud](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="89076-120">Questo modello si applica solo ai componenti senza stato dell'app.</span><span class="sxs-lookup"><span data-stu-id="89076-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="89076-121">Componenti</span><span class="sxs-lookup"><span data-stu-id="89076-121">Components</span></span>

<span data-ttu-id="89076-122">Il modello di scalabilità tra cloud è costituito dai componenti seguenti.</span><span class="sxs-lookup"><span data-stu-id="89076-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="89076-123">All'esterno del cloud</span><span class="sxs-lookup"><span data-stu-id="89076-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="89076-124">Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="89076-124">Traffic Manager</span></span>

<span data-ttu-id="89076-125">Nel diagramma, si trova all'esterno del gruppo di cloud pubblico, ma è necessario che sia in grado di coordinare il traffico sia nel Data Center locale che nel cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="89076-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="89076-126">Il servizio di bilanciamento garantisce la disponibilità elevata per l'app monitorando gli endpoint e fornendo la ridistribuzione del failover quando necessario.</span><span class="sxs-lookup"><span data-stu-id="89076-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="89076-127">Domain Name System (DNS)</span><span class="sxs-lookup"><span data-stu-id="89076-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="89076-128">Il nome DNS (Domain Name System) è responsabile della conversione (o risoluzione) del nome di un sito Web o del servizio nel relativo indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="89076-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="89076-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="89076-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="89076-130">Server di compilazione ospitato</span><span class="sxs-lookup"><span data-stu-id="89076-130">Hosted build server</span></span>

<span data-ttu-id="89076-131">Ambiente per l'hosting della pipeline di compilazione.</span><span class="sxs-lookup"><span data-stu-id="89076-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="89076-132">Risorse per le app</span><span class="sxs-lookup"><span data-stu-id="89076-132">App resources</span></span>

<span data-ttu-id="89076-133">Le risorse dell'app devono essere in grado di eseguire la scalabilità orizzontale e orizzontale, ad esempio i set di scalabilità di macchine virtuali e i contenitori.</span><span class="sxs-lookup"><span data-stu-id="89076-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="89076-134">Nome di dominio personalizzato</span><span class="sxs-lookup"><span data-stu-id="89076-134">Custom domain name</span></span>

<span data-ttu-id="89076-135">Usare un nome di dominio personalizzato per il routing delle richieste glob.</span><span class="sxs-lookup"><span data-stu-id="89076-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="89076-136">Indirizzi IP pubblici</span><span class="sxs-lookup"><span data-stu-id="89076-136">Public IP addresses</span></span>

<span data-ttu-id="89076-137">Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="89076-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="89076-138">Cloud locale</span><span class="sxs-lookup"><span data-stu-id="89076-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="89076-139">Server di compilazione ospitato</span><span class="sxs-lookup"><span data-stu-id="89076-139">Hosted build server</span></span>

<span data-ttu-id="89076-140">Ambiente per l'hosting della pipeline di compilazione.</span><span class="sxs-lookup"><span data-stu-id="89076-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="89076-141">Risorse per le app</span><span class="sxs-lookup"><span data-stu-id="89076-141">App resources</span></span>

<span data-ttu-id="89076-142">Le risorse dell'app richiedono la possibilità di scalare in orizzontale e in orizzontale, come i set di scalabilità di macchine virtuali e i contenitori.</span><span class="sxs-lookup"><span data-stu-id="89076-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="89076-143">Nome di dominio personalizzato</span><span class="sxs-lookup"><span data-stu-id="89076-143">Custom domain name</span></span>

<span data-ttu-id="89076-144">Usare un nome di dominio personalizzato per il routing delle richieste glob.</span><span class="sxs-lookup"><span data-stu-id="89076-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="89076-145">Indirizzi IP pubblici</span><span class="sxs-lookup"><span data-stu-id="89076-145">Public IP addresses</span></span>

<span data-ttu-id="89076-146">Gli indirizzi IP pubblici vengono usati per indirizzare il traffico in ingresso attraverso gestione traffico all'endpoint di risorse dell'app cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="89076-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="89076-147">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="89076-147">Issues and considerations</span></span>

<span data-ttu-id="89076-148">Prima di decidere come implementare questo modello, considerare quanto segue:</span><span class="sxs-lookup"><span data-stu-id="89076-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="89076-149">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="89076-149">Scalability</span></span>

<span data-ttu-id="89076-150">Il componente chiave del ridimensionamento tra cloud è la possibilità di fornire scalabilità su richiesta.</span><span class="sxs-lookup"><span data-stu-id="89076-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="89076-151">La scalabilità deve essere eseguita tra l'infrastruttura cloud pubblica e quella locale e fornire un servizio coerente e affidabile in base alla richiesta.</span><span class="sxs-lookup"><span data-stu-id="89076-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="89076-152">Disponibilità</span><span class="sxs-lookup"><span data-stu-id="89076-152">Availability</span></span>

<span data-ttu-id="89076-153">Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata tramite la configurazione hardware locale e la distribuzione software.</span><span class="sxs-lookup"><span data-stu-id="89076-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="89076-154">Gestione</span><span class="sxs-lookup"><span data-stu-id="89076-154">Manageability</span></span>

<span data-ttu-id="89076-155">Il modello tra cloud garantisce una gestione semplice e un'interfaccia familiare tra gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="89076-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="89076-156">Quando usare questo modello</span><span class="sxs-lookup"><span data-stu-id="89076-156">When to use this pattern</span></span>

<span data-ttu-id="89076-157">Usare questo schema:</span><span class="sxs-lookup"><span data-stu-id="89076-157">Use this pattern:</span></span>

- <span data-ttu-id="89076-158">Quando è necessario aumentare la capacità dell'app con richieste impreviste o richieste periodiche nella richiesta.</span><span class="sxs-lookup"><span data-stu-id="89076-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="89076-159">Quando non si vuole investire in risorse che verranno usate solo durante i picchi.</span><span class="sxs-lookup"><span data-stu-id="89076-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="89076-160">Pagare per le risorse usate.</span><span class="sxs-lookup"><span data-stu-id="89076-160">Pay for what you use.</span></span>

<span data-ttu-id="89076-161">Questo modello non è consigliato nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="89076-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="89076-162">Per la soluzione è necessario che gli utenti si connettano tramite Internet.</span><span class="sxs-lookup"><span data-stu-id="89076-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="89076-163">L'azienda dispone di normative locali che richiedono che la connessione di origine provenga da una chiamata al sito.</span><span class="sxs-lookup"><span data-stu-id="89076-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="89076-164">La rete presenta colli di bottiglia regolari che limitano le prestazioni del ridimensionamento.</span><span class="sxs-lookup"><span data-stu-id="89076-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="89076-165">L'ambiente è disconnesso da Internet e non può raggiungere il cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="89076-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="89076-166">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="89076-166">Next steps</span></span>

<span data-ttu-id="89076-167">Per ulteriori informazioni sugli argomenti introdotti in questo articolo:</span><span class="sxs-lookup"><span data-stu-id="89076-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="89076-168">Vedere [Panoramica di gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview) per altre informazioni sul funzionamento di questo servizio di bilanciamento del carico del traffico basato su DNS.</span><span class="sxs-lookup"><span data-stu-id="89076-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="89076-169">Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte per eventuali domande aggiuntive, vedere [considerazioni sulla progettazione di applicazioni ibride](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="89076-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="89076-170">Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="89076-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="89076-171">Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione di soluzioni di scalabilità tra cloud](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="89076-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="89076-172">Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.</span><span class="sxs-lookup"><span data-stu-id="89076-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="89076-173">Si apprenderà come creare una soluzione tra cloud per fornire un processo attivato manualmente per passare da un'app Web ospitata nell'hub Azure Stack a un'app Web ospitata in Azure.</span><span class="sxs-lookup"><span data-stu-id="89076-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="89076-174">Si apprenderà anche come usare la scalabilità automatica tramite Gestione traffico, garantendo un'utilità cloud flessibile e scalabile quando sono sotto carico.</span><span class="sxs-lookup"><span data-stu-id="89076-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
