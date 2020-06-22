---
title: Distribuire un'app con scalabilità tra cloud in Azure e hub Azure Stack
description: Informazioni su come distribuire un'app con scalabilità tra cloud in Azure e hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911524"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="d3bd0-103">Distribuire un'app scalabile tra cloud usando Azure e l'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="d3bd0-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d3bd0-104">Informazioni su come creare una soluzione tra cloud per fornire un processo attivato manualmente per passare da un'app Web ospitata nell'hub Azure Stack a un'app Web ospitata in Azure con scalabilità automatica tramite Gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="d3bd0-105">Questo processo garantisce un'utilità cloud flessibile e scalabile in fase di caricamento.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="d3bd0-106">Con questo modello, il tenant potrebbe non essere pronto per l'esecuzione dell'app nel cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="d3bd0-107">Tuttavia, potrebbe non essere economicamente fattibile per l'azienda mantenere la capacità richiesta nell'ambiente locale per gestire i picchi di domanda per l'app.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="d3bd0-108">Il tenant può sfruttare l'elasticità del cloud pubblico con la soluzione locale.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="d3bd0-109">In questa soluzione verrà compilato un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="d3bd0-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d3bd0-110">Creare un'app Web a più nodi.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="d3bd0-111">Configurare e gestire il processo di distribuzione continua (CD).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="d3bd0-112">Pubblicare l'app Web nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="d3bd0-113">Creare una versione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-113">Create a release.</span></span>
> - <span data-ttu-id="d3bd0-114">Informazioni su come monitorare e tenere traccia delle distribuzioni.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="d3bd0-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d3bd0-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d3bd0-116">Microsoft Azure Stack Hub è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d3bd0-117">Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d3bd0-118">L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d3bd0-119">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d3bd0-120">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="d3bd0-120">Prerequisites</span></span>

- <span data-ttu-id="d3bd0-121">Sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-121">Azure subscription.</span></span> <span data-ttu-id="d3bd0-122">Se necessario, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) prima di iniziare.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="d3bd0-123">Un sistema integrato di Azure Stack Hub o una distribuzione di Azure Stack Development Kit (Gabriele).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="d3bd0-124">Per istruzioni sull'installazione di Azure Stack Hub, vedere [Install the Gabriele](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="d3bd0-125">Per uno script di automazione post-distribuzione di Gabriele, vedere:[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="d3bd0-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="d3bd0-126">Per il completamento di questa installazione potrebbero essere necessarie alcune ore.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="d3bd0-127">Distribuire i servizi PaaS del [servizio app](/azure-stack/operator/azure-stack-app-service-deploy.md) nell'hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="d3bd0-128">[Consente di creare piani/offerte](/azure-stack/operator/service-plan-offer-subscription-overview.md) all'interno dell'ambiente Azure stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="d3bd0-129">[Creare una sottoscrizione tenant](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) all'interno dell'ambiente Azure stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="d3bd0-130">Creare un'app Web all'interno della sottoscrizione tenant.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="d3bd0-131">Prendere nota del nuovo URL dell'app Web per usarlo in seguito.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="d3bd0-132">Distribuire Azure Pipelines macchina virtuale (VM) nella sottoscrizione tenant.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="d3bd0-133">È necessaria una macchina virtuale Windows Server 2016 con .NET 3,5.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="d3bd0-134">Questa macchina virtuale verrà compilata nella sottoscrizione tenant nell'hub Azure Stack come agente di compilazione privato.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="d3bd0-135">[Windows Server 2016 con immagine di macchina virtuale SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) è disponibile nel Marketplace di hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="d3bd0-136">Se questa immagine non è disponibile, usare un operatore Azure Stack hub per assicurarsi che venga aggiunta all'ambiente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="d3bd0-137">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="d3bd0-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="d3bd0-138">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="d3bd0-138">Scalability</span></span>

<span data-ttu-id="d3bd0-139">Il componente principale del ridimensionamento tra cloud è la possibilità di fornire scalabilità immediata e su richiesta tra l'infrastruttura cloud pubblica e locale, fornendo un servizio coerente e affidabile.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="d3bd0-140">Disponibilità</span><span class="sxs-lookup"><span data-stu-id="d3bd0-140">Availability</span></span>

