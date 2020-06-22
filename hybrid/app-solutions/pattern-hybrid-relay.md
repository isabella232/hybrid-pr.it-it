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
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="f7f76-103">Modello di inoltro ibrido</span><span class="sxs-lookup"><span data-stu-id="f7f76-103">Hybrid relay pattern</span></span>

<span data-ttu-id="f7f76-104">Informazioni su come connettersi a risorse o dispositivi perimetrali protetti da firewall usando il modello di inoltro ibrido e il relè di Azure.</span><span class="sxs-lookup"><span data-stu-id="f7f76-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="f7f76-105">Contesto e problema</span><span class="sxs-lookup"><span data-stu-id="f7f76-105">Context and problem</span></span>

<span data-ttu-id="f7f76-106">I dispositivi perimetrali sono spesso protetti da un firewall aziendale o da un dispositivo NAT.</span><span class="sxs-lookup"><span data-stu-id="f7f76-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="f7f76-107">Sebbene siano sicure, potrebbero non essere in grado di comunicare con il cloud pubblico o i dispositivi perimetrali in altre reti aziendali.</span><span class="sxs-lookup"><span data-stu-id="f7f76-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="f7f76-108">Potrebbe essere necessario esporre determinate porte e funzionalità agli utenti nel cloud pubblico in modo sicuro.</span><span class="sxs-lookup"><span data-stu-id="f7f76-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="f7f76-109">Soluzione</span><span class="sxs-lookup"><span data-stu-id="f7f76-109">Solution</span></span>

