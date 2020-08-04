---
title: Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub di Azure Stack
description: Informazioni su come indirizzare il traffico a endpoint specifici tramite una soluzione app con distribuzione geografica usando Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 741ddf2c3ed234788af359dd233f6a656fbea13c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477355"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="11ce0-103">Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="11ce0-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="11ce0-104">Informazioni su come indirizzare il traffico a endpoint specifici in base a diverse metriche usando il criterio delle app con distribuzione geografica.</span><span class="sxs-lookup"><span data-stu-id="11ce0-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="11ce0-105">La creazione di un profilo di Gestione traffico con routing geografico e configurazione degli endpoint garantisce che le informazioni vengano indirizzate agli endpoint in base ai requisiti internazionali, alle normative aziendali e internazionali e alle esigenze a livello di dati.</span><span class="sxs-lookup"><span data-stu-id="11ce0-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="11ce0-106">In questa soluzione si compilerà un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="11ce0-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="11ce0-107">Creare un'app con distribuzione geografica.</span><span class="sxs-lookup"><span data-stu-id="11ce0-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="11ce0-108">Usare Gestione traffico per definire la destinazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="11ce0-109">Usare il criterio delle app con distribuzione geografica</span><span class="sxs-lookup"><span data-stu-id="11ce0-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="11ce0-110">Con il criterio di distribuzione geografica, l'app può raggiungere più aree geografiche.</span><span class="sxs-lookup"><span data-stu-id="11ce0-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="11ce0-111">È possibile usare come impostazione predefinita il cloud pubblico, ma alcuni utenti potrebbero richiedere che i loro dati risiedano nella loro area geografica.</span><span class="sxs-lookup"><span data-stu-id="11ce0-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="11ce0-112">È possibile indirizzare gli utenti al cloud più adatto ai loro requisiti.</span><span class="sxs-lookup"><span data-stu-id="11ce0-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="11ce0-113">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="11ce0-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="11ce0-114">Considerazioni sulla scalabilità</span><span class="sxs-lookup"><span data-stu-id="11ce0-114">Scalability considerations</span></span>

<span data-ttu-id="11ce0-115">La soluzione che verrà creata in questo articolo non prevede l'adattamento in base alla scalabilità.</span><span class="sxs-lookup"><span data-stu-id="11ce0-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="11ce0-116">Se tuttavia viene usata in combinazione con altre soluzioni Azure e locali, può soddisfare requisiti di scalabilità.</span><span class="sxs-lookup"><span data-stu-id="11ce0-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="11ce0-117">Per informazioni sulla creazione di una soluzione ibrida con scalabilità automatica tramite Gestione traffico, vedere [Distribuire un'app scalabile tra cloud usando Azure e l'hub di Azure Stack](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="11ce0-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="11ce0-118">Considerazioni sulla disponibilità</span><span class="sxs-lookup"><span data-stu-id="11ce0-118">Availability considerations</span></span>

<span data-ttu-id="11ce0-119">Come nel caso delle considerazioni sulla scalabilità, questa soluzione non risolve direttamente la disponibilità.</span><span class="sxs-lookup"><span data-stu-id="11ce0-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="11ce0-120">Tuttavia è possibile implementare al suo interno soluzioni Azure e locali, per garantire la disponibilità elevata per tutti i componenti interessati.</span><span class="sxs-lookup"><span data-stu-id="11ce0-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="11ce0-121">Quando usare questo modello</span><span class="sxs-lookup"><span data-stu-id="11ce0-121">When to use this pattern</span></span>

- <span data-ttu-id="11ce0-122">L'organizzazione dispone di filiali internazionali che richiedono criteri personalizzati per la sicurezza e la distribuzione nelle aree specifiche.</span><span class="sxs-lookup"><span data-stu-id="11ce0-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="11ce0-123">Ogni ufficio dell'organizzazione esegue il pull dei dati relativi a dipendenti, attività e strutture, pertanto è necessaria un'attività di creazione di report basata sulle normative e sui fusi orari locali.</span><span class="sxs-lookup"><span data-stu-id="11ce0-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="11ce0-124">I requisiti di scalabilità elevata vengono soddisfatti con la scalabilità orizzontale delle applicazioni e con più distribuzioni di app in una singola area geografica e in più aree, al fine di gestire i requisiti di carico particolarmente elevati.</span><span class="sxs-lookup"><span data-stu-id="11ce0-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="11ce0-125">Pianificazione della topologia</span><span class="sxs-lookup"><span data-stu-id="11ce0-125">Planning the topology</span></span>