<span data-ttu-id="d3bd0-141">Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata tramite la configurazione hardware locale e la distribuzione software.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="d3bd0-142">Gestione</span><span class="sxs-lookup"><span data-stu-id="d3bd0-142">Manageability</span></span>

<span data-ttu-id="d3bd0-143">La soluzione tra cloud garantisce una gestione semplice e un'interfaccia familiare tra gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="d3bd0-144">PowerShell è consigliato per la gestione multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="d3bd0-145">Scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="d3bd0-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="d3bd0-146">Ottenere un dominio personalizzato e configurare DNS</span><span class="sxs-lookup"><span data-stu-id="d3bd0-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="d3bd0-147">Aggiornare il file di zona DNS per il dominio.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="d3bd0-148">Azure AD verificherà la proprietà del nome di dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="d3bd0-149">Usare [DNS di Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in [un registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-149">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="d3bd0-150">Registrare un dominio personalizzato con un registrar pubblico.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="d3bd0-151">Accedere al registrar per il dominio.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="d3bd0-152">Per eseguire gli aggiornamenti DNS, potrebbe essere necessario un amministratore approvato.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="d3bd0-153">Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS fornita da Azure AD.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="d3bd0-154">La voce DNS non influirà sul routing della posta elettronica o sui comportamenti di hosting Web.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="d3bd0-155">Creare un'app Web multinodo predefinita nell'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="d3bd0-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="d3bd0-156">Configurare l'integrazione continua ibrida e la distribuzione continua (CI/CD) per distribuire app Web in Azure e Azure Stack Hub e per eseguire il push delle modifiche in entrambi i cloud.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="d3bd0-157">È necessario Azure Stack Hub con le immagini appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="d3bd0-158">Per altre informazioni, vedere la documentazione del servizio app [prerequisiti per la distribuzione del servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="d3bd0-159">Aggiungi codice a Azure Repos</span><span class="sxs-lookup"><span data-stu-id="d3bd0-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="d3bd0-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="d3bd0-160">Azure Repos</span></span>

1. <span data-ttu-id="d3bd0-161">Accedere a Azure Repos con un account che disponga dei diritti di creazione del progetto per Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="d3bd0-162">CI/CD ibrido può essere applicato sia al codice dell'app che al codice dell'infrastruttura.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="d3bd0-163">USA [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) per lo sviluppo cloud privato e ospitato.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Connettersi a un progetto in Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="d3bd0-165">**Clonare il repository** creando e aprendo l'app Web predefinita.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonare il repository nell'app Web di Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="d3bd0-167">Creare una distribuzione di app Web indipendente per i servizi app in entrambi i cloud</span><span class="sxs-lookup"><span data-stu-id="d3bd0-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="d3bd0-168">Modificare il file **WebApplication. csproj** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="d3bd0-169">Selezionare `Runtimeidentifier` e aggiungere `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="d3bd0-170">Vedere la documentazione sulla [distribuzione indipendente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-170">(See [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Modificare il file di progetto dell'app Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="d3bd0-172">Archiviare il codice per Azure Repos usando Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="d3bd0-173">Verificare che il codice dell'app sia stato archiviato Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="d3bd0-174">Creare la definizione di compilazione</span><span class="sxs-lookup"><span data-stu-id="d3bd0-174">Create the build definition</span></span>

1. <span data-ttu-id="d3bd0-175">Accedere a Azure Pipelines per verificare la possibilità di creare definizioni di compilazione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="d3bd0-176">Add **-r WIN10-x64** code.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="d3bd0-177">Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Aggiungere codice all'app Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="d3bd0-179">Eseguire la compilazione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-179">Run the build.</span></span> <span data-ttu-id="d3bd0-180">Il processo di [compilazione della distribuzione autonoma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti eseguiti in Azure e in hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-180">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="d3bd0-181">Usare un agente ospitato di Azure</span><span class="sxs-lookup"><span data-stu-id="d3bd0-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="d3bd0-182">L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="d3bd0-183">La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, consentendo un ciclo di sviluppo continuo e senza interruzioni.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="d3bd0-184">Gestire e configurare il processo CD</span><span class="sxs-lookup"><span data-stu-id="d3bd0-184">Manage and configure the CD process</span></span>

<span data-ttu-id="d3bd0-185">Azure Pipelines e Azure DevOps Services forniscono una pipeline altamente configurabile e gestibile per i rilasci in più ambienti come gli ambienti di sviluppo, staging, controllo di qualità e produzione; inclusa la richiesta di approvazioni in fasi specifiche.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="d3bd0-186">Creare una definizione della versione</span><span class="sxs-lookup"><span data-stu-id="d3bd0-186">Create release definition</span></span>

1. <span data-ttu-id="d3bd0-187">Selezionare il pulsante **più** per aggiungere una nuova versione nella scheda **versioni** della sezione **compilazione e versione** di Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Creare una definizione di versione](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="d3bd0-189">Applicare il modello di distribuzione del servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-189">Apply the Azure App Service Deployment template.</span></span>

   ![Applicare il modello di distribuzione del servizio app Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="d3bd0-191">In **Aggiungi artefatto**aggiungere l'artefatto per l'app Azure cloud Build.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Aggiungere un elemento alla compilazione cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="d3bd0-193">In scheda pipeline selezionare la **fase,** il collegamento all'attività dell'ambiente e impostare i valori dell'ambiente cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Impostare i valori dell'ambiente cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="d3bd0-195">Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="d3bd0-197">In **nome servizio app**impostare il nome del servizio app di Azure richiesto.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Imposta il nome del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="d3bd0-199">Immettere "Hosted VS2017" nella **coda dell'agente per l'** ambiente ospitato nel cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Impostare la coda agente per l'ambiente ospitato nel cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="d3bd0-201">Nel menu Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="d3bd0-202">Selezionare **OK** per **percorso cartella**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-202">Select **OK** to **folder location**.</span></span>
  
      ![Selezionare il pacchetto o la cartella per l'ambiente del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selezionare il pacchetto o la cartella per l'ambiente del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="d3bd0-205">Salvare tutte le modifiche e tornare alla **pipeline di rilascio**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Salva le modifiche nella pipeline di versione](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="d3bd0-207">Aggiungere un nuovo elemento selezionando la compilazione per l'App Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Aggiungi nuovo artefatto per Azure Stack App Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="d3bd0-209">Aggiungere un altro ambiente applicando la distribuzione del servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Aggiungi ambiente alla distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="d3bd0-211">Assegnare al nuovo ambiente il nome "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="d3bd0-211">Name the new environment "Azure Stack".</span></span>

    ![Ambiente del nome nella distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="d3bd0-213">Trovare l'ambiente Azure Stack nella scheda **attività** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Ambiente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="d3bd0-215">Selezionare la sottoscrizione per l'endpoint Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Selezionare la sottoscrizione per l'endpoint Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="d3bd0-217">Impostare il nome dell'app Web Azure Stack come nome del servizio app.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="d3bd0-218">![Imposta il nome dell'app Web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="d3bd0-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="d3bd0-219">Selezionare l'agente di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-219">Select the Azure Stack agent.</span></span>

    ![Selezionare l'agente di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="d3bd0-221">Nella sezione Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="d3bd0-222">Selezionare **OK** per percorso cartella.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-222">Select **OK** to folder location.</span></span>

    ![Selezionare la cartella per la distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selezionare la cartella per la distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="d3bd0-225">In scheda variabile aggiungere una variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , impostarne il valore su **true**e l'ambito su Azure stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Aggiungere una variabile alla distribuzione app Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="d3bd0-227">Selezionare l'icona del trigger di distribuzione **continua** in entrambi gli artefatti e abilitare il trigger di distribuzione **continua** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selezionare il trigger di distribuzione continua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="d3bd0-229">Selezionare l'icona condizioni **pre-distribuzione** nell'ambiente Azure stack e impostare il trigger su **dopo il rilascio.**</span><span class="sxs-lookup"><span data-stu-id="d3bd0-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Selezionare le condizioni di pre-distribuzione](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="d3bd0-231">Salvare tutte le modifiche.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="d3bd0-232">Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) quando si crea una definizione di versione da un modello.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-232">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="d3bd0-233">Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. per modificare queste impostazioni è necessario invece selezionare l'elemento dell'ambiente padre.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="d3bd0-234">Pubblicare nell'hub Azure Stack tramite Visual Studio</span><span class="sxs-lookup"><span data-stu-id="d3bd0-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="d3bd0-235">Creando gli endpoint, una Azure DevOps Services Build può distribuire le app del servizio di Azure nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="d3bd0-236">Azure Pipelines si connette all'agente di compilazione, che si connette all'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="d3bd0-237">Accedere a Azure DevOps Services e passare alla pagina delle impostazioni dell'app.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="d3bd0-238">Su **Impostazioni**, selezionare **Sicurezza**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="d3bd0-239">In **gruppi VSTS**selezionare **creatori endpoint**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="d3bd0-240">Nella scheda **Membri** selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="d3bd0-241">In **Aggiungi utenti e gruppi**immettere un nome utente e selezionare l'utente dall'elenco di utenti.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="d3bd0-242">Selezionare **Save changes** (Salva modifiche).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="d3bd0-243">Nell'elenco **gruppi VSTS** selezionare **amministratori endpoint**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="d3bd0-244">Nella scheda **Membri** selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="d3bd0-245">In **Aggiungi utenti e gruppi**immettere un nome utente e selezionare l'utente dall'elenco di utenti.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="d3bd0-246">Selezionare **Save changes** (Salva modifiche).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-246">Select **Save changes**.</span></span>

<span data-ttu-id="d3bd0-247">Ora che le informazioni sull'endpoint esistono, il Azure Pipelines Azure Stack connessione hub è pronto per l'uso.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="d3bd0-248">L'agente di compilazione nell'hub Azure Stack ottiene le istruzioni da Azure Pipelines e quindi l'agente trasmette informazioni sugli endpoint per la comunicazione con l'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="d3bd0-249">Sviluppare la compilazione dell'app</span><span class="sxs-lookup"><span data-stu-id="d3bd0-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="d3bd0-250">È necessario Azure Stack Hub con le immagini appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="d3bd0-251">Per altre informazioni, vedere [prerequisiti per la distribuzione del servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="d3bd0-252">Usare [modelli Azure Resource Manager](https://azure.microsoft.com/resources/templates/) come il codice dell'app web da Azure Repos per eseguire la distribuzione in entrambi i cloud.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="d3bd0-253">Aggiungere codice a un progetto Azure Repos</span><span class="sxs-lookup"><span data-stu-id="d3bd0-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="d3bd0-254">Accedere a Azure Repos con un account che disponga dei diritti di creazione del progetto nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="d3bd0-255">**Clonare il repository** creando e aprendo l'app Web predefinita.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="d3bd0-256">Creare una distribuzione di app Web indipendente per i servizi app in entrambi i cloud</span><span class="sxs-lookup"><span data-stu-id="d3bd0-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="d3bd0-257">Modificare il file **WebApplication. csproj** : selezionare `Runtimeidentifier` e quindi Aggiungi `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="d3bd0-258">Per ulteriori informazioni, vedere la documentazione sulla [distribuzione indipendente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-258">For more information, see [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="d3bd0-259">Usare Team Explorer per controllare il codice in Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="d3bd0-260">Verificare che il codice dell'app sia stato archiviato Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="d3bd0-261">Creare la definizione di compilazione</span><span class="sxs-lookup"><span data-stu-id="d3bd0-261">Create the build definition</span></span>

1. <span data-ttu-id="d3bd0-262">Accedere a Azure Pipelines con un account che può creare una definizione di compilazione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="d3bd0-263">Passare alla pagina **Compila applicazione Web** per il progetto.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="d3bd0-264">In **argomenti**aggiungere il codice **WIN10-x64** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="d3bd0-265">Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="d3bd0-266">Eseguire la compilazione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-266">Run the build.</span></span> <span data-ttu-id="d3bd0-267">Il processo di [compilazione della distribuzione autonoma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e in hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-267">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="d3bd0-268">Usare un agente di compilazione ospitato in Azure</span><span class="sxs-lookup"><span data-stu-id="d3bd0-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="d3bd0-269">L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="d3bd0-270">La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, consentendo un ciclo di sviluppo continuo e senza interruzioni.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="d3bd0-271">Configurare il processo di distribuzione continua (CD)</span><span class="sxs-lookup"><span data-stu-id="d3bd0-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="d3bd0-272">Azure Pipelines e Azure DevOps Services forniscono una pipeline altamente configurabile e gestibile per le versioni in più ambienti quali sviluppo, staging, controllo di qualità (QA) e produzione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="d3bd0-273">Questo processo può includere la richiesta di approvazioni in fasi specifiche del ciclo di vita dell'app.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="d3bd0-274">Creare una definizione della versione</span><span class="sxs-lookup"><span data-stu-id="d3bd0-274">Create release definition</span></span>

<span data-ttu-id="d3bd0-275">La creazione di una definizione di versione è il passaggio finale del processo di compilazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="d3bd0-276">Questa definizione di versione viene usata per creare una versione e distribuire una compilazione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="d3bd0-277">Accedere a Azure Pipelines e passare a **compilazione e versione** per il progetto.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="d3bd0-278">Nella scheda **versioni** selezionare **[+]** , quindi scegliere **Crea definizione di versione**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="d3bd0-279">In **selezionare un modello**scegliere **app Azure distribuzione del servizio**e quindi selezionare **applica**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="d3bd0-280">In **Aggiungi artefatto**selezionare l'app Azure cloud Build dall' **origine (definizione di compilazione)**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="d3bd0-281">Nella scheda **pipeline** selezionare il collegamento **1 fase**, **1 attività** per visualizzare le **attività dell'ambiente**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="d3bd0-282">Nella scheda **attività** immettere Azure come **nome dell'ambiente** e selezionare il AzureCloud Traders-Web EP dall'elenco **sottoscrizione di Azure** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="d3bd0-283">Immettere il **nome del servizio app di Azure**, che si trova `northwindtraders` nella schermata successiva.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="d3bd0-284">Per la fase Agent selezionare **Hosted VS2017** dall'elenco **coda agente** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="d3bd0-285">In **Distribuisci servizio app Azure**selezionare il **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="d3bd0-286">In **Seleziona file o cartella**selezionare **OK** in **percorso**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="d3bd0-287">Salvare tutte le modifiche e tornare alla **pipeline**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="d3bd0-288">Nella scheda **pipeline** selezionare **Aggiungi artefatto**e scegliere **NorthwindCloud Traders-vessel** dall'elenco di **origine (definizione di compilazione)** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="d3bd0-289">In **selezionare un modello**aggiungere un altro ambiente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="d3bd0-290">Scegliere **app Azure distribuzione del servizio** e quindi selezionare **applica**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="d3bd0-291">Immettere `Azure Stack Hub` come **nome dell'ambiente**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="d3bd0-292">Nella scheda **attività** trovare e selezionare Azure stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="d3bd0-293">Dall'elenco **sottoscrizione di Azure** selezionare **AzureStack Traders-vessel EP** per l'endpoint dell'hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="d3bd0-294">Immettere il nome dell'app Web dell'hub Azure Stack come **nome del servizio app**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="d3bd0-295">In **Selezione agente**selezionare **AzureStack-b Douglas Fir** dall'elenco **coda agente** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="d3bd0-296">Per **Distribuisci servizio app Azure**selezionare il **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="d3bd0-297">In **Seleziona file o cartella**selezionare **OK** per **percorso**cartella.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="d3bd0-298">Nella scheda **variabile** individuare la variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="d3bd0-299">Impostare il valore della variabile su **true**e impostarne l'ambito su **Azure stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="d3bd0-300">Nella scheda **pipeline** selezionare l'icona del **trigger di distribuzione continua** per l'artefatto NorthwindCloud Traders-Web e impostare il **trigger di distribuzione continua** su **abilitato**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="d3bd0-301">Eseguire la stessa operazione per l'artefatto **NorthwindCloud Traders-vessel** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="d3bd0-302">Per l'ambiente Azure Stack Hub, selezionare l'icona **condizioni pre-distribuzione** impostare il trigger su **dopo la versione**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="d3bd0-303">Salvare tutte le modifiche.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="d3bd0-304">Alcune impostazioni per le attività di rilascio vengono definite automaticamente come [variabili di ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) quando si crea una definizione di versione da un modello.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-304">Some settings for release tasks are automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="d3bd0-305">Queste impostazioni non possono essere modificate nelle impostazioni dell'attività, ma possono essere modificate negli elementi dell'ambiente padre.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="d3bd0-306">Creare una versione</span><span class="sxs-lookup"><span data-stu-id="d3bd0-306">Create a release</span></span>

1. <span data-ttu-id="d3bd0-307">Nella scheda **pipeline** aprire l'elenco **versione** e selezionare **Crea versione**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="d3bd0-308">Immettere una descrizione per la versione, verificare che siano selezionati gli elementi corretti, quindi selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="d3bd0-309">Dopo alcuni istanti, viene visualizzato un banner che indica che la nuova versione è stata creata e il nome della versione viene visualizzato come collegamento.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="d3bd0-310">Selezionare il collegamento per visualizzare la pagina di riepilogo della versione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="d3bd0-311">Nella pagina di riepilogo della versione vengono visualizzati i dettagli relativi alla versione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="d3bd0-312">Nella schermata seguente per "Release-2", la sezione **environments** Mostra lo **stato di distribuzione** per Azure come "in corso" e lo stato di Azure stack Hub è "succeeded".</span><span class="sxs-lookup"><span data-stu-id="d3bd0-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="d3bd0-313">Quando lo stato della distribuzione per l'ambiente Azure diventa "SUCCEEDed", viene visualizzato un banner che indica che il rilascio è pronto per l'approvazione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="d3bd0-314">Quando una distribuzione è in sospeso o non è riuscita, viene visualizzata un'icona di informazioni blu **(i)** .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="d3bd0-315">Passare il puntatore del mouse sull'icona per visualizzare un popup che contiene il motivo del ritardo o dell'errore.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="d3bd0-316">Altre visualizzazioni, come l'elenco di versioni, visualizzano anche un'icona che indica che l'approvazione è in sospeso.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="d3bd0-317">Il popup per questa icona Mostra il nome dell'ambiente e altri dettagli relativi alla distribuzione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="d3bd0-318">È facile per un amministratore vedere lo stato di avanzamento complessivo dei rilasci e vedere quali versioni sono in attesa di approvazione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="d3bd0-319">Monitorare e tenere traccia delle distribuzioni</span><span class="sxs-lookup"><span data-stu-id="d3bd0-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="d3bd0-320">Nella pagina Riepilogo **versione 2** selezionare **log**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="d3bd0-321">Durante una distribuzione, in questa pagina viene visualizzato il log live dall'agente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="d3bd0-322">Il riquadro sinistro mostra lo stato di ogni operazione nella distribuzione per ogni ambiente.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="d3bd0-323">Selezionare l'icona person nella colonna **azione** per un'approvazione pre-distribuzione o post-distribuzione per verificare chi ha approvato (o rifiutato) la distribuzione e il messaggio fornito.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="d3bd0-324">Al termine della distribuzione, nel riquadro destro viene visualizzato l'intero file di log.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="d3bd0-325">Selezionare un **passaggio** nel riquadro sinistro per visualizzare il file di log per un singolo passaggio, ad esempio **Initialize Job**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="d3bd0-326">La possibilità di visualizzare i singoli log semplifica la traccia e il debug delle parti della distribuzione complessiva.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="d3bd0-327">**Salvare** il file di log per un passaggio o **scaricare tutti i log come zip**.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="d3bd0-328">Aprire la scheda **Riepilogo** per visualizzare le informazioni generali sulla versione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="d3bd0-329">Questa visualizzazione Mostra i dettagli relativi alla compilazione, gli ambienti in cui è stato distribuito, lo stato di distribuzione e altre informazioni sulla versione.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="d3bd0-330">Selezionare un collegamento all'ambiente (**Azure** o **Hub Azure stack**) per visualizzare informazioni sulle distribuzioni esistenti e in sospeso in un ambiente specifico.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="d3bd0-331">Usare queste visualizzazioni come metodo rapido per verificare che la stessa compilazione sia stata distribuita in entrambi gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="d3bd0-332">Aprire l' **app di produzione distribuita** in un browser.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="d3bd0-333">Per il sito web app Azure Services, ad esempio, aprire l'URL `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="d3bd0-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="d3bd0-334">L'integrazione di Azure e Azure Stack Hub offre una soluzione scalabile tra cloud</span><span class="sxs-lookup"><span data-stu-id="d3bd0-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="d3bd0-335">Un servizio multicloud flessibile e affidabile offre sicurezza dei dati, backup e ridondanza, disponibilità coerente e rapida, archiviazione e distribuzione scalabili e routing conforme a geografica.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="d3bd0-336">Questo processo attivato manualmente garantisce un cambio di carico affidabile ed efficiente tra le app Web ospitate e la disponibilità immediata dei dati cruciali.</span><span class="sxs-lookup"><span data-stu-id="d3bd0-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d3bd0-337">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="d3bd0-337">Next steps</span></span>

- <span data-ttu-id="d3bd0-338">Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="d3bd0-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