<span data-ttu-id="f7f76-110">Il modello di inoltro ibrido usa il relè di Azure per stabilire un tunnel di WebSocket tra due endpoint che non possono comunicare direttamente.</span><span class="sxs-lookup"><span data-stu-id="f7f76-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="f7f76-111">I dispositivi che non sono locali ma che devono connettersi a un endpoint locale si connetteranno a un endpoint nel cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="f7f76-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="f7f76-112">Questo endpoint reindirizza il traffico su route predefinite su un canale sicuro.</span><span class="sxs-lookup"><span data-stu-id="f7f76-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="f7f76-113">Un endpoint all'interno dell'ambiente locale riceve il traffico e lo instrada alla destinazione corretta.</span><span class="sxs-lookup"><span data-stu-id="f7f76-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![architettura della soluzione modello di inoltro ibrido](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="f7f76-115">Ecco come funziona il modello di inoltro ibrido:</span><span class="sxs-lookup"><span data-stu-id="f7f76-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="f7f76-116">Un dispositivo si connette alla macchina virtuale (VM) in Azure, su una porta predefinita.</span><span class="sxs-lookup"><span data-stu-id="f7f76-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="f7f76-117">Il traffico viene inoltrato al relè di Azure in Azure.</span><span class="sxs-lookup"><span data-stu-id="f7f76-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="f7f76-118">La VM nell'hub Azure Stack, che ha già stabilito una connessione di lunga durata al relè di Azure, riceve il traffico e lo inoltra alla destinazione.</span><span class="sxs-lookup"><span data-stu-id="f7f76-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="f7f76-119">Il servizio o l'endpoint locale elabora la richiesta.</span><span class="sxs-lookup"><span data-stu-id="f7f76-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="f7f76-120">Componenti</span><span class="sxs-lookup"><span data-stu-id="f7f76-120">Components</span></span>

<span data-ttu-id="f7f76-121">Questa soluzione USA i componenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="f7f76-121">This solution uses the following components:</span></span>

| <span data-ttu-id="f7f76-122">Livello</span><span class="sxs-lookup"><span data-stu-id="f7f76-122">Layer</span></span> | <span data-ttu-id="f7f76-123">Componente</span><span class="sxs-lookup"><span data-stu-id="f7f76-123">Component</span></span> | <span data-ttu-id="f7f76-124">Descrizione</span><span class="sxs-lookup"><span data-stu-id="f7f76-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="f7f76-125">Azure</span><span class="sxs-lookup"><span data-stu-id="f7f76-125">Azure</span></span> | <span data-ttu-id="f7f76-126">Macchina virtuale di Azure</span><span class="sxs-lookup"><span data-stu-id="f7f76-126">Azure VM</span></span> | <span data-ttu-id="f7f76-127">Una macchina virtuale di Azure fornisce un endpoint accessibile pubblicamente per la risorsa locale.</span><span class="sxs-lookup"><span data-stu-id="f7f76-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="f7f76-128">Servizio di inoltro di Azure</span><span class="sxs-lookup"><span data-stu-id="f7f76-128">Azure Relay</span></span> | <span data-ttu-id="f7f76-129">Un [inoltro di Azure](/azure/azure-relay/) fornisce l'infrastruttura per la gestione del tunnel e della connessione tra la macchina virtuale di Azure e la macchina virtuale dell'hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="f7f76-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="f7f76-130">Hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="f7f76-130">Azure Stack Hub</span></span> | <span data-ttu-id="f7f76-131">Calcolo</span><span class="sxs-lookup"><span data-stu-id="f7f76-131">Compute</span></span> | <span data-ttu-id="f7f76-132">Una macchina virtuale Azure Stack Hub fornisce il lato server del tunnel di inoltro ibrido.</span><span class="sxs-lookup"><span data-stu-id="f7f76-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="f7f76-133">Archiviazione</span><span class="sxs-lookup"><span data-stu-id="f7f76-133">Storage</span></span> | <span data-ttu-id="f7f76-134">Il cluster del motore AKS distribuito in Azure Stack Hub fornisce un motore resiliente e scalabile per eseguire il API Viso contenitore.</span><span class="sxs-lookup"><span data-stu-id="f7f76-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="f7f76-135">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="f7f76-135">Issues and considerations</span></span>

<span data-ttu-id="f7f76-136">Quando si decide come implementare questa soluzione, tenere presente quanto segue:</span><span class="sxs-lookup"><span data-stu-id="f7f76-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="f7f76-137">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="f7f76-137">Scalability</span></span>

<span data-ttu-id="f7f76-138">Questo modello consente solo i mapping delle porte 1:1 nel client e nel server.</span><span class="sxs-lookup"><span data-stu-id="f7f76-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="f7f76-139">Se, ad esempio, la porta 80 viene sottoferita a tunneling per un servizio nell'endpoint di Azure, non può essere usata per un altro servizio.</span><span class="sxs-lookup"><span data-stu-id="f7f76-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="f7f76-140">I mapping delle porte devono essere pianificati di conseguenza.</span><span class="sxs-lookup"><span data-stu-id="f7f76-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="f7f76-141">Il relay e le macchine virtuali di Azure devono essere ridimensionati in modo appropriato per gestire il traffico.</span><span class="sxs-lookup"><span data-stu-id="f7f76-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="f7f76-142">Disponibilità</span><span class="sxs-lookup"><span data-stu-id="f7f76-142">Availability</span></span>

<span data-ttu-id="f7f76-143">Questi tunnel e connessioni non sono ridondanti.</span><span class="sxs-lookup"><span data-stu-id="f7f76-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="f7f76-144">Per garantire la disponibilità elevata, è opportuno implementare il codice di controllo degli errori.</span><span class="sxs-lookup"><span data-stu-id="f7f76-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="f7f76-145">Un'altra opzione consiste nel disporre di un pool di VM connesse al servizio di inoltro di Azure dietro un servizio di bilanciamento del carico.</span><span class="sxs-lookup"><span data-stu-id="f7f76-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="f7f76-146">Gestione</span><span class="sxs-lookup"><span data-stu-id="f7f76-146">Manageability</span></span>

<span data-ttu-id="f7f76-147">Questa soluzione può estendersi a molti dispositivi e posizioni, che potrebbero essere difficili da gestire.</span><span class="sxs-lookup"><span data-stu-id="f7f76-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="f7f76-148">I servizi Internet delle cose di Azure possono portare automaticamente nuovi percorsi e dispositivi e mantenerli aggiornati.</span><span class="sxs-lookup"><span data-stu-id="f7f76-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="f7f76-149">Sicurezza</span><span class="sxs-lookup"><span data-stu-id="f7f76-149">Security</span></span>

<span data-ttu-id="f7f76-150">Questo modello, come illustrato, consente l'accesso illimitato a una porta in un dispositivo interno dal perimetro.</span><span class="sxs-lookup"><span data-stu-id="f7f76-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="f7f76-151">Prendere in considerazione l'aggiunta di un meccanismo di autenticazione al servizio nel dispositivo interno o davanti all'endpoint di inoltro ibrido.</span><span class="sxs-lookup"><span data-stu-id="f7f76-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f7f76-152">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="f7f76-152">Next steps</span></span>

<span data-ttu-id="f7f76-153">Per ulteriori informazioni sugli argomenti introdotti in questo articolo:</span><span class="sxs-lookup"><span data-stu-id="f7f76-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="f7f76-154">Questo modello usa il relè di Azure.</span><span class="sxs-lookup"><span data-stu-id="f7f76-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="f7f76-155">Per altre informazioni, vedere la [documentazione di inoltro di Azure](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="f7f76-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="f7f76-156">Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte a eventuali domande aggiuntive, vedere [considerazioni sulla progettazione di applicazioni ibride](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="f7f76-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="f7f76-157">Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="f7f76-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="f7f76-158">Quando si è pronti per testare l'esempio di soluzione, continuare con la [Guida alla distribuzione di soluzioni di inoltro ibrido](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="f7f76-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="f7f76-159">Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.</span><span class="sxs-lookup"><span data-stu-id="f7f76-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>