<span data-ttu-id="11ce0-126">Prima di creare un footprint per le app distribuite, è utile sapere quanto segue:</span><span class="sxs-lookup"><span data-stu-id="11ce0-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="11ce0-127">**Dominio personalizzato per l'app:** nome di dominio personalizzato che i clienti useranno per accedere all'app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="11ce0-128">Per l'app di esempio il nome di dominio personalizzato è *www\.scalableasedemo.com*.</span><span class="sxs-lookup"><span data-stu-id="11ce0-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="11ce0-129">**Dominio di Gestione traffico:** quando si crea un [profilo di Gestione traffico di Azure](/azure/traffic-manager/traffic-manager-manage-profiles) è necessario scegliere un nome di dominio.</span><span class="sxs-lookup"><span data-stu-id="11ce0-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="11ce0-130">Questo nome viene combinato con il suffisso *trafficmanager.net* per registrare una voce di dominio gestita da Gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="11ce0-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="11ce0-131">Per l'app di esempio il nome scelto è *scalable-ase-demo*.</span><span class="sxs-lookup"><span data-stu-id="11ce0-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="11ce0-132">Il nome di dominio completo gestito da Gestione traffico sarà quindi *scalable-ase-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="11ce0-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="11ce0-133">**Strategia per la scalabilità del footprint dell'app:** decidere se il footprint dell'app verrà distribuito tra più ambienti del servizio app in una singola area geografica, in più aree o con una combinazione di entrambi gli approcci.</span><span class="sxs-lookup"><span data-stu-id="11ce0-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="11ce0-134">La decisione dovrà basarsi sulla previsione dell'origine del traffico dei clienti, oltre che sull'efficacia della scalabilità nel resto dell'infrastruttura di back-end che supporta un'app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="11ce0-135">Ad esempio, un'app senza stato al 100% può essere ridimensionata notevolmente usando una combinazione di più ambienti del servizio app per ogni area di Azure, moltiplicati per gli ambienti del servizio app distribuiti tra più aree di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="11ce0-136">Con oltre 15 aree di Azure globali tra cui scegliere, i clienti possono creare un footprint dell'app iperscalabile in tutto il mondo.</span><span class="sxs-lookup"><span data-stu-id="11ce0-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="11ce0-137">Per l'app di esempio usata qui sono stati creati tre ambienti del servizio app in una singola area di Azure (Stati Uniti centro-meridionali).</span><span class="sxs-lookup"><span data-stu-id="11ce0-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="11ce0-138">**Convenzione di denominazione per gli ambienti del servizio app:** ogni ambiente del servizio app richiede un nome univoco.</span><span class="sxs-lookup"><span data-stu-id="11ce0-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="11ce0-139">Se si hanno più di due ambienti del servizio app, è utile implementare una convenzione di denominazione per facilitare l'identificazione di ogni ambiente del servizio app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="11ce0-140">Per l'app di esempio usata qui è stata usata una convenzione di denominazione semplice.</span><span class="sxs-lookup"><span data-stu-id="11ce0-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="11ce0-141">I nomi dei tre ambienti del Servizio app di Azure sono *fe1ase*, *fe2ase* e *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="11ce0-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="11ce0-142">**Convenzione di denominazione per le app:** poiché verranno distribuite più istanze dell'app, è necessario definire un nome per ogni istanza dell'app distribuita.</span><span class="sxs-lookup"><span data-stu-id="11ce0-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="11ce0-143">Con l'ambiente del servizio app per Power Apps, lo stesso nome di app può essere usato in più ambienti.</span><span class="sxs-lookup"><span data-stu-id="11ce0-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="11ce0-144">Poiché ogni ambiente del servizio app ha un suffisso univoco, gli sviluppatori possono scegliere di riusare esattamente lo stesso nome dell'app in ogni ambiente.</span><span class="sxs-lookup"><span data-stu-id="11ce0-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="11ce0-145">Ad esempio, uno sviluppatore può avere app denominate come segue: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* e così via.</span><span class="sxs-lookup"><span data-stu-id="11ce0-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="11ce0-146">Per l'app usata nel presente contesto, ogni istanza dell'app ha un nome univoco.</span><span class="sxs-lookup"><span data-stu-id="11ce0-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="11ce0-147">I nomi delle istanze dell'app usati sono *webfrontend1*, *webfrontend2* e *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="11ce0-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="11ce0-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="11ce0-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="11ce0-149">L'hub di Azure Stack è un'estensione di Azure</span><span class="sxs-lookup"><span data-stu-id="11ce0-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="11ce0-150">che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="11ce0-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="11ce0-151">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="11ce0-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="11ce0-152">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="11ce0-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="11ce0-153">Parte 1: Creare un'app con distribuzione geografica</span><span class="sxs-lookup"><span data-stu-id="11ce0-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="11ce0-154">In questa parte si creerà un'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="11ce0-155">Creare e pubblicare app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="11ce0-156">Aggiungere codice ad Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="11ce0-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="11ce0-157">Indirizzare la compilazione dell'app su più destinazioni cloud.</span><span class="sxs-lookup"><span data-stu-id="11ce0-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="11ce0-158">Gestire e configurare il processo di recapito continuo.</span><span class="sxs-lookup"><span data-stu-id="11ce0-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="11ce0-159">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="11ce0-159">Prerequisites</span></span>

<span data-ttu-id="11ce0-160">Sono necessarie una sottoscrizione di Azure e l'installazione dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="11ce0-161">Procedura per app con distribuzione geografica</span><span class="sxs-lookup"><span data-stu-id="11ce0-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="11ce0-162">Ottenere un dominio personalizzato e configurare il DNS</span><span class="sxs-lookup"><span data-stu-id="11ce0-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="11ce0-163">Aggiornare il file della zona DNS per il dominio.</span><span class="sxs-lookup"><span data-stu-id="11ce0-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="11ce0-164">Azure AD può quindi verificare la proprietà del nome di dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="11ce0-165">Usare [DNS Azure](/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in un [registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="11ce0-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="11ce0-166">Registrare un dominio personalizzato con un registrar pubblico.</span><span class="sxs-lookup"><span data-stu-id="11ce0-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="11ce0-167">Accedere al registrar per il dominio.</span><span class="sxs-lookup"><span data-stu-id="11ce0-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="11ce0-168">Per eseguire gli aggiornamenti al DNS potrebbe essere necessario un amministratore approvato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="11ce0-169">Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS specificata da Azure AD.</span><span class="sxs-lookup"><span data-stu-id="11ce0-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="11ce0-170">La voce DNS non modifica comportamenti come il routing della posta elettronica o l'hosting Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="11ce0-171">Creare e pubblicare app Web</span><span class="sxs-lookup"><span data-stu-id="11ce0-171">Create web apps and publish</span></span>

