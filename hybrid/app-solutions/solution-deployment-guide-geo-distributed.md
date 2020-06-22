---
title: Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub Azure Stack
description: Informazioni su come indirizzare il traffico a endpoint specifici con una soluzione di app con distribuzione geografica con Azure e hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911325"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="f4fcf-103">Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="f4fcf-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f4fcf-104">Informazioni su come indirizzare il traffico a endpoint specifici in base a diverse metriche usando il modello di app con distribuzione geografica.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="f4fcf-105">La creazione di un profilo di gestione traffico con routing basato su geografia e configurazione dell'endpoint garantisce che le informazioni vengano indirizzate agli endpoint in base ai requisiti internazionali, alla regolamentazione aziendale e internazionale e alle esigenze dei dati.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="f4fcf-106">In questa soluzione verrà compilato un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f4fcf-107">Creare un'app con distribuzione geografica.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="f4fcf-108">Usare gestione traffico per la destinazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="f4fcf-109">Usare il modello di app con distribuzione geografica</span><span class="sxs-lookup"><span data-stu-id="f4fcf-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="f4fcf-110">Con il modello con distribuzione geografica, l'app si estende su più aree.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="f4fcf-111">Per impostazione predefinita, è possibile utilizzare il cloud pubblico, ma alcuni utenti potrebbero richiedere che i loro dati rimangano nella propria area geografica.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="f4fcf-112">Puoi indirizzare gli utenti al cloud più adatto in base ai requisiti.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="f4fcf-113">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="f4fcf-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="f4fcf-114">Considerazioni sulla scalabilità</span><span class="sxs-lookup"><span data-stu-id="f4fcf-114">Scalability considerations</span></span>