<span data-ttu-id="11ce0-172">Configurare l'integrazione continua e il recapito continuo (CI/CD) ibridi per distribuire l'app Web in Azure e nell'hub di Azure Stack e per eseguire il push automatico delle modifiche in entrambi i cloud.</span><span class="sxs-lookup"><span data-stu-id="11ce0-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="11ce0-173">Sono necessari l'hub di Azure Stack con immagini diffuse appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="11ce0-174">Per altre informazioni, vedere [Prerequisiti per la distribuzione del servizio app nell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="11ce0-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="11ce0-175">Aggiungere codice ad Azure Repos</span><span class="sxs-lookup"><span data-stu-id="11ce0-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="11ce0-176">Accedere a Visual Studio con un **account provvisto di diritti di creazione progetti** in Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="11ce0-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="11ce0-177">L'integrazione continua e il recapito continuo (CI/CD) possono essere applicati sia al codice dell'app sia al codice dell'infrastruttura.</span><span class="sxs-lookup"><span data-stu-id="11ce0-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="11ce0-178">Usare [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) sia per lo sviluppo cloud privato che per quello ospitato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Connettersi a un progetto in Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="11ce0-180">**Clonare il repository** creando e aprendo l'app Web predefinita.</span><span class="sxs-lookup"><span data-stu-id="11ce0-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonare il repository in Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="11ce0-182">Creare una distribuzione app Web in entrambi i cloud</span><span class="sxs-lookup"><span data-stu-id="11ce0-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="11ce0-183">Modificare il file **WebApplication.csproj**: Selezionare `Runtimeidentifier` e aggiungere `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="11ce0-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="11ce0-184">Vedere la documentazione [Distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="11ce0-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Modificare il file di progetto dell'app Web in Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="11ce0-186">**Archiviare il codice in Azure Repos** usando Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="11ce0-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="11ce0-187">Verificare che il **codice dell'applicazione** sia stato archiviato in Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="11ce0-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="11ce0-188">Creare la definizione di compilazione</span><span class="sxs-lookup"><span data-stu-id="11ce0-188">Create the build definition</span></span>

1. <span data-ttu-id="11ce0-189">**Accedere ad Azure Pipelines** per verificare di poter creare definizioni di compilazione.</span><span class="sxs-lookup"><span data-stu-id="11ce0-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="11ce0-190">Aggiungere il codice `-r win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="11ce0-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="11ce0-191">Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.</span><span class="sxs-lookup"><span data-stu-id="11ce0-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Aggiungere codice alla definizione di compilazione in Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="11ce0-193">**Eseguire la compilazione**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-193">**Run the build**.</span></span> <span data-ttu-id="11ce0-194">Il processo di [compilazione della distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="11ce0-195">Uso di un agente ospitato di Azure</span><span class="sxs-lookup"><span data-stu-id="11ce0-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="11ce0-196">L'uso di un agente ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="11ce0-197">La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, e questo consente di eseguire sviluppo, test e distribuzione senza interruzioni.</span><span class="sxs-lookup"><span data-stu-id="11ce0-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="11ce0-198">Gestire e configurare il processo di distribuzione continua</span><span class="sxs-lookup"><span data-stu-id="11ce0-198">Manage and configure the CD process</span></span>

<span data-ttu-id="11ce0-199">Azure DevOps Services offre una pipeline altamente configurabile e facilmente gestibile per i rilasci in più ambienti (ad esempio gli ambienti di sviluppo, gestione temporanea, controllo qualità e produzione) e include la richiesta di approvazioni in fasi specifiche.</span><span class="sxs-lookup"><span data-stu-id="11ce0-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="11ce0-200">Creare una definizione della versione</span><span class="sxs-lookup"><span data-stu-id="11ce0-200">Create release definition</span></span>

1. <span data-ttu-id="11ce0-201">Selezionare il **pulsante con il segno più** per aggiungere una nuova versione nella scheda **Releases** (Rilasci) della sezione **Build and Release** (Compilazione e versione) di Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="11ce0-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Creare una definizione di versione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="11ce0-203">Applicare il modello di distribuzione del Servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-203">Apply the Azure App Service Deployment template.</span></span>

   ![Applicare il modello di distribuzione del Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="11ce0-205">In **Aggiungi artefatto** aggiungere l'artefatto corrispondente all'app della build del cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Aggiungere un elemento alla build del cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="11ce0-207">Nella scheda Pipeline selezionare il collegamento **Phase, Task** (Fase, Attività) dell'ambiente e impostare i valori di ambiente del cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Impostare i valori dell'ambiente cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="11ce0-209">Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="11ce0-211">In **Nome del servizio app** impostare il nome del servizio app di Azure richiesto.</span><span class="sxs-lookup"><span data-stu-id="11ce0-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Impostare il nome del servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="11ce0-213">Immettere "Hosted VS2017" in **Agent queue** (Coda agente) per l'ambiente ospitato del cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Impostare la coda agente per l'ambiente ospitato del cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="11ce0-215">Nel menu Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare l'opzione **Package or Folder** (Pacchetto o cartella) per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="11ce0-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="11ce0-216">Selezionare **OK** per **folder location** (Percorso cartella).</span><span class="sxs-lookup"><span data-stu-id="11ce0-216">Select **OK** to **folder location**.</span></span>
  
      ![Selezionare il pacchetto o la cartella dell'ambiente Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selezionare il pacchetto o la cartella dell'ambiente servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="11ce0-219">Salvare tutte le modifiche e tornare alla **pipeline di versione**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Salvare le modifiche nella pipeline di versione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="11ce0-221">Aggiungere un nuovo artefatto selezionando la build dell'app hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Aggiungere un nuovo artefatto per l'hub di Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="11ce0-223">Aggiungere un altro ambiente applicando la Distribuzione Servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Aggiungere un ambiente a Distribuzione servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="11ce0-225">Assegnare al nuovo ambiente il nome Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="11ce0-225">Name the new environment Azure Stack Hub.</span></span>

    ![Assegnare un nome all'ambiente in Distribuzione servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="11ce0-227">Trovare l'ambiente Azure Stack Hub nella scheda **Tasks** (Attività).</span><span class="sxs-lookup"><span data-stu-id="11ce0-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Ambiente Azure Stack Hub in Azure DevOps Services in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="11ce0-229">Selezionare la sottoscrizione per l'endpoint dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Selezionare la sottoscrizione per l'endpoint dell'hub di Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="11ce0-231">Impostare il nome dell'app Web Azure Stack Hub come nome del servizio app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Impostare il nome dell'app Web Azure Stack Hub in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="11ce0-233">Selezionare l'agente dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-233">Select the Azure Stack Hub agent.</span></span>

    ![Selezionare l'agente dell'hub di Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="11ce0-235">Nella sezione Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare l'opzione **Package or Folder** (Pacchetto o cartella) valida per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="11ce0-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="11ce0-236">Selezionare **OK** per il percorso della cartella.</span><span class="sxs-lookup"><span data-stu-id="11ce0-236">Select **OK** to folder location.</span></span>

    ![Selezionare la cartella per Distribuzione Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selezionare la cartella per Distribuzione Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="11ce0-239">Nella scheda Variable (Variabile) aggiungere una variabile con nome `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, impostare il valore come **true** e l'hub di Azure Stack come ambito.</span><span class="sxs-lookup"><span data-stu-id="11ce0-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Aggiungere una variabile a Distribuzione Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="11ce0-241">Selezionare l'icona del trigger di distribuzione **Continuous** (Continua) in entrambi gli artefatti e abilitare il trigger di distribuzione **Continues** (Continua).</span><span class="sxs-lookup"><span data-stu-id="11ce0-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selezionare il trigger di distribuzione Continuous (Continua) in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="11ce0-243">Selezionare l'icona delle condizioni **Pre-deployment** (Pre-distribuzione) nell'ambiente dell'hub di Azure Stack e impostare il trigger su **Dopo la versione**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Selezionare condizioni pre-distribuzione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="11ce0-245">Salvare tutte le modifiche.</span><span class="sxs-lookup"><span data-stu-id="11ce0-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="11ce0-246">Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) durante la creazione di una definizione di versione da un modello.</span><span class="sxs-lookup"><span data-stu-id="11ce0-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="11ce0-247">Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. Per modificarle è invece necessario selezionare l'elemento dell'ambiente padre.</span><span class="sxs-lookup"><span data-stu-id="11ce0-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="11ce0-248">Parte 2: Aggiornare le opzioni dell'app Web</span><span class="sxs-lookup"><span data-stu-id="11ce0-248">Part 2: Update web app options</span></span>

<span data-ttu-id="11ce0-249">[Servizio app di Azure](/azure/app-service/overview) offre un servizio di hosting Web con scalabilità elevata e funzioni di auto-correzione.</span><span class="sxs-lookup"><span data-stu-id="11ce0-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Servizio app di Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="11ce0-251">Eseguire il mapping di un nome DNS personalizzato esistente ad app Web di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="11ce0-252">Usare un **record CNAME** e un **record A** per eseguire il mapping di un nome DNS personalizzato al servizio app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="11ce0-253">Esecuzione del mapping di un nome DNS personalizzato esistente con un app Web di Azure</span><span class="sxs-lookup"><span data-stu-id="11ce0-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="11ce0-254">Usare un record CNAME per tutti i nomi DNS personalizzati, tranne che per il dominio radice (ad esempio northwind.com).</span><span class="sxs-lookup"><span data-stu-id="11ce0-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="11ce0-255">Per eseguire la migrazione di un sito in tempo reale e del relativo nome di dominio DNS al servizio app, vedere [Migrare un nome DNS attivo nel servizio app di Azure](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="11ce0-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="11ce0-256">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="11ce0-256">Prerequisites</span></span>

<span data-ttu-id="11ce0-257">Per completare questa soluzione:</span><span class="sxs-lookup"><span data-stu-id="11ce0-257">To complete this solution:</span></span>

- <span data-ttu-id="11ce0-258">[Creare un'app del servizio app](/azure/app-service/) oppure usare un'app creata per un'altra soluzione.</span><span class="sxs-lookup"><span data-stu-id="11ce0-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="11ce0-259">Acquistare un nome di dominio e verificare l'accesso al registro DNS per il provider di dominio.</span><span class="sxs-lookup"><span data-stu-id="11ce0-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="11ce0-260">Aggiornare il file della zona DNS per il dominio.</span><span class="sxs-lookup"><span data-stu-id="11ce0-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="11ce0-261">Azure AD verifica quindi la proprietà del nome di dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="11ce0-262">Usare [DNS Azure](/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in un [registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="11ce0-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

- <span data-ttu-id="11ce0-263">Registrare un dominio personalizzato con un registrar pubblico.</span><span class="sxs-lookup"><span data-stu-id="11ce0-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="11ce0-264">Accedere al registrar per il dominio.</span><span class="sxs-lookup"><span data-stu-id="11ce0-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="11ce0-265">Per eseguire gli aggiornamenti al DNS potrebbe essere necessario un amministratore approvato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="11ce0-266">Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS specificata da Azure AD.</span><span class="sxs-lookup"><span data-stu-id="11ce0-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="11ce0-267">Ad esempio, per aggiungere le voci DNS per northwindcloud.com e www\.northwindcloud.com, configurare le impostazioni DNS per il dominio radice northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="11ce0-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="11ce0-268">È possibile acquistare un nome di dominio usando il [portale di Azure](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="11ce0-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="11ce0-269">Per eseguire il mapping di un nome DNS personalizzato a un'app Web, è necessario che il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) dell'app Web sia un livello a pagamento (**Condiviso**, **Base**, **Standard** o **Premium**).</span><span class="sxs-lookup"><span data-stu-id="11ce0-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="11ce0-270">Creare ed eseguire il mapping di record CNAME e A</span><span class="sxs-lookup"><span data-stu-id="11ce0-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="11ce0-271">Accedere ai record DNS con il provider di dominio</span><span class="sxs-lookup"><span data-stu-id="11ce0-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="11ce0-272">È possibile usare DNS di Azure per configurare un nome DNS personalizzato per le app Web di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="11ce0-273">Per altre informazioni, vedere [Usare il servizio DNS di Azure per specificare impostazioni di dominio personalizzate per un servizio di Azure](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="11ce0-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="11ce0-274">Accedere al sito Web del provider principale.</span><span class="sxs-lookup"><span data-stu-id="11ce0-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="11ce0-275">Individuare la pagina relativa alla gestione dei record DNS.</span><span class="sxs-lookup"><span data-stu-id="11ce0-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="11ce0-276">Ogni provider di dominio ha una propria interfaccia di record DNS.</span><span class="sxs-lookup"><span data-stu-id="11ce0-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="11ce0-277">Cercare le aree del sito denominate **Domain Name** (Nome di dominio), **DNS** o **Name Server Management** (Gestione server dei nomi).</span><span class="sxs-lookup"><span data-stu-id="11ce0-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="11ce0-278">È possibile visualizzare la pagina dei record DNS in **My domains** (Domini personali).</span><span class="sxs-lookup"><span data-stu-id="11ce0-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="11ce0-279">Trovare il collegamento con nome **File di zona**, **Record DNS** o **Configurazione avanzata**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="11ce0-280">La schermata seguente è un esempio di pagina di record DNS:</span><span class="sxs-lookup"><span data-stu-id="11ce0-280">The following screenshot is an example of a DNS records page:</span></span>

![Pagina record DNS di esempio](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="11ce0-282">In Domain Name Registrar (Nome registrar del dominio) selezionare **Add or Create** (Aggiungi o crea) per creare un record.</span><span class="sxs-lookup"><span data-stu-id="11ce0-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="11ce0-283">Per alcuni provider esistono collegamenti diversi per aggiungere tipi di record diversi.</span><span class="sxs-lookup"><span data-stu-id="11ce0-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="11ce0-284">Vedere la documentazione del provider.</span><span class="sxs-lookup"><span data-stu-id="11ce0-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="11ce0-285">Aggiungere un record CNAME per eseguire il mapping di un sottodominio al nome host predefinito dell'app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="11ce0-286">Per l'esempio di dominio www\.northwindcloud.com, aggiungere un record CNAME che esegue il mapping del nome a `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="11ce0-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="11ce0-287">Dopo l'aggiunta del record CNAME la pagina dei record DNS è simile all'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="11ce0-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Passaggio all'app di Azure nel portale](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="11ce0-289">Abilitare il mapping dei record CNAME in Azure</span><span class="sxs-lookup"><span data-stu-id="11ce0-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="11ce0-290">In una nuova finestra accedere al portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="11ce0-291">Passare a Servizi app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-291">Go to App Services.</span></span>

3. <span data-ttu-id="11ce0-292">Selezionare l'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-292">Select web app.</span></span>

4. <span data-ttu-id="11ce0-293">Nel riquadro di spostamento a sinistra della pagina dell'app nel portale di Azure selezionare **Domini personalizzati**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="11ce0-294">Selezionare l'icona **+** accanto ad **Aggiungi il nome host**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="11ce0-295">Digitare il nome di dominio completo, ad esempio `www.northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="11ce0-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="11ce0-296">Selezionare **Convalida**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-296">Select **Validate**.</span></span>

8. <span data-ttu-id="11ce0-297">Se indicato, aggiungere altri record di altri tipi (`A` o `TXT`) ai record DNS del registrar del dominio.</span><span class="sxs-lookup"><span data-stu-id="11ce0-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="11ce0-298">Azure specificherà i valori e i tipi di record seguenti:</span><span class="sxs-lookup"><span data-stu-id="11ce0-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="11ce0-299">a.</span><span class="sxs-lookup"><span data-stu-id="11ce0-299">a.</span></span>  <span data-ttu-id="11ce0-300">Un record **A** di cui eseguire il mapping all'indirizzo IP dell'app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="11ce0-301">b.</span><span class="sxs-lookup"><span data-stu-id="11ce0-301">b.</span></span>  <span data-ttu-id="11ce0-302">Un record **TXT** di cui eseguire il mapping al nome host predefinito dell'app `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="11ce0-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="11ce0-303">Il servizio app usa questo record solo in fase di configurazione per verificare la proprietà del dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="11ce0-304">Dopo la verifica, eliminare il record TXT.</span><span class="sxs-lookup"><span data-stu-id="11ce0-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="11ce0-305">Completare questa attività nella scheda del registrar del dominio e riconvalidare fino a quando non viene attivato il pulsante **Aggiungi il nome host**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="11ce0-306">Assicurarsi che **Tipo di record del nome host** sia impostato su **CNAME** (www.example.com o qualsiasi sottodominio).</span><span class="sxs-lookup"><span data-stu-id="11ce0-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="11ce0-307">Selezionare **Aggiungi il nome host**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="11ce0-308">Digitare il nome di dominio completo, ad esempio `northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="11ce0-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="11ce0-309">Selezionare **Convalida**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-309">Select **Validate**.</span></span> <span data-ttu-id="11ce0-310">Viene attivato **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="11ce0-311">Assicurarsi che **Tipo di record del nome host** sia impostato su **Record A** (example.com).</span><span class="sxs-lookup"><span data-stu-id="11ce0-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="11ce0-312">**Aggiungere il nome host**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-312">**Add hostname**.</span></span>

    <span data-ttu-id="11ce0-313">La visualizzazione dei nuovi nomi host nella pagina **Domini personalizzati** dell'app potrebbe richiedere qualche minuto.</span><span class="sxs-lookup"><span data-stu-id="11ce0-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="11ce0-314">Provare ad aggiornare il browser per visualizzare i dati più recenti.</span><span class="sxs-lookup"><span data-stu-id="11ce0-314">Try refreshing the browser to update the data.</span></span>
  
    ![Domini personalizzati](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="11ce0-316">Se si verifica un errore, nella parte inferiore della pagina viene segnalato un errore di verifica.</span><span class="sxs-lookup"><span data-stu-id="11ce0-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Errore di verifica del dominio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="11ce0-318">È possibile ripetere i passaggi precedenti per eseguire il mapping di un dominio con caratteri jolly (\*.northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="11ce0-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="11ce0-319">In questo modo è possibile aggiungere altri sottodomini al servizio app senza dover creare un record CNAME separato per ciascuno di essi.</span><span class="sxs-lookup"><span data-stu-id="11ce0-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="11ce0-320">Per configurare questa impostazione, seguire le istruzioni del registrar.</span><span class="sxs-lookup"><span data-stu-id="11ce0-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="11ce0-321">Eseguire test in un browser</span><span class="sxs-lookup"><span data-stu-id="11ce0-321">Test in a browser</span></span>

<span data-ttu-id="11ce0-322">Passare al nome o ai nomi DNS configurati in precedenza, ad esempio `northwindcloud.com` o `www.northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="11ce0-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="11ce0-323">Parte 3: Associare un certificato SSL personalizzato</span><span class="sxs-lookup"><span data-stu-id="11ce0-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="11ce0-324">In questa parte si eseguiranno le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="11ce0-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="11ce0-325">Associare il certificato SSL personalizzato al Servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="11ce0-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="11ce0-326">Applicare HTTPS per l'app.</span><span class="sxs-lookup"><span data-stu-id="11ce0-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="11ce0-327">Automatizzare l'associazione dei certificati SSL con gli script.</span><span class="sxs-lookup"><span data-stu-id="11ce0-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="11ce0-328">Se necessario, ottenere un certificato SSL del cliente nel portale di Azure e associarlo all'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="11ce0-329">Per altre informazioni, vedere l'[esercitazione sui certificati del Servizio app](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="11ce0-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="11ce0-330">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="11ce0-330">Prerequisites</span></span>

<span data-ttu-id="11ce0-331">Per completare questa soluzione:</span><span class="sxs-lookup"><span data-stu-id="11ce0-331">To complete this  solution:</span></span>

- <span data-ttu-id="11ce0-332">[Creare un'app del Servizio app di Azure](/azure/app-service/).</span><span class="sxs-lookup"><span data-stu-id="11ce0-332">[Create an App Service app.](/azure/app-service/)</span></span>
- [<span data-ttu-id="11ce0-333">Eseguire il mapping di un nome DNS personalizzato all'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="11ce0-334">Acquisire un certificato SSL da un'autorità di certificazione attendibile e usare la chiave per firmare la richiesta.</span><span class="sxs-lookup"><span data-stu-id="11ce0-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="11ce0-335">Requisiti per il certificato SSL</span><span class="sxs-lookup"><span data-stu-id="11ce0-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="11ce0-336">Per poter essere usato nel servizio app, il certificato deve soddisfare tutti i requisiti seguenti:</span><span class="sxs-lookup"><span data-stu-id="11ce0-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="11ce0-337">Deve essere firmato da un'autorità di certificazione attendibile.</span><span class="sxs-lookup"><span data-stu-id="11ce0-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="11ce0-338">Deve essere esportato come file PFX protetto da password.</span><span class="sxs-lookup"><span data-stu-id="11ce0-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="11ce0-339">Deve contenere una chiave privata costituita da almeno 2048 bit.</span><span class="sxs-lookup"><span data-stu-id="11ce0-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="11ce0-340">Deve contenere tutti i certificati intermedi nella catena di certificati.</span><span class="sxs-lookup"><span data-stu-id="11ce0-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="11ce0-341">I **certificati di crittografia a curva ellittica (ECC)** funzionano con il Servizio app di Azure, ma non sono inclusi in questa guida.</span><span class="sxs-lookup"><span data-stu-id="11ce0-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="11ce0-342">Per assistenza nella creazione di certificati ECC contattare un'autorità di certificazione.</span><span class="sxs-lookup"><span data-stu-id="11ce0-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="11ce0-343">Preparare l'app Web</span><span class="sxs-lookup"><span data-stu-id="11ce0-343">Prepare the web app</span></span>

<span data-ttu-id="11ce0-344">Per l'associazione di un certificato SSL personalizzato all'app Web, il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) in uso deve essere di livello **Basic**, **Standard** o **Premium**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="11ce0-345">Accedere ad Azure</span><span class="sxs-lookup"><span data-stu-id="11ce0-345">Sign in to Azure</span></span>

1. <span data-ttu-id="11ce0-346">Accedere al [portale di Azure](https://portal.azure.com/) e andare all'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="11ce0-347">Nel menu a sinistra selezionare **Servizi app** e quindi selezionare il nome dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Selezionare l'app Web nel portale di Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="11ce0-349">Scegliere il piano tariffario</span><span class="sxs-lookup"><span data-stu-id="11ce0-349">Check the pricing tier</span></span>

1. <span data-ttu-id="11ce0-350">Nel riquadro di spostamento a sinistra della pagina dell'app Web scorrere fino alla sezione **Impostazioni** e selezionare **Aumenta prestazioni (piano di servizio app)** .</span><span class="sxs-lookup"><span data-stu-id="11ce0-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menu Aumenta prestazioni nell'app Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="11ce0-352">Assicurarsi che l'app Web non appartenga al livello **Gratis** o **Condiviso**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="11ce0-353">Il livello corrente dell'app Web è evidenziato in una casella blu scuro.</span><span class="sxs-lookup"><span data-stu-id="11ce0-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Controllare il piano tariffario nell'app Web](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="11ce0-355">Il certificato SSL personalizzato non è supportato nei livelli **Gratuito** o **Condiviso**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="11ce0-356">Per passare a un piano di servizio superiore, seguire i passaggi nella sezione successiva o nella pagina **Scegliere il piano tariffario** e passare a [Caricare il certificato SSL e Associare il certificato SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="11ce0-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="11ce0-357">Passare a un piano di servizio app superiore</span><span class="sxs-lookup"><span data-stu-id="11ce0-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="11ce0-358">Selezionare uno tra i livelli **Basic**, **Standard** o **Premium**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="11ce0-359">Scegliere **Seleziona**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-359">Select **Select**.</span></span>

![Scegliere il piano tariffario per l'app Web.](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="11ce0-361">L'operazione di passaggio a un livello superiore viene completata quando viene visualizzata la notifica.</span><span class="sxs-lookup"><span data-stu-id="11ce0-361">The scale operation is complete when notification is displayed.</span></span>

![Notifica di passaggio al livello superiore](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="11ce0-363">Associare il certificato SSL e unire i certificati intermedi</span><span class="sxs-lookup"><span data-stu-id="11ce0-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="11ce0-364">Unire più certificati nella catena.</span><span class="sxs-lookup"><span data-stu-id="11ce0-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="11ce0-365">**Aprire ogni certificato** ricevuto in un editor di testo.</span><span class="sxs-lookup"><span data-stu-id="11ce0-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="11ce0-366">Creare un file per il certificato unito denominato *mergedcertificate.crt*.</span><span class="sxs-lookup"><span data-stu-id="11ce0-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="11ce0-367">In un editor di testo copiare il contenuto di ogni certificato nel file.</span><span class="sxs-lookup"><span data-stu-id="11ce0-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="11ce0-368">L'ordine dei certificati deve corrispondere all'ordine nella catena di certificati, che inizia con il certificato dell'utente e termina con il certificato radice.</span><span class="sxs-lookup"><span data-stu-id="11ce0-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="11ce0-369">Sarà simile a quanto indicato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="11ce0-369">It looks like the following example:</span></span>

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

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="11ce0-370">Esportare il certificato in un file PFX</span><span class="sxs-lookup"><span data-stu-id="11ce0-370">Export certificate to PFX</span></span>

<span data-ttu-id="11ce0-371">Esportare il certificato SSL unito con la chiave privata generata dal certificato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="11ce0-372">Un file di chiave privata viene creato tramite OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="11ce0-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="11ce0-373">Per esportare il certificato in un file con estensione pfx, eseguire il comando seguente e sostituire i segnaposto `<private-key-file>` e `<merged-certificate-file>` con il percorso della chiave privata e il file del certificato unito:</span><span class="sxs-lookup"><span data-stu-id="11ce0-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="11ce0-374">Quando richiesto, definire una password di esportazione per il caricamento del certificato SSL nel servizio app in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="11ce0-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="11ce0-375">Quando si usa IIS o **Certreq.exe** per generare la richiesta di certificato, installare il certificato in un computer locale e quindi [esportarlo in un file con estensione pfx](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="11ce0-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="11ce0-376">Caricare il certificato SSL</span><span class="sxs-lookup"><span data-stu-id="11ce0-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="11ce0-377">Selezionare **Impostazioni SSL** nel riquadro di spostamento a sinistra dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="11ce0-378">Selezionare **Carica certificato**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="11ce0-379">In **File del certificato PFX** selezionare il file con estensione pfx.</span><span class="sxs-lookup"><span data-stu-id="11ce0-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="11ce0-380">In **Password certificato** digitare la password creata durante l'esportazione del file con estensione pfx.</span><span class="sxs-lookup"><span data-stu-id="11ce0-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="11ce0-381">Selezionare **Carica**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-381">Select **Upload**.</span></span>

    ![Caricare il certificato SSL](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="11ce0-383">Dopo che il Servizio app ha terminato il caricamento del certificato, viene visualizzata la pagina **Impostazioni SSL**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Impostazioni SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="11ce0-385">Associare il certificato SSL</span><span class="sxs-lookup"><span data-stu-id="11ce0-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="11ce0-386">Nella sezione **Associazioni SSL** selezionare **Aggiungi binding**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="11ce0-387">Se il certificato è stato caricato ma non viene visualizzato nei nomi di dominio nell'elenco a discesa **Nome host**, provare ad aggiornare la pagina del browser.</span><span class="sxs-lookup"><span data-stu-id="11ce0-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="11ce0-388">Nel pannello **Aggiungi associazione SSL** usare gli elenchi a discesa per selezionare il nome di dominio da proteggere e il certificato da usare.</span><span class="sxs-lookup"><span data-stu-id="11ce0-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="11ce0-389">In **Tipo SSL** selezionare se usare l'SSL basato su [**indicazione nome server (SNI)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) o basato su IP.</span><span class="sxs-lookup"><span data-stu-id="11ce0-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="11ce0-390">**SSL basato su SNI**: è possibile aggiungere più associazioni SSL basate su SNI.</span><span class="sxs-lookup"><span data-stu-id="11ce0-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="11ce0-391">Questa opzione consente di usare più certificati SSL per proteggere più domini nello stesso indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="11ce0-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="11ce0-392">La maggior parte dei browser moderni (tra cui Internet Explorer, Chrome, Firefox e Opera) supporta SNI. Per altre informazioni sul supporto dei browser, vedere [Indicazione nome server](https://wikipedia.org/wiki/Server_Name_Indication).</span><span class="sxs-lookup"><span data-stu-id="11ce0-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="11ce0-393">**SSL basato su IP**: è possibile aggiungere una sola associazione SSL basata su IP.</span><span class="sxs-lookup"><span data-stu-id="11ce0-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="11ce0-394">Questa opzione consente di usare solo un certificato SSL per proteggere un indirizzo IP pubblico dedicato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="11ce0-395">Se si proteggono più domini, proteggerli tutti usando lo stesso certificato SSL.</span><span class="sxs-lookup"><span data-stu-id="11ce0-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="11ce0-396">SSL basato su IP è l'opzione standard per il binding SSL.</span><span class="sxs-lookup"><span data-stu-id="11ce0-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="11ce0-397">Selezionare **Aggiungi binding**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-397">Select **Add Binding**.</span></span>

    ![Aggiungere il binding SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="11ce0-399">Al termine del caricamento da parte del Servizio app, il certificato viene visualizzato nella sezione **Binding SSL**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Caricamento dei binding SSL completato](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="11ce0-401">Nuovo mapping del record A per IP SSL</span><span class="sxs-lookup"><span data-stu-id="11ce0-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="11ce0-402">Se non si usa SSL basato su IP nell'app Web, passare a [Testare HTTPS per il dominio personalizzato](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="11ce0-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="11ce0-403">Per impostazione predefinita l'app Web usa un indirizzo IP pubblico condiviso.</span><span class="sxs-lookup"><span data-stu-id="11ce0-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="11ce0-404">Quando il certificato è associato a SSL basato su IP, il servizio app crea un nuovo indirizzo IP dedicato per l'app Web.</span><span class="sxs-lookup"><span data-stu-id="11ce0-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="11ce0-405">Quando viene eseguito il mapping di un record A all'app Web, è necessario aggiornare il registro del dominio con l'indirizzo IP dedicato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="11ce0-406">La pagina **Dominio personalizzato** viene aggiornata con il nuovo indirizzo IP dedicato.</span><span class="sxs-lookup"><span data-stu-id="11ce0-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="11ce0-407">Copiare questo [indirizzo IP](/azure/app-service/app-service-web-tutorial-custom-domain), quindi eseguire nuovamente il mapping del [record A](/azure/app-service/app-service-web-tutorial-custom-domain) su questo indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="11ce0-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="11ce0-408">Testare HTTPS</span><span class="sxs-lookup"><span data-stu-id="11ce0-408">Test HTTPS</span></span>

<span data-ttu-id="11ce0-409">In diversi browser, passare a `https://<your.custom.domain>` per assicurarsi che l'app Web funzioni correttamente.</span><span class="sxs-lookup"><span data-stu-id="11ce0-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Passare all'app Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="11ce0-411">Se si verificano errori di convalida del certificato, la causa può essere un certificato autofirmato oppure l'omissione di certificati intermedi durante l'esportazione nel file con estensione pfx.</span><span class="sxs-lookup"><span data-stu-id="11ce0-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="11ce0-412">Applicare HTTPS</span><span class="sxs-lookup"><span data-stu-id="11ce0-412">Enforce HTTPS</span></span>

<span data-ttu-id="11ce0-413">Per impostazione predefinita, chiunque può accedere all'app Web tramite HTTP.</span><span class="sxs-lookup"><span data-stu-id="11ce0-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="11ce0-414">Tutte le richieste HTTP alla porta HTTPS possono essere reindirizzate.</span><span class="sxs-lookup"><span data-stu-id="11ce0-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="11ce0-415">Nella pagina dell'app Web selezionare **Impostazioni SSL**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="11ce0-416">Quindi in **Solo HTTPS**, selezionare **Attiva**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Applicare HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="11ce0-418">Al termine dell'operazione, passare a uno degli URL HTTP che fanno riferimento all'applicazione.</span><span class="sxs-lookup"><span data-stu-id="11ce0-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="11ce0-419">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="11ce0-419">For example:</span></span>

- <span data-ttu-id="11ce0-420">https://<nome_app>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="11ce0-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="11ce0-421">Applicare TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="11ce0-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="11ce0-422">Per impostazione predefinita l'app supporta [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0, che non è più considerato sicuro in base agli standard di settore come [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard).</span><span class="sxs-lookup"><span data-stu-id="11ce0-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="11ce0-423">Per applicare versioni di TLS più recenti, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="11ce0-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="11ce0-424">Nel riquadro di spostamento a sinistra della pagina dell'app Web selezionare **Impostazioni SSL**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="11ce0-425">In **Versione TLS** selezionare la versione minima di TLS.</span><span class="sxs-lookup"><span data-stu-id="11ce0-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Applicare TLS 1.1 o 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="11ce0-427">Creare un profilo di Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="11ce0-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="11ce0-428">Selezionare **Crea una risorsa** > **Rete** > **Profilo di Gestione traffico** > **Crea**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="11ce0-429">In **Crea profilo di Gestione traffico** procedere come segue:</span><span class="sxs-lookup"><span data-stu-id="11ce0-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="11ce0-430">In **Nome** specificare un nome per il profilo.</span><span class="sxs-lookup"><span data-stu-id="11ce0-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="11ce0-431">Questo nome deve essere univoco all'interno della zona trafficmanager.net e determina il nome DNS, trafficmanager.net, usato per accedere al profilo di Gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="11ce0-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="11ce0-432">In **Metodo di routing** selezionare il metodo di routing **Geografico**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="11ce0-433">In **Sottoscrizione** selezionare la sottoscrizione in cui si vuole creare il profilo.</span><span class="sxs-lookup"><span data-stu-id="11ce0-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="11ce0-434">In **Gruppo di risorse** creare un nuovo gruppo di risorse in cui aggiungere il profilo.</span><span class="sxs-lookup"><span data-stu-id="11ce0-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="11ce0-435">In **Località del gruppo di risorse** selezionare la località del gruppo di risorse.</span><span class="sxs-lookup"><span data-stu-id="11ce0-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="11ce0-436">Questa impostazione indica la località del gruppo di risorse e non ha alcun impatto sul profilo di Gestione traffico distribuito a livello globale.</span><span class="sxs-lookup"><span data-stu-id="11ce0-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="11ce0-437">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-437">Select **Create**.</span></span>

    7. <span data-ttu-id="11ce0-438">Dopo il completamento della distribuzione globale del profilo di Gestione traffico, il profilo viene elencato come risorsa nel rispettivo gruppo di risorse.</span><span class="sxs-lookup"><span data-stu-id="11ce0-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Gruppi di risorse in Crea profilo di Gestione traffico](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="11ce0-440">Aggiungere endpoint di Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="11ce0-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="11ce0-441">Nella barra di ricerca del portale cercare il nome del **profilo di Gestione traffico** creato nella sezione precedente e selezionarlo nei risultati visualizzati.</span><span class="sxs-lookup"><span data-stu-id="11ce0-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="11ce0-442">In **Profilo di Gestione traffico** nella sezione **Impostazioni** selezionare **Endpoint**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="11ce0-443">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-443">Select **Add**.</span></span>

4. <span data-ttu-id="11ce0-444">Aggiunta dell'endpoint dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="11ce0-445">In **Tipo** selezionare **Endpoint esterno**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="11ce0-446">Specificare un **Nome** per l'endpoint, idealmente il nome dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="11ce0-447">Come nome di dominio completo (**FQDN**) usare l'URL esterno dell'app Web hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="11ce0-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="11ce0-448">In Mapping geografico selezionare l'area geografica/continente in cui si trova la risorsa.</span><span class="sxs-lookup"><span data-stu-id="11ce0-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="11ce0-449">Ad esempio, **Europa**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="11ce0-450">Nell'elenco a discesa Paese/area geografica visualizzato selezionare il paese corrispondente a questo endpoint.</span><span class="sxs-lookup"><span data-stu-id="11ce0-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="11ce0-451">Ad esempio, **Germania**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="11ce0-452">Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="11ce0-453">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-453">Select **OK**.</span></span>

12. <span data-ttu-id="11ce0-454">Aggiunta dell'endpoint di Azure:</span><span class="sxs-lookup"><span data-stu-id="11ce0-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="11ce0-455">In **Tipo** selezionare **Endpoint Azure**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="11ce0-456">In **Nome** immettere un nome per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="11ce0-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="11ce0-457">In **Tipo di risorsa di destinazione** fare clic su **Servizio app**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="11ce0-458">In **Risorsa di destinazione** selezionare **Scegliere un servizio app** per visualizzare l'elenco delle App Web nella stessa sottoscrizione.</span><span class="sxs-lookup"><span data-stu-id="11ce0-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="11ce0-459">In **Risorsa** selezionare il servizio app usato come primo endpoint.</span><span class="sxs-lookup"><span data-stu-id="11ce0-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="11ce0-460">In Mapping geografico selezionare l'area geografica/continente in cui si trova la risorsa.</span><span class="sxs-lookup"><span data-stu-id="11ce0-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="11ce0-461">Ad esempio **America del Nord/America centrale/Caraibi.**</span><span class="sxs-lookup"><span data-stu-id="11ce0-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="11ce0-462">Nell'elenco a discesa Paese/area geografica visualizzato non immettere nessun valore, per selezionare l'intero gruppo di area precedente.</span><span class="sxs-lookup"><span data-stu-id="11ce0-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="11ce0-463">Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="11ce0-464">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="11ce0-465">Creare almeno un endpoint con ambito geografico Tutte (mondo) che svolga la funzione di endpoint predefinito per la risorsa.</span><span class="sxs-lookup"><span data-stu-id="11ce0-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="11ce0-466">Dopo il completamento dell'aggiunta di entrambi gli endpoint, questi vengono visualizzati in **Profilo di Gestione traffico** insieme al relativo stato di monitoraggio **Online**.</span><span class="sxs-lookup"><span data-stu-id="11ce0-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Stato dell'endpoint in Profilo di Gestione traffico](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="11ce0-468">Le aziende globali fanno affidamento sulle funzionalità di distribuzione geografica di Azure</span><span class="sxs-lookup"><span data-stu-id="11ce0-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="11ce0-469">L'indirizzamento del traffico di dati tramite Gestione traffico di Azure e gli endpoint geografici specifici consente alle aziende globali di rispettare le normative locali e di garantire la sicurezza e la conformità dei dati. Queste caratteristiche sono fondamentali per il funzionamento corretto di sedi operative locali e remote.</span><span class="sxs-lookup"><span data-stu-id="11ce0-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="11ce0-470">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="11ce0-470">Next steps</span></span>

- <span data-ttu-id="11ce0-471">Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="11ce0-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