<span data-ttu-id="f4fcf-115">La soluzione che verrà creata con questo articolo non è quella di adattare la scalabilità.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="f4fcf-116">Tuttavia, se usato in combinazione con altre soluzioni Azure e locali, è possibile soddisfare i requisiti di scalabilità.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="f4fcf-117">Per informazioni sulla creazione di una soluzione ibrida con la scalabilità automatica tramite Gestione traffico, vedere [creare soluzioni di scalabilità tra cloud con Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="f4fcf-118">Considerazioni sulla disponibilità</span><span class="sxs-lookup"><span data-stu-id="f4fcf-118">Availability considerations</span></span>

<span data-ttu-id="f4fcf-119">Come nel caso delle considerazioni sulla scalabilità, questa soluzione non risolve direttamente la disponibilità.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="f4fcf-120">Tuttavia, Azure e le soluzioni locali possono essere implementate all'interno di questa soluzione per garantire la disponibilità elevata per tutti i componenti interessati.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="f4fcf-121">Quando usare questo modello</span><span class="sxs-lookup"><span data-stu-id="f4fcf-121">When to use this pattern</span></span>

- <span data-ttu-id="f4fcf-122">L'organizzazione dispone di rami internazionali che richiedono criteri personalizzati per la sicurezza e la distribuzione a livello di area.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="f4fcf-123">Ogni ufficio dell'organizzazione effettua il pull dei dati relativi a dipendenti, aziende e funzionalità, che richiedono l'attività di creazione di report in base alle normative locali e ai fusi orari.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="f4fcf-124">I requisiti per la scalabilità elevata sono soddisfatti con la scalabilità orizzontale delle app con più distribuzioni di app all'interno di una singola area e tra aree per gestire i requisiti di carico estremi.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="f4fcf-125">Pianificazione della topologia</span><span class="sxs-lookup"><span data-stu-id="f4fcf-125">Planning the topology</span></span>

<span data-ttu-id="f4fcf-126">Prima di creare un footprint per le app distribuite, è utile sapere quanto segue:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="f4fcf-127">**Dominio personalizzato per l'app:** Qual è il nome di dominio personalizzato che i clienti useranno per accedere all'app?</span><span class="sxs-lookup"><span data-stu-id="f4fcf-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="f4fcf-128">Per l'app di esempio, il nome di dominio personalizzato è *www \. scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="f4fcf-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="f4fcf-129">**Dominio di Traffic Manager:** Quando si crea un [profilo di gestione traffico di Azure](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles), viene scelto un nome di dominio.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="f4fcf-130">Questo nome viene combinato con il suffisso *trafficmanager.NET* per registrare una voce di dominio gestita da Gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="f4fcf-131">Per l'app di esempio il nome scelto è *scalable-ase-demo*.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="f4fcf-132">Di conseguenza, il nome di dominio completo gestito da Traffic Manager è *Scalable-ASE-demo.trafficmanager.NET*.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="f4fcf-133">**Strategia per la scalabilità del footprint dell'app:** Decidere se il footprint dell'app verrà distribuito tra più ambienti del servizio app in una singola area, più aree o una combinazione di entrambi gli approcci.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="f4fcf-134">La decisione deve essere basata sulle aspettative relative al modo in cui il traffico dei clienti produrrà e quanto il resto dell'infrastruttura di back-end di supporto di un'app possa essere ridimensionato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="f4fcf-135">Con un'app senza stato del 100%, ad esempio, un'app può essere ridimensionata in modo massiccio usando una combinazione di più ambienti del servizio app per ogni area di Azure, moltiplicata per gli ambienti del servizio app distribuiti in più aree di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="f4fcf-136">Con più di 15 aree globali di Azure disponibili per la scelta, i clienti possono effettivamente creare un footprint di app su vasta scala mondiale.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="f4fcf-137">Per l'app di esempio usata in questo articolo, sono stati creati tre ambienti del servizio app in una singola area di Azure (Stati Uniti centro-meridionali).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="f4fcf-138">**Convenzione di denominazione per gli ambienti del servizio app:** Ogni ambiente del servizio app richiede un nome univoco.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="f4fcf-139">Oltre uno o due ambienti del servizio app, è utile avere una convenzione di denominazione che consenta di identificare ogni ambiente del servizio app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="f4fcf-140">Per l'app di esempio usata in questo argomento, è stata usata una convenzione di denominazione semplice.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="f4fcf-141">I nomi dei tre ambienti del servizio app sono *fe1ase*, *fe2ase*e *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="f4fcf-142">**Convenzione di denominazione per le app:** Poiché verranno distribuite più istanze dell'app, è necessario un nome per ogni istanza dell'app distribuita.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="f4fcf-143">Con ambiente del servizio app per Power Apps, lo stesso nome di app può essere usato in più ambienti.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="f4fcf-144">Poiché ogni ambiente del servizio app ha un suffisso di dominio univoco, gli sviluppatori possono scegliere di riutilizzare esattamente lo stesso nome di app in ogni ambiente.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="f4fcf-145">Ad esempio, uno sviluppatore potrebbe avere app denominate come segue: *MyApp.foo1.p.azurewebsites.NET*, *MyApp.foo2.p.azurewebsites.NET*, *MyApp.foo3.p.azurewebsites.NET*e così via.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="f4fcf-146">Per l'app usata qui, ogni istanza dell'app ha un nome univoco.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="f4fcf-147">I nomi delle istanze dell'app usati sono *webfrontend1*, *webfrontend2* e *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="f4fcf-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="f4fcf-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="f4fcf-149">Microsoft Azure Stack Hub è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="f4fcf-150">Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="f4fcf-151">L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="f4fcf-152">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="f4fcf-153">Parte 1: creare un'app con distribuzione geografica</span><span class="sxs-lookup"><span data-stu-id="f4fcf-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="f4fcf-154">In questa parte verrà creata un'app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f4fcf-155">Creazione di app Web e pubblicazione.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="f4fcf-156">Aggiungere il codice Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="f4fcf-157">Puntare la compilazione dell'app a più destinazioni cloud.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="f4fcf-158">Gestire e configurare il processo CD.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f4fcf-159">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="f4fcf-159">Prerequisites</span></span>

<span data-ttu-id="f4fcf-160">Sono necessarie una sottoscrizione di Azure e l'installazione dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="f4fcf-161">Passaggi dell'app con distribuzione geografica</span><span class="sxs-lookup"><span data-stu-id="f4fcf-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="f4fcf-162">Ottenere un dominio personalizzato e configurare DNS</span><span class="sxs-lookup"><span data-stu-id="f4fcf-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="f4fcf-163">Aggiornare il file di zona DNS per il dominio.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="f4fcf-164">Azure AD possibile verificare la proprietà del nome di dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="f4fcf-165">Usare [DNS di Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in [un registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-165">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="f4fcf-166">Registrare un dominio personalizzato con un registrar pubblico.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="f4fcf-167">Accedere al registrar per il dominio.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="f4fcf-168">Potrebbe essere necessario un amministratore approvato per eseguire gli aggiornamenti DNS.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="f4fcf-169">Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS fornita da Azure AD.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="f4fcf-170">La voce DNS non modifica i comportamenti, ad esempio il routing della posta elettronica o l'hosting Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="f4fcf-171">Creare app Web e pubblicare</span><span class="sxs-lookup"><span data-stu-id="f4fcf-171">Create web apps and publish</span></span>

<span data-ttu-id="f4fcf-172">Configurare l'integrazione continua e la distribuzione continua (CI/CD) ibride per distribuire l'app Web in Azure e l'hub Azure Stack e per eseguire il push automatico delle modifiche in entrambi i cloud.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="f4fcf-173">È necessario Azure Stack Hub con le immagini appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="f4fcf-174">Per altre informazioni, vedere [prerequisiti per la distribuzione del servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="f4fcf-175">Aggiungi codice a Azure Repos</span><span class="sxs-lookup"><span data-stu-id="f4fcf-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="f4fcf-176">Accedere a Visual Studio con un **account che disponga dei diritti di creazione del progetto** in Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="f4fcf-177">CI/CD può essere applicato sia al codice dell'app che al codice dell'infrastruttura.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="f4fcf-178">USA [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) per lo sviluppo cloud privato e ospitato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Connettersi a un progetto in Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="f4fcf-180">**Clonare il repository** creando e aprendo l'app Web predefinita.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonare il repository in Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="f4fcf-182">Creare una distribuzione di app Web in entrambi i cloud</span><span class="sxs-lookup"><span data-stu-id="f4fcf-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="f4fcf-183">Modificare il file **WebApplication. csproj** : selezionare `Runtimeidentifier` e aggiungere `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="f4fcf-184">Vedere la documentazione sulla [distribuzione indipendente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-184">(See [Self-contained Deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Modificare il file di progetto dell'app Web in Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="f4fcf-186">**Archiviare il codice per Azure Repos** usando Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="f4fcf-187">Verificare che il **codice dell'applicazione** sia stato archiviato Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="f4fcf-188">Creare la definizione di compilazione</span><span class="sxs-lookup"><span data-stu-id="f4fcf-188">Create the build definition</span></span>

1. <span data-ttu-id="f4fcf-189">**Accedere a Azure Pipelines** per verificare la possibilità di creare definizioni di compilazione.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="f4fcf-190">Aggiungere il `-r win10-x64` codice.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="f4fcf-191">Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Aggiungere codice alla definizione di compilazione in Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="f4fcf-193">**Eseguire la compilazione**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-193">**Run the build**.</span></span> <span data-ttu-id="f4fcf-194">Il processo di [compilazione della distribuzione autonoma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e in hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-194">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="f4fcf-195">Uso di un agente ospitato di Azure</span><span class="sxs-lookup"><span data-stu-id="f4fcf-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="f4fcf-196">L'uso di un agente ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="f4fcf-197">La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, che consente lo sviluppo, il test e la distribuzione senza interruzioni.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="f4fcf-198">Gestire e configurare il processo CD</span><span class="sxs-lookup"><span data-stu-id="f4fcf-198">Manage and configure the CD process</span></span>

<span data-ttu-id="f4fcf-199">Azure DevOps Services forniscono una pipeline altamente configurabile e gestibile per le versioni in più ambienti, ad esempio sviluppo, gestione temporanea, controllo di qualità e ambienti di produzione; inclusa la richiesta di approvazioni in fasi specifiche.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="f4fcf-200">Creare una definizione della versione</span><span class="sxs-lookup"><span data-stu-id="f4fcf-200">Create release definition</span></span>

1. <span data-ttu-id="f4fcf-201">Selezionare il pulsante **più** per aggiungere una nuova versione nella scheda **versioni** della sezione **compilazione e versione** di Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Creare una definizione di versione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="f4fcf-203">Applicare il modello di distribuzione del servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-203">Apply the Azure App Service Deployment template.</span></span>

   ![Applicare il modello di distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="f4fcf-205">In **Aggiungi artefatto**aggiungere l'artefatto per l'app Azure cloud Build.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Aggiungere un elemento alla build del cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="f4fcf-207">In scheda pipeline selezionare la **fase,** il collegamento all'attività dell'ambiente e impostare i valori dell'ambiente cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Impostare i valori dell'ambiente cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="f4fcf-209">Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="f4fcf-211">In **nome servizio app**impostare il nome del servizio app di Azure richiesto.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Impostare il nome del servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="f4fcf-213">Immettere "Hosted VS2017" nella **coda dell'agente per l'** ambiente ospitato nel cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Impostare la coda agente per l'ambiente ospitato nel cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="f4fcf-215">Nel menu Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="f4fcf-216">Selezionare **OK** per **percorso cartella**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-216">Select **OK** to **folder location**.</span></span>
  
      ![Selezionare il pacchetto o la cartella per app Azure ambiente del servizio in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selezionare il pacchetto o la cartella per app Azure ambiente del servizio in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="f4fcf-219">Salvare tutte le modifiche e tornare alla **pipeline di rilascio**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Salva le modifiche nella pipeline di rilascio in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="f4fcf-221">Aggiungere un nuovo elemento selezionando la compilazione per l'App Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Aggiungere un nuovo artefatto per Azure Stack App Hub in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="f4fcf-223">Aggiungere un altro ambiente applicando la distribuzione del servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Aggiungere ambiente alla distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="f4fcf-225">Assegnare un nome al nuovo ambiente Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-225">Name the new environment Azure Stack Hub.</span></span>

    ![Ambiente del nome nella distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="f4fcf-227">Trovare l'ambiente dell'hub Azure Stack nella scheda **attività** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Ambiente Azure Stack Hub in Azure DevOps Services Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="f4fcf-229">Selezionare la sottoscrizione per l'endpoint dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Selezionare la sottoscrizione per l'endpoint dell'hub Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="f4fcf-231">Impostare il nome dell'app Web dell'hub Azure Stack come nome del servizio app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Impostare il nome dell'app Web dell'hub Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="f4fcf-233">Selezionare l'agente dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-233">Select the Azure Stack Hub agent.</span></span>

    ![Selezionare l'agente Hub Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="f4fcf-235">Nella sezione Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="f4fcf-236">Selezionare **OK** per percorso cartella.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-236">Select **OK** to folder location.</span></span>

    ![Selezionare la cartella per la distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selezionare la cartella per la distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="f4fcf-239">Nella scheda variabile aggiungere una variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , impostarne il valore su **true**e l'ambito su Azure stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Aggiungere una variabile alla distribuzione app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="f4fcf-241">Selezionare l'icona del trigger di distribuzione **continua** in entrambi gli artefatti e abilitare il trigger di distribuzione **continua** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selezionare il trigger di distribuzione continua in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="f4fcf-243">Selezionare l'icona condizioni **pre-distribuzione** nell'ambiente dell'hub Azure stack e impostare il trigger su **dopo la versione.**</span><span class="sxs-lookup"><span data-stu-id="f4fcf-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Selezionare le condizioni di pre-distribuzione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="f4fcf-245">Salvare tutte le modifiche.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="f4fcf-246">Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) quando si crea una definizione di versione da un modello.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-246">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="f4fcf-247">Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. per modificare queste impostazioni è necessario invece selezionare l'elemento dell'ambiente padre.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="f4fcf-248">Parte 2: aggiornare le opzioni dell'app Web</span><span class="sxs-lookup"><span data-stu-id="f4fcf-248">Part 2: Update web app options</span></span>

<span data-ttu-id="f4fcf-249">[Servizio app di Azure](https://docs.microsoft.com/azure/app-service/overview) offre un servizio di hosting Web con scalabilità elevata e funzioni di auto-correzione.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-249">[Azure App Service](https://docs.microsoft.com/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Servizio app di Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="f4fcf-251">Eseguire il mapping di un nome DNS personalizzato esistente ad app Web di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="f4fcf-252">Usare un **record CNAME** e un **record** a per eseguire il mapping di un nome DNS personalizzato al servizio app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="f4fcf-253">Esecuzione del mapping di un nome DNS personalizzato esistente con un app Web di Azure</span><span class="sxs-lookup"><span data-stu-id="f4fcf-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="f4fcf-254">Usare un record CNAME per tutti i nomi DNS personalizzati ad eccezione di un dominio radice (ad esempio, northwind.com).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="f4fcf-255">Per eseguire la migrazione di un sito in tempo reale e del relativo nome di dominio DNS al servizio app, vedere [Migrare un nome DNS attivo nel servizio app di Azure](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f4fcf-256">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="f4fcf-256">Prerequisites</span></span>

<span data-ttu-id="f4fcf-257">Per completare questa soluzione:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-257">To complete this solution:</span></span>

- <span data-ttu-id="f4fcf-258">[Creare un'app del servizio app](https://docs.microsoft.com/azure/app-service/)o usare un'app creata per un'altra soluzione.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-258">[Create an App Service app](https://docs.microsoft.com/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="f4fcf-259">Acquistare un nome di dominio e verificare l'accesso al registro DNS per il provider di dominio.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="f4fcf-260">Aggiornare il file di zona DNS per il dominio.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="f4fcf-261">Azure AD verificherà la proprietà del nome di dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="f4fcf-262">Usare [DNS di Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in [un registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-262">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

- <span data-ttu-id="f4fcf-263">Registrare un dominio personalizzato con un registrar pubblico.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="f4fcf-264">Accedere al registrar per il dominio.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="f4fcf-265">Per eseguire gli aggiornamenti DNS, potrebbe essere necessario un amministratore approvato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="f4fcf-266">Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS fornita da Azure AD.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="f4fcf-267">Ad esempio, per aggiungere le voci DNS per northwindcloud.com e www \. northwindcloud.com, configurare le impostazioni DNS per il dominio radice northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="f4fcf-268">È possibile acquistare un nome di dominio usando il [portale di Azure](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-268">A domain name may be purchased using the [Azure portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="f4fcf-269">Per eseguire il mapping di un nome DNS personalizzato a un'app Web, è necessario che il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) dell'app Web sia un livello a pagamento (**Condiviso**, **Base**, **Standard** o **Premium**).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="f4fcf-270">Creare ed eseguire il mapping di record CNAME e A</span><span class="sxs-lookup"><span data-stu-id="f4fcf-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="f4fcf-271">Accedere ai record DNS con il provider di dominio</span><span class="sxs-lookup"><span data-stu-id="f4fcf-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="f4fcf-272">Usare DNS di Azure per configurare un nome DNS personalizzato per le app Web di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="f4fcf-273">Per altre informazioni, vedere [Usare il servizio DNS di Azure per specificare impostazioni di dominio personalizzate per un servizio di Azure](https://docs.microsoft.com/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](https://docs.microsoft.com/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="f4fcf-274">Accedere al sito Web del provider principale.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="f4fcf-275">Individuare la pagina relativa alla gestione dei record DNS.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="f4fcf-276">Ogni provider di dominio ha una propria interfaccia di record DNS.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="f4fcf-277">Cercare le aree del sito denominate **Domain Name** (Nome di dominio), **DNS** o **Name Server Management** (Gestione server dei nomi).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="f4fcf-278">È possibile visualizzare la pagina record DNS nei **domini personali**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="f4fcf-279">Trovare il collegamento nome **file di zona**, **record DNS**o **configurazione avanzata**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="f4fcf-280">La schermata seguente è un esempio di pagina di record DNS:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-280">The following screenshot is an example of a DNS records page:</span></span>

![Pagina record DNS di esempio](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="f4fcf-282">In registrar, selezionare **Aggiungi o crea** per creare un record.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="f4fcf-283">Per alcuni provider esistono collegamenti diversi per aggiungere tipi di record diversi.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="f4fcf-284">Consultare la documentazione del provider.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="f4fcf-285">Aggiungere un record CNAME per eseguire il mapping di un sottodominio al nome host predefinito dell'app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="f4fcf-286">Per l' \. esempio di dominio www northwindcloud.com aggiungere un record CNAME che esegue il mapping del nome a `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="f4fcf-287">Dopo aver aggiunto il record CNAME, la pagina dei record DNS è simile all'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Passaggio all'app di Azure nel portale](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="f4fcf-289">Abilitare il mapping dei record CNAME in Azure</span><span class="sxs-lookup"><span data-stu-id="f4fcf-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="f4fcf-290">In una nuova scheda accedere al portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="f4fcf-291">Passare a Servizi app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-291">Go to App Services.</span></span>

3. <span data-ttu-id="f4fcf-292">Selezionare app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-292">Select web app.</span></span>

4. <span data-ttu-id="f4fcf-293">Nel riquadro di spostamento a sinistra della pagina dell'app nel portale di Azure selezionare **Domini personalizzati**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="f4fcf-294">Selezionare l' **+** icona accanto a **Aggiungi nome host**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="f4fcf-295">Digitare il nome di dominio completo, ad esempio `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="f4fcf-296">Selezionare **Convalida**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-296">Select **Validate**.</span></span>

8. <span data-ttu-id="f4fcf-297">Se indicato, aggiungere altri record di altri tipi ( `A` o `TXT` ) ai record DNS del registrar.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="f4fcf-298">Azure fornirà i valori e i tipi dei record seguenti:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="f4fcf-299">a.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-299">a.</span></span>  <span data-ttu-id="f4fcf-300">Un record **A** di cui eseguire il mapping all'indirizzo IP dell'app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="f4fcf-301">b.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-301">b.</span></span>  <span data-ttu-id="f4fcf-302">Un record **TXT** di cui eseguire il mapping al nome host predefinito dell'app `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="f4fcf-303">Il servizio app usa questo record solo in fase di configurazione per verificare la proprietà del dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="f4fcf-304">Dopo la verifica, eliminare il record TXT.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="f4fcf-305">Completare questa attività nella scheda registrar e rivalidare fino a quando non viene attivato il pulsante **Aggiungi nome host** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="f4fcf-306">Verificare che il **tipo di record del nome host** sia impostato su **CNAME** (www.example.com o qualsiasi sottodominio).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="f4fcf-307">Selezionare **Aggiungi il nome host**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="f4fcf-308">Digitare il nome di dominio completo, ad esempio `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="f4fcf-309">Selezionare **Convalida**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-309">Select **Validate**.</span></span> <span data-ttu-id="f4fcf-310">L' **aggiunta** è attivata.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="f4fcf-311">Verificare che il **tipo di record hostname** sia impostato su **un record** (example.com).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="f4fcf-312">**Aggiungere il nome host**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-312">**Add hostname**.</span></span>

    <span data-ttu-id="f4fcf-313">La reflection dei nuovi nomi host nella pagina **domini personalizzati** dell'app potrebbe richiedere del tempo.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="f4fcf-314">Provare ad aggiornare il browser per visualizzare i dati più recenti.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-314">Try refreshing the browser to update the data.</span></span>
  
    ![Domini personalizzati](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="f4fcf-316">Se si verifica un errore, nella parte inferiore della pagina verrà visualizzata una notifica di errore di verifica.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Errore di verifica del dominio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="f4fcf-318">I passaggi precedenti possono essere ripetuti per eseguire il mapping di un dominio con caratteri jolly ( \* northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="f4fcf-319">In questo modo è possibile aggiungere altri sottodomini a questo servizio app senza dover creare un record CNAME separato per ciascuno di essi.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="f4fcf-320">Per configurare questa impostazione, seguire le istruzioni del registrar.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="f4fcf-321">Esegui test in un browser</span><span class="sxs-lookup"><span data-stu-id="f4fcf-321">Test in a browser</span></span>

<span data-ttu-id="f4fcf-322">Individuare i nomi DNS configurati in precedenza (ad esempio, `northwindcloud.com` o `www.northwindcloud.com` ).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="f4fcf-323">Parte 3: associare un certificato SSL personalizzato</span><span class="sxs-lookup"><span data-stu-id="f4fcf-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="f4fcf-324">In questa parte verrà:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f4fcf-325">Associare il certificato SSL personalizzato al servizio app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="f4fcf-326">Applicare HTTPS per l'app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="f4fcf-327">Automatizzare l'associazione di certificati SSL con gli script.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="f4fcf-328">Se necessario, ottenere un certificato SSL del cliente nel portale di Azure e associarlo all'app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="f4fcf-329">Per altre informazioni, vedere l' [esercitazione sui certificati del servizio app](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-329">For more information, see the [App Service Certificates tutorial](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f4fcf-330">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="f4fcf-330">Prerequisites</span></span>

<span data-ttu-id="f4fcf-331">Per completare questa soluzione:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-331">To complete this  solution:</span></span>

- [<span data-ttu-id="f4fcf-332">Creare un'app del servizio app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-332">Create an App Service app.</span></span>](https://docs.microsoft.com/azure/app-service/)
- [<span data-ttu-id="f4fcf-333">Eseguire il mapping di un nome DNS personalizzato all'app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-333">Map a custom DNS name to your web app.</span></span>](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="f4fcf-334">Acquisire un certificato SSL da un'autorità di certificazione attendibile e usare la chiave per firmare la richiesta.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="f4fcf-335">Requisiti per il certificato SSL</span><span class="sxs-lookup"><span data-stu-id="f4fcf-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="f4fcf-336">Per poter essere usato nel servizio app, il certificato deve soddisfare tutti i requisiti seguenti:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="f4fcf-337">Firmato da un'autorità di certificazione attendibile.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="f4fcf-338">Esportato come file PFX protetto da password.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="f4fcf-339">Contiene una chiave privata con una lunghezza minima di 2048 bit.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="f4fcf-340">Contiene tutti i certificati intermedi nella catena di certificati.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="f4fcf-341">I **certificati di crittografia a curva ellittica (ecc)** funzionano con il servizio app, ma non sono inclusi in questa guida.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="f4fcf-342">Per assistenza nella creazione di certificati ECC, consultare un'autorità di certificazione.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="f4fcf-343">Preparare l'app Web</span><span class="sxs-lookup"><span data-stu-id="f4fcf-343">Prepare the web app</span></span>

<span data-ttu-id="f4fcf-344">Per associare un certificato SSL personalizzato all'app Web, il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) deve essere nel livello **Basic**, **standard**o **Premium** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="f4fcf-345">Accedere ad Azure</span><span class="sxs-lookup"><span data-stu-id="f4fcf-345">Sign in to Azure</span></span>

1. <span data-ttu-id="f4fcf-346">Aprire il [portale di Azure](https://portal.azure.com/) e passare all'app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="f4fcf-347">Nel menu a sinistra selezionare **Servizi app**e quindi selezionare il nome dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Selezionare l'app Web in portale di Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="f4fcf-349">Scegliere il piano tariffario</span><span class="sxs-lookup"><span data-stu-id="f4fcf-349">Check the pricing tier</span></span>

1. <span data-ttu-id="f4fcf-350">Nella finestra di spostamento a sinistra della pagina dell'app Web scorrere fino alla sezione **Impostazioni** e selezionare **scalabilità verticale (piano di servizio app)**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menu di scalabilità verticale nell'app Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="f4fcf-352">Assicurarsi che l'app Web non sia **inclusa** nel livello gratuito o **condiviso** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="f4fcf-353">Il livello corrente dell'app Web è evidenziato in una casella blu scuro.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Controllare il piano tariffario nell'app Web](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="f4fcf-355">SSL personalizzato non è supportato nel livello **gratuito** o **condiviso** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="f4fcf-356">Per eseguire la scalabilità, seguire i passaggi nella sezione successiva o nella pagina scegliere il piano **tariffario** e passare a [caricare e associare il certificato SSL](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="f4fcf-357">Passare a un piano di servizio app superiore</span><span class="sxs-lookup"><span data-stu-id="f4fcf-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="f4fcf-358">Selezionare uno tra i livelli **Basic**, **Standard** o **Premium**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="f4fcf-359">Scegliere **Seleziona**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-359">Select **Select**.</span></span>

![Scegliere il piano tariffario per l'app Web](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="f4fcf-361">L'operazione di ridimensionamento viene completata quando viene visualizzata la notifica.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-361">The scale operation is complete when notification is displayed.</span></span>

![Notifica di passaggio al livello superiore](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="f4fcf-363">Associare il certificato SSL e unire i certificati intermedi</span><span class="sxs-lookup"><span data-stu-id="f4fcf-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="f4fcf-364">Unire più certificati nella catena.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="f4fcf-365">**Aprire ogni certificato** ricevuto in un editor di testo.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="f4fcf-366">Creare un file per il certificato Unito denominato *mergedcertificate. CRT*.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="f4fcf-367">In un editor di testo copiare il contenuto di ogni certificato nel file.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="f4fcf-368">L'ordine dei certificati deve corrispondere all'ordine nella catena di certificati, che inizia con il certificato dell'utente e termina con il certificato radice.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="f4fcf-369">Sarà simile a quanto indicato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="f4fcf-370">Esportare il certificato in un file PFX</span><span class="sxs-lookup"><span data-stu-id="f4fcf-370">Export certificate to PFX</span></span>

<span data-ttu-id="f4fcf-371">Esportare il certificato SSL Unito con la chiave privata generata dal certificato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="f4fcf-372">Un file di chiave privata viene creato tramite OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="f4fcf-373">Per esportare il certificato in PFX, eseguire il comando seguente e sostituire i segnaposto `<private-key-file>` e `<merged-certificate-file>` con il percorso della chiave privata e il file di certificato Unito:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="f4fcf-374">Quando richiesto, definire una password di esportazione per il caricamento del certificato SSL nel servizio app in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="f4fcf-375">Quando si usano IIS o **Certreq.exe** per generare la richiesta di certificato, installare il certificato in un computer locale e quindi [esportare il certificato in pfx](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="f4fcf-376">Caricare il certificato SSL</span><span class="sxs-lookup"><span data-stu-id="f4fcf-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="f4fcf-377">Selezionare **Impostazioni SSL** nel percorso di spostamento sinistro dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="f4fcf-378">Selezionare **Carica certificato**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="f4fcf-379">In **file di certificato PFX**selezionare file PFX.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="f4fcf-380">In **password certificato**Digitare la password creata durante l'esportazione del file PFX.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="f4fcf-381">Selezionare **Carica**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-381">Select **Upload**.</span></span>

    ![Caricare il certificato SSL](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="f4fcf-383">Al termine del caricamento del certificato, il servizio app viene visualizzato nella pagina **Impostazioni SSL** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Impostazioni SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="f4fcf-385">Associare il certificato SSL</span><span class="sxs-lookup"><span data-stu-id="f4fcf-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="f4fcf-386">Nella sezione **binding SSL** selezionare **Aggiungi binding**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="f4fcf-387">Se il certificato è stato caricato, ma non viene visualizzato nei nomi di dominio nell'elenco a discesa nome **host** , provare ad aggiornare la pagina del browser.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="f4fcf-388">Nella pagina **Aggiungi binding SSL** usare gli elenchi a discesa per selezionare il nome di dominio da proteggere e il certificato da usare.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="f4fcf-389">In **Tipo SSL** selezionare se usare l'SSL basato su [**indicazione nome server (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) o basato su IP.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="f4fcf-390">**SSL basato su SNI**: è possibile aggiungere più associazioni SSL basate su SNI.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="f4fcf-391">Questa opzione consente di usare più certificati SSL per proteggere più domini nello stesso indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="f4fcf-392">La maggior parte dei browser moderni (tra cui Internet Explorer, Chrome, Firefox e Opera) supporta SNI. Per altre informazioni sul supporto dei browser, vedere [Indicazione nome server](https://wikipedia.org/wiki/Server_Name_Indication).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="f4fcf-393">**SSL basato su IP**: è possibile aggiungere una sola associazione SSL basata su IP.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="f4fcf-394">Questa opzione consente di usare solo un certificato SSL per proteggere un indirizzo IP pubblico dedicato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="f4fcf-395">Per proteggere più domini, è necessario proteggerli tutti utilizzando lo stesso certificato SSL.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="f4fcf-396">Il protocollo SSL basato su IP è l'opzione tradizionale per l'associazione SSL.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="f4fcf-397">Selezionare **Aggiungi binding**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-397">Select **Add Binding**.</span></span>

    ![Aggiungere il binding SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="f4fcf-399">Al termine del caricamento del certificato, il servizio app viene visualizzato nelle sezioni **associazioni SSL** .</span><span class="sxs-lookup"><span data-stu-id="f4fcf-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Il caricamento delle associazioni SSL è stato completato](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="f4fcf-401">Modificare il mapping del record A per IP SSL</span><span class="sxs-lookup"><span data-stu-id="f4fcf-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="f4fcf-402">Se l'SSL basato su IP non viene usato nell'app Web, passare a [testare HTTPS per il dominio personalizzato](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="f4fcf-403">Per impostazione predefinita, l'app Web usa un indirizzo IP pubblico condiviso.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="f4fcf-404">Quando il certificato è associato a SSL basato su IP, il servizio app crea un nuovo indirizzo IP dedicato per l'app Web.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="f4fcf-405">Quando viene eseguito il mapping di un record a all'app Web, è necessario aggiornare il registro di sistema con l'indirizzo IP dedicato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="f4fcf-406">La pagina **dominio personalizzato** viene aggiornata con il nuovo indirizzo IP dedicato.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="f4fcf-407">Copiare questo [indirizzo IP](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain), quindi modificare il mapping del [record](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) a in questo nuovo indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-407">Copy this [IP address](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="f4fcf-408">Testare HTTPS</span><span class="sxs-lookup"><span data-stu-id="f4fcf-408">Test HTTPS</span></span>

<span data-ttu-id="f4fcf-409">In diversi browser passare a `https://<your.custom.domain>` per assicurarsi che l'app Web venga servita.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![passare ad app Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="f4fcf-411">Se si verificano errori di convalida del certificato, è possibile che venga generato un certificato autofirmato oppure che i certificati intermedi siano stati lasciati durante l'esportazione nel file PFX.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="f4fcf-412">Imporre HTTPS</span><span class="sxs-lookup"><span data-stu-id="f4fcf-412">Enforce HTTPS</span></span>

<span data-ttu-id="f4fcf-413">Per impostazione predefinita, chiunque può accedere all'app Web tramite HTTP.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="f4fcf-414">È possibile reindirizzare tutte le richieste HTTP alla porta HTTPS.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="f4fcf-415">Nella pagina app Web selezionare **Impostazioni SL**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="f4fcf-416">Quindi in **Solo HTTPS**, selezionare **Attiva**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Applicare HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="f4fcf-418">Al termine dell'operazione, passare a uno degli URL HTTP che puntano all'app.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="f4fcf-419">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-419">For example:</span></span>

- <span data-ttu-id="f4fcf-420">https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="f4fcf-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="f4fcf-421">Applicare TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="f4fcf-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="f4fcf-422">Per impostazione predefinita, l'app consente [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0, che non è più considerato sicuro dagli standard di settore, ad esempio [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="f4fcf-423">Per applicare versioni di TLS più recenti, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="f4fcf-424">Nel spostamento a sinistra della pagina dell'app Web selezionare **Impostazioni SSL**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="f4fcf-425">In **versione TLS**selezionare la versione minima di TLS.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Applicare TLS 1.1 o 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="f4fcf-427">Creare un profilo di Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="f4fcf-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="f4fcf-428">Selezionare **Crea una risorsa**  >  **rete**  >  **profilo di gestione traffico**  >  **Crea**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="f4fcf-429">In **Crea profilo di Gestione traffico** procedere come segue:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="f4fcf-430">In **nome**specificare un nome per il profilo.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="f4fcf-431">Questo nome deve essere univoco all'interno della zona manager.net del traffico e restituisce il nome DNS, trafficmanager.net, usato per accedere al profilo di gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="f4fcf-432">In **metodo di routing**selezionare il **metodo di routing geografico**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="f4fcf-433">In **sottoscrizione**selezionare la sottoscrizione in cui creare il profilo.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="f4fcf-434">In **Gruppo di risorse** creare un nuovo gruppo di risorse in cui aggiungere il profilo.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="f4fcf-435">In **Località del gruppo di risorse** selezionare la località del gruppo di risorse.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="f4fcf-436">Questa impostazione si riferisce al percorso del gruppo di risorse e non ha alcun effetto sul profilo di gestione traffico distribuito a livello globale.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="f4fcf-437">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-437">Select **Create**.</span></span>

    7. <span data-ttu-id="f4fcf-438">Una volta completata la distribuzione globale del profilo di gestione traffico, questa viene elencata nel rispettivo gruppo di risorse come una delle risorse.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Gruppi di risorse in Crea profilo di gestione traffico](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="f4fcf-440">Aggiungere endpoint di Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="f4fcf-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="f4fcf-441">Nella barra di ricerca del portale cercare il nome del **profilo di gestione traffico** creato nella sezione precedente e selezionare il profilo di gestione traffico nei risultati visualizzati.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="f4fcf-442">Nella sezione **Impostazioni** del **profilo di gestione traffico**selezionare **endpoint**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="f4fcf-443">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-443">Select **Add**.</span></span>

4. <span data-ttu-id="f4fcf-444">Aggiunta dell'endpoint dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="f4fcf-445">Per **tipo**selezionare **endpoint esterno**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="f4fcf-446">Specificare un **nome** per l'endpoint, idealmente il nome dell'hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="f4fcf-447">Per nome di dominio completo (**FQDN**) usare l'URL esterno per l'app Web Hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="f4fcf-448">In mapping geografico selezionare un'area/continente in cui si trova la risorsa.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="f4fcf-449">Ad esempio, **Europa.**</span><span class="sxs-lookup"><span data-stu-id="f4fcf-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="f4fcf-450">Nell'elenco a discesa paese/area geografica visualizzato selezionare il paese che si applica a questo endpoint.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="f4fcf-451">Ad esempio, **Germania**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="f4fcf-452">Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="f4fcf-453">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-453">Select **OK**.</span></span>

12. <span data-ttu-id="f4fcf-454">Aggiunta dell'endpoint di Azure:</span><span class="sxs-lookup"><span data-stu-id="f4fcf-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="f4fcf-455">Per **tipo**selezionare **endpoint di Azure**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="f4fcf-456">Consente di specificare un **nome** per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="f4fcf-457">Per **tipo di risorsa di destinazione**selezionare **servizio app**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="f4fcf-458">Per **risorsa di destinazione**selezionare **Scegli un servizio app** per visualizzare l'elenco delle app Web nella stessa sottoscrizione.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="f4fcf-459">In **risorsa**selezionare il servizio app usato come primo endpoint.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="f4fcf-460">In mapping geografico selezionare un'area/continente in cui si trova la risorsa.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="f4fcf-461">Ad esempio, **America del Nord/America Centrale/Caraibi.**</span><span class="sxs-lookup"><span data-stu-id="f4fcf-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="f4fcf-462">Nell'elenco a discesa paese/area geografica che viene visualizzato lasciare vuoto questo punto per selezionare tutto il raggruppamento a livello di area precedente.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="f4fcf-463">Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="f4fcf-464">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="f4fcf-465">Creare almeno un endpoint con un ambito geografico di All (World) per fungere da endpoint predefinito per la risorsa.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="f4fcf-466">Quando l'aggiunta di entrambi gli endpoint è completa, vengono visualizzati nel **profilo di gestione traffico** insieme al relativo stato di monitoraggio **online**.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Stato dell'endpoint del profilo di gestione traffico](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="f4fcf-468">Global Enterprise si basa sulle funzionalità di distribuzione geografica di Azure</span><span class="sxs-lookup"><span data-stu-id="f4fcf-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="f4fcf-469">L'indirizzamento del traffico dati tramite gli endpoint di gestione traffico di Azure e di geografia consente alle aziende globali di rispettare le normative regionali e di mantenere i dati conformi e sicuri, il che è fondamentale per il successo di posizioni aziendali locali e remote.</span><span class="sxs-lookup"><span data-stu-id="f4fcf-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f4fcf-470">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="f4fcf-470">Next steps</span></span>

- <span data-ttu-id="f4fcf-471">Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="f4fcf-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
