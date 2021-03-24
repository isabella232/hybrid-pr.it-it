---
title: Distribuire un'app con scalabilità tra cloud in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire un'app con scalabilità tra cloud in Azure e nell'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895415"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="a85e4-103">Distribuire un'app con scalabilità tra cloud usando Azure e l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="a85e4-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a85e4-104">Informazioni su come creare una soluzione tra cloud per offrire un processo attivato manualmente per passare da un'app Web ospitata nell'hub di Azure Stack a un'app Web ospitata in Azure con scalabilità automatica tramite Gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="a85e4-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="a85e4-105">Questo processo garantisce un'utilità cloud flessibile e scalabile in fase di caricamento.</span><span class="sxs-lookup"><span data-stu-id="a85e4-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="a85e4-106">Con questo modello il tenant può non essere pronto a eseguire l'app nel cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="a85e4-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="a85e4-107">Tuttavia, per l'azienda può non essere economicamente fattibile mantenere la capacità richiesta nell'ambiente locale per gestire i picchi di domanda per l'app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="a85e4-108">Il tenant può sfruttare l'elasticità del cloud pubblico con la soluzione locale in uso.</span><span class="sxs-lookup"><span data-stu-id="a85e4-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="a85e4-109">In questa soluzione si compilerà un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="a85e4-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="a85e4-110">Creare un'app Web a più nodi.</span><span class="sxs-lookup"><span data-stu-id="a85e4-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="a85e4-111">Configurare e gestire il processo di distribuzione continua.</span><span class="sxs-lookup"><span data-stu-id="a85e4-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="a85e4-112">Pubblicare l'app Web nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="a85e4-113">Creare una versione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-113">Create a release.</span></span>
> - <span data-ttu-id="a85e4-114">Imparare a monitorare e tenere traccia delle distribuzioni.</span><span class="sxs-lookup"><span data-stu-id="a85e4-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="a85e4-115">![diagramma delle colonne ibride](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="a85e4-115">![hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="a85e4-116">L'hub di Microsoft Azure Stack è un'estensione di Azure</span><span class="sxs-lookup"><span data-stu-id="a85e4-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="a85e4-117">che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="a85e4-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="a85e4-118">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="a85e4-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="a85e4-119">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="a85e4-120">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="a85e4-120">Prerequisites</span></span>

- <span data-ttu-id="a85e4-121">Sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="a85e4-121">Azure subscription.</span></span> <span data-ttu-id="a85e4-122">Prima di iniziare, se necessario, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).</span><span class="sxs-lookup"><span data-stu-id="a85e4-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="a85e4-123">Un sistema integrato dell'hub di Azure Stack o una distribuzione di Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="a85e4-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="a85e4-124">Per informazioni sull'installazione dell'hub di Azure Stack, vedere le [istruzioni di installazione di ASDK](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="a85e4-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install).</span></span>
  - <span data-ttu-id="a85e4-125">Per uno script di automazione post-distribuzione di ASDK, vedere [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="a85e4-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="a85e4-126">Per completare questa installazione possono essere necessarie alcune ore.</span><span class="sxs-lookup"><span data-stu-id="a85e4-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="a85e4-127">Distribuire i servizi PaaS del [servizio app](/azure-stack/operator/azure-stack-app-service-deploy) nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="a85e4-128">[Creare piani o offerte](/azure-stack/operator/service-plan-offer-subscription-overview) nell'ambiente dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="a85e4-129">[Creare una sottoscrizione tenant](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) nell'ambiente dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="a85e4-130">Creare un'app Web all'interno della sottoscrizione tenant.</span><span class="sxs-lookup"><span data-stu-id="a85e4-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="a85e4-131">Prendere nota dell'URL della nuova app Web per usarlo in seguito.</span><span class="sxs-lookup"><span data-stu-id="a85e4-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="a85e4-132">Distribuire la macchina virtuale (VM) di Azure Pipelines nella sottoscrizione tenant.</span><span class="sxs-lookup"><span data-stu-id="a85e4-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="a85e4-133">È necessaria una macchina virtuale Windows Server 2016 con .NET 3.5.</span><span class="sxs-lookup"><span data-stu-id="a85e4-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="a85e4-134">Questa macchina virtuale verrà compilata nella sottoscrizione tenant per l'hub di Azure Stack come agente di compilazione privato.</span><span class="sxs-lookup"><span data-stu-id="a85e4-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="a85e4-135">[Windows Server 2016 con immagine VM di SQL 2017](/azure-stack/operator/azure-stack-add-vm-image) è disponibile nel Marketplace dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="a85e4-136">Se questa immagine non è disponibile, usare un operatore dell'hub di Azure Stack per assicurarsi che venga aggiunta all'ambiente.</span><span class="sxs-lookup"><span data-stu-id="a85e4-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="a85e4-137">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="a85e4-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="a85e4-138">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="a85e4-138">Scalability</span></span>

<span data-ttu-id="a85e4-139">Il componente chiave della scalabilità tra cloud è la possibilità di rendere immediatamente disponibile la scalabilità su richiesta tra l'infrastruttura cloud pubblica e quella locale, offrendo un servizio coerente e affidabile.</span><span class="sxs-lookup"><span data-stu-id="a85e4-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="a85e4-140">Disponibilità</span><span class="sxs-lookup"><span data-stu-id="a85e4-140">Availability</span></span>

<span data-ttu-id="a85e4-141">Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata verificando la configurazione hardware locale e la distribuzione del software.</span><span class="sxs-lookup"><span data-stu-id="a85e4-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="a85e4-142">Gestione</span><span class="sxs-lookup"><span data-stu-id="a85e4-142">Manageability</span></span>

<span data-ttu-id="a85e4-143">La soluzione tra cloud garantisce una gestione senza problemi e un'interfaccia familiare tra gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="a85e4-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="a85e4-144">Si consiglia di usare PowerShell per la gestione multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="a85e4-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="a85e4-145">Scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="a85e4-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="a85e4-146">Ottenere un dominio personalizzato e configurare il DNS</span><span class="sxs-lookup"><span data-stu-id="a85e4-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="a85e4-147">Aggiornare il file della zona DNS per il dominio.</span><span class="sxs-lookup"><span data-stu-id="a85e4-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="a85e4-148">Azure AD verifica quindi la proprietà del nome di dominio personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a85e4-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="a85e4-149">Usare il [DNS di Azure](/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Microsoft 365/esterni in Azure oppure aggiungere la voce DNS in un [registrar DNS diverso](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="a85e4-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="a85e4-150">Registrare un dominio personalizzato con un registrar pubblico.</span><span class="sxs-lookup"><span data-stu-id="a85e4-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="a85e4-151">Accedere al registrar per il dominio.</span><span class="sxs-lookup"><span data-stu-id="a85e4-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="a85e4-152">Per eseguire gli aggiornamenti al DNS potrebbe essere necessario un amministratore approvato.</span><span class="sxs-lookup"><span data-stu-id="a85e4-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="a85e4-153">Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS specificata da Azure AD.</span><span class="sxs-lookup"><span data-stu-id="a85e4-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="a85e4-154">La voce DNS non influirà sul routing della posta elettronica né sui comportamenti di hosting Web.</span><span class="sxs-lookup"><span data-stu-id="a85e4-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="a85e4-155">Creare un'app Web multinodo predefinita nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="a85e4-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="a85e4-156">Configurare l'integrazione continua e la distribuzione continua (CI/CD) ibride per distribuire le app Web in Azure e nell'hub di Azure Stack e per eseguire il push automatico delle modifiche in entrambi i cloud.</span><span class="sxs-lookup"><span data-stu-id="a85e4-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="a85e4-157">Sono necessari l'hub di Azure Stack con immagini diffuse appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="a85e4-158">Per altre informazioni, vedere i [prerequisiti per la distribuzione del servizio app nell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started) nella documentazione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="a85e4-159">Aggiungere codice ad Azure Repos</span><span class="sxs-lookup"><span data-stu-id="a85e4-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="a85e4-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="a85e4-160">Azure Repos</span></span>

1. <span data-ttu-id="a85e4-161">Accedere ad Azure Repos con un account provvisto di diritti di creazione progetti per Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="a85e4-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="a85e4-162">L'integrazione continua e la distribuzione continua (CI/CD) ibride possono essere applicate sia al codice dell'app sia al codice dell'infrastruttura.</span><span class="sxs-lookup"><span data-stu-id="a85e4-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="a85e4-163">Usare [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) sia per lo sviluppo cloud privato che per quello ospitato.</span><span class="sxs-lookup"><span data-stu-id="a85e4-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Eseguire la connessione a un progetto in Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="a85e4-165">**Clonare il repository** creando e aprendo l'app Web predefinita.</span><span class="sxs-lookup"><span data-stu-id="a85e4-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonare il repository nell'app Web di Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="a85e4-167">Creare una distribuzione di app Web autonoma per i servizi app in entrambi i cloud</span><span class="sxs-lookup"><span data-stu-id="a85e4-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="a85e4-168">Modificare il file **WebApplication.csproj**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="a85e4-169">Selezionare `Runtimeidentifier` e aggiungere `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="a85e4-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="a85e4-170">Vedere la documentazione relativa alla [distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="a85e4-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Modificare un file di progetto di app Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="a85e4-172">Archiviare il codice in Azure Repos usando Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="a85e4-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="a85e4-173">Verificare che il codice dell'applicazione sia stato archiviato in Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="a85e4-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="a85e4-174">Creare la definizione di compilazione</span><span class="sxs-lookup"><span data-stu-id="a85e4-174">Create the build definition</span></span>

1. <span data-ttu-id="a85e4-175">Accedere ad Azure Pipelines per assicurarsi di poter creare definizioni di compilazione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="a85e4-176">Aggiungere il codice **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="a85e4-177">Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.</span><span class="sxs-lookup"><span data-stu-id="a85e4-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Aggiungere codice all'app Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="a85e4-179">Eseguire la compilazione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-179">Run the build.</span></span> <span data-ttu-id="a85e4-180">Il processo di [compilazione della distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che vengono eseguiti in Azure e nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="a85e4-181">Usare un agente ospitato di Azure</span><span class="sxs-lookup"><span data-stu-id="a85e4-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="a85e4-182">L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web.</span><span class="sxs-lookup"><span data-stu-id="a85e4-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="a85e4-183">La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, quindi il ciclo di sviluppo è continuo e ininterrotto.</span><span class="sxs-lookup"><span data-stu-id="a85e4-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="a85e4-184">Gestire e configurare il processo di distribuzione continua</span><span class="sxs-lookup"><span data-stu-id="a85e4-184">Manage and configure the CD process</span></span>

<span data-ttu-id="a85e4-185">Azure Pipelines e Azure DevOps Services offrono una pipeline altamente configurabile e facilmente gestibile per i rilasci in più ambienti, ad esempio gli ambienti di sviluppo, gestione temporanea, controllo qualità e produzione, e include la richiesta di approvazioni in fasi specifiche.</span><span class="sxs-lookup"><span data-stu-id="a85e4-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="a85e4-186">Creare una definizione della versione</span><span class="sxs-lookup"><span data-stu-id="a85e4-186">Create release definition</span></span>

1. <span data-ttu-id="a85e4-187">Selezionare il pulsante con il **segno più** per aggiungere una nuova versione nella scheda **Releases** (Rilasci) della sezione **Build and Release** (Compilazione e versione) di Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="a85e4-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Creare una definizione di versione](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="a85e4-189">Applicare il modello di distribuzione del servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="a85e4-189">Apply the Azure App Service Deployment template.</span></span>

   ![Applicare il modello di distribuzione del servizio app di Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="a85e4-191">In **Aggiungi artefatto** aggiungere l'artefatto corrispondente all'app della build del cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="a85e4-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Aggiungere l'artefatto alla build del cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="a85e4-193">Nella scheda Pipeline selezionare il collegamento **Phase, Task** (Fase, Attività) dell'ambiente e impostare i valori di ambiente del cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="a85e4-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Impostare i valori di ambiente del cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="a85e4-195">Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="a85e4-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="a85e4-197">Come **nome del servizio app** specificare il nome del servizio app di Azure richiesto.</span><span class="sxs-lookup"><span data-stu-id="a85e4-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Impostare il nome del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="a85e4-199">Immettere "Hosted VS2017" in **Agent queue** (Coda agente) per l'ambiente ospitato del cloud di Azure.</span><span class="sxs-lookup"><span data-stu-id="a85e4-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Impostare la coda agente per l'ambiente ospitato del cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="a85e4-201">Nel menu Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare l'opzione **Package or Folder** (Pacchetto o cartella) per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="a85e4-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="a85e4-202">Selezionare **OK** per **folder location** (Percorso cartella).</span><span class="sxs-lookup"><span data-stu-id="a85e4-202">Select **OK** to **folder location**.</span></span>
  
      ![Selezionare il pacchetto o la cartella per l'ambiente del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Finestra di dialogo di selezione cartelle 1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="a85e4-205">Salvare tutte le modifiche e tornare alla **pipeline di versione**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Salvare le modifiche nella pipeline di versione](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="a85e4-207">Aggiungere un nuovo artefatto selezionando la build per l'app hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Aggiungere un nuovo artefatto per l'app hub di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="a85e4-209">Aggiungere un altro ambiente applicando la distribuzione del servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="a85e4-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Aggiungere l'ambiente alla distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="a85e4-211">Assegnare al nuovo ambiente il nome "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="a85e4-211">Name the new environment "Azure Stack".</span></span>

    ![Assegnare un nome all'ambiente nella distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="a85e4-213">Individuare l'ambiente Azure Stack nella scheda **Tasks** (Attività).</span><span class="sxs-lookup"><span data-stu-id="a85e4-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Ambiente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="a85e4-215">Selezionare la sottoscrizione per l'endpoint di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Selezionare la sottoscrizione per l'endpoint di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="a85e4-217">Impostare il nome dell'app Web Azure Stack come nome del servizio app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="a85e4-218">![Impostare il nome dell'app Web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="a85e4-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="a85e4-219">Selezionare l'agente di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-219">Select the Azure Stack agent.</span></span>

    ![Selezionare l'agente di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="a85e4-221">Nella sezione Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="a85e4-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="a85e4-222">Selezionare **OK** per il percorso della cartella.</span><span class="sxs-lookup"><span data-stu-id="a85e4-222">Select **OK** to folder location.</span></span>

    ![Selezionare la cartella per la distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Finestra di dialogo di selezione cartelle 2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="a85e4-225">Nella scheda Variable (Variabile) aggiungere una variabile con nome `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, impostare il valore come **true** e Azure Stack come ambito.</span><span class="sxs-lookup"><span data-stu-id="a85e4-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Aggiungere una variabile alla distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="a85e4-227">Selezionare l'icona del trigger di distribuzione **Continuous** (Continua) in entrambi gli artefatti e abilitare il trigger di distribuzione **Continues** (Continua).</span><span class="sxs-lookup"><span data-stu-id="a85e4-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selezionare il trigger di distribuzione continua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="a85e4-229">Selezionare l'icona delle condizioni **Pre-deployment** (Pre-distribuzione) nell'ambiente Azure Stack e impostare il trigger su **Dopo la versione**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Selezionare le condizioni di pre-distribuzione](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="a85e4-231">Salvare tutte le modifiche.</span><span class="sxs-lookup"><span data-stu-id="a85e4-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="a85e4-232">Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) durante la creazione di una definizione di versione da un modello.</span><span class="sxs-lookup"><span data-stu-id="a85e4-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="a85e4-233">Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. Per modificarle è invece necessario selezionare l'elemento dell'ambiente padre.</span><span class="sxs-lookup"><span data-stu-id="a85e4-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="a85e4-234">Pubblicare nell'hub di Azure Stack con Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a85e4-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="a85e4-235">Creando gli endpoint, una build di Azure DevOps Services può distribuire le app del servizio di Azure nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="a85e4-236">Azure Pipelines si connette all'agente di compilazione, che si connette all'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="a85e4-237">Accedere ad Azure DevOps Services e passare alla pagina delle impostazioni dell'app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="a85e4-238">Su **Impostazioni**, selezionare **Sicurezza**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="a85e4-239">In **Gruppi VSTS** selezionare **Creatori di endpoint**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="a85e4-240">Nella scheda **Membri** fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="a85e4-241">In **Aggiungi utenti e gruppi** immettere un nome utente e selezionare l'utente dall'elenco di utenti.</span><span class="sxs-lookup"><span data-stu-id="a85e4-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="a85e4-242">Selezionare **Save changes** (Salva modifiche).</span><span class="sxs-lookup"><span data-stu-id="a85e4-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="a85e4-243">Nell'elenco **Gruppi VSTS** selezionare **Amministratori endpoint**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="a85e4-244">Nella scheda **Membri** fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="a85e4-245">In **Aggiungi utenti e gruppi** immettere un nome utente e selezionare l'utente dall'elenco di utenti.</span><span class="sxs-lookup"><span data-stu-id="a85e4-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="a85e4-246">Selezionare **Save changes** (Salva modifiche).</span><span class="sxs-lookup"><span data-stu-id="a85e4-246">Select **Save changes**.</span></span>

<span data-ttu-id="a85e4-247">Poiché ora sono disponibili le informazioni sugli endpoint, la connessione da Azure Pipelines all'hub di Azure Stack è pronta per l'uso.</span><span class="sxs-lookup"><span data-stu-id="a85e4-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="a85e4-248">L'agente di compilazione nell'hub di Azure Stack riceve le istruzioni da Azure Pipelines e quindi specifica le informazioni sugli endpoint per la comunicazione con l'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="a85e4-249">Sviluppare la build dell'app</span><span class="sxs-lookup"><span data-stu-id="a85e4-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="a85e4-250">Sono necessari l'hub di Azure Stack con immagini diffuse appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="a85e4-251">Per altre informazioni, vedere [Prerequisiti per la distribuzione del servizio app nell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="a85e4-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

<span data-ttu-id="a85e4-252">Usare i [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) come codice dell'app Web da Azure Repos per la distribuzione in entrambi i cloud.</span><span class="sxs-lookup"><span data-stu-id="a85e4-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="a85e4-253">Aggiungere codice a un progetto di Azure Repos</span><span class="sxs-lookup"><span data-stu-id="a85e4-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="a85e4-254">Accedere ad Azure Repos con un account con diritti di creazione progetti per l'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="a85e4-255">**Clonare il repository** creando e aprendo l'app Web predefinita.</span><span class="sxs-lookup"><span data-stu-id="a85e4-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="a85e4-256">Creare una distribuzione di app Web autonoma per i servizi app in entrambi i cloud</span><span class="sxs-lookup"><span data-stu-id="a85e4-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="a85e4-257">Modificare il file **WebApplication.csproj**: Selezionare `Runtimeidentifier` e quindi aggiungere `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="a85e4-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="a85e4-258">Per altre informazioni, vedere la documentazione relativa alla [distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="a85e4-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="a85e4-259">Usare Team Explorer per archiviare il codice in Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="a85e4-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="a85e4-260">Verificare che il codice dell'app sia stato archiviato in Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="a85e4-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="a85e4-261">Creare la definizione di compilazione</span><span class="sxs-lookup"><span data-stu-id="a85e4-261">Create the build definition</span></span>

1. <span data-ttu-id="a85e4-262">Accedere ad Azure Pipelines con un account che può creare una definizione di compilazione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="a85e4-263">Passare alla pagina di **compilazione dell'applicazione Web** per il progetto.</span><span class="sxs-lookup"><span data-stu-id="a85e4-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="a85e4-264">In **Argomenti** aggiungere il codice **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="a85e4-265">Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.</span><span class="sxs-lookup"><span data-stu-id="a85e4-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="a85e4-266">Eseguire la compilazione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-266">Run the build.</span></span> <span data-ttu-id="a85e4-267">Il processo di [compilazione della distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="a85e4-268">Usare un agente di compilazione ospitato di Azure</span><span class="sxs-lookup"><span data-stu-id="a85e4-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="a85e4-269">L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web.</span><span class="sxs-lookup"><span data-stu-id="a85e4-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="a85e4-270">La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, quindi il ciclo di sviluppo è continuo e ininterrotto.</span><span class="sxs-lookup"><span data-stu-id="a85e4-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="a85e4-271">Configurare il processo di distribuzione continua</span><span class="sxs-lookup"><span data-stu-id="a85e4-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="a85e4-272">Azure Pipelines e Azure DevOps Services offrono una pipeline altamente configurabile e facilmente gestibile per i rilasci in più ambienti, ad esempio gli ambienti di sviluppo, gestione temporanea, controllo qualità e produzione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="a85e4-273">Questo processo può includere la richiesta di approvazioni in fasi specifiche del ciclo di vita dell'app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="a85e4-274">Creare una definizione della versione</span><span class="sxs-lookup"><span data-stu-id="a85e4-274">Create release definition</span></span>

<span data-ttu-id="a85e4-275">La creazione di una definizione della versione è il passaggio finale del processo di compilazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="a85e4-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="a85e4-276">La definizione della versione viene usata per creare una versione e distribuire una build.</span><span class="sxs-lookup"><span data-stu-id="a85e4-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="a85e4-277">Accedere ad Azure Pipelines e passare a **Compilazione e versione** per il progetto.</span><span class="sxs-lookup"><span data-stu-id="a85e4-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="a85e4-278">Nella scheda **Versioni** selezionare **[+]** e quindi scegliere **Crea definizione di versione**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="a85e4-279">In **Seleziona un modello** scegliere **Distribuzione Servizio app di Azure** e quindi selezionare **Applica**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="a85e4-280">In **Aggiungi artefatto** selezionare l'app della build del cloud di Azure da **Origine (definizione di compilazione)** .</span><span class="sxs-lookup"><span data-stu-id="a85e4-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="a85e4-281">Nella scheda **Pipeline** selezionare il collegamento **fase 1**, **attività 1** a **Visualizza le attività dell'ambiente**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="a85e4-282">Nella scheda **Attività** immettere Azure come **nome dell'ambiente** e selezionare AzureCloud Traders-Web EP dall'elenco delle **sottoscrizioni di Azure**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="a85e4-283">Immettere il **nome del servizio app di Azure**, che è `northwindtraders` nella successiva acquisizione di schermata.</span><span class="sxs-lookup"><span data-stu-id="a85e4-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="a85e4-284">Per la fase Agente selezionare **Hosted VS2017** dall'elenco **Coda agente**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="a85e4-285">In **Deploy Azure App Service** (Distribuisci servizio app di Azure) selezionare l'opzione **Package or folder** (Pacchetto o cartella) per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="a85e4-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="a85e4-286">In **Seleziona file o cartella** selezionare **OK** per **Percorso**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="a85e4-287">Salvare tutte le modifiche e tornare a **Pipeline**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="a85e4-288">Nella scheda **Pipeline** selezionare **Aggiungi artefatto** e scegliere **NorthwindCloud Traders-Vessel** dall'elenco **Origine (definizione di compilazione)** .</span><span class="sxs-lookup"><span data-stu-id="a85e4-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="a85e4-289">Aggiungere un altro ambiente in **Seleziona un modello**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="a85e4-290">Scegliere **Distribuzione Servizio app di Azure** e quindi selezionare **Applica**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="a85e4-291">Immettere `Azure Stack Hub` come **nome dell'ambiente**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="a85e4-292">Nella scheda **Attività** trovare e selezionare Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a85e4-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="a85e4-293">Nell'elenco delle **sottoscrizioni di Azure** selezionare **AzureStack Traders-Vessel EP** per l'endpoint dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="a85e4-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="a85e4-294">Immettere il nome dell'app Web Azure Stack Hub come **nome del servizio app**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="a85e4-295">In **Selezione agente** scegliere **AzureStack -b Douglas Fir** dall'elenco **Coda agente**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="a85e4-296">Per **Deploy Azure App Service** (Distribuisci servizio app di Azure) selezionare il **pacchetto o la cartella** validi per l'ambiente.</span><span class="sxs-lookup"><span data-stu-id="a85e4-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="a85e4-297">In **Seleziona file o cartella** selezionare **OK** per la cartella **Percorso**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="a85e4-298">Nella scheda **Variabile** individuare la variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span><span class="sxs-lookup"><span data-stu-id="a85e4-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="a85e4-299">Impostare il valore della variabile su **true** e impostare l'ambito su **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="a85e4-300">Nella scheda **Pipeline** selezionare l'icona del **trigger di distribuzione continua** per l'artefatto NorthwindCloud Traders-Web e impostare il **trigger di distribuzione continua** su **Abilitato**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="a85e4-301">Eseguire la stessa operazione per l'artefatto **NorthwindCloud Traders-Vessel**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="a85e4-302">Per l'ambiente Azure Stack Hub, selezionare l'icona delle **condizioni di pre-distribuzione** e impostare il trigger su **Dopo la versione**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="a85e4-303">Salvare tutte le modifiche.</span><span class="sxs-lookup"><span data-stu-id="a85e4-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="a85e4-304">Alcune impostazioni per le attività di rilascio vengono definite automaticamente come [variabili di ambiente](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) durante la creazione di una definizione di versione da un modello.</span><span class="sxs-lookup"><span data-stu-id="a85e4-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="a85e4-305">Queste impostazioni non possono essere modificate nelle impostazioni delle attività, ma si possono modificare negli elementi dell'ambiente padre.</span><span class="sxs-lookup"><span data-stu-id="a85e4-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="a85e4-306">Creare una versione</span><span class="sxs-lookup"><span data-stu-id="a85e4-306">Create a release</span></span>

1. <span data-ttu-id="a85e4-307">Nella scheda **Pipeline** aprire l'elenco **Versione** e selezionare **Crea versione**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="a85e4-308">Immettere una descrizione per la versione, verificare che siano selezionati gli artefatti corretti, quindi selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="a85e4-309">Dopo alcuni istanti viene visualizzato un banner che indica che la nuova versione è stata creata e il nome della versione viene visualizzato come collegamento.</span><span class="sxs-lookup"><span data-stu-id="a85e4-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="a85e4-310">Selezionare il collegamento per visualizzare la pagina di riepilogo della versione,</span><span class="sxs-lookup"><span data-stu-id="a85e4-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="a85e4-311">che contiene i dettagli relativi alla versione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="a85e4-312">Nella seguente acquisizione di schermata per "Release-2", nella sezione **Ambienti** lo **stato della distribuzione** per Azure è "IN PROGRESS" (in corso) e lo stato per l'hub di Azure Stack è "SUCCEEDED" (riuscita).</span><span class="sxs-lookup"><span data-stu-id="a85e4-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="a85e4-313">Quando lo stato della distribuzione per l'ambiente di Azure diventa "SUCCEEDED", viene visualizzato un banner che indica che la versione è pronta per l'approvazione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="a85e4-314">Quando una distribuzione è in sospeso o non riesce, viene visualizzata un'icona blu con la **(i)** di Informazioni.</span><span class="sxs-lookup"><span data-stu-id="a85e4-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="a85e4-315">Passare il mouse sull'icona per visualizzare un elemento popup che contiene il motivo del ritardo o dell'errore.</span><span class="sxs-lookup"><span data-stu-id="a85e4-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="a85e4-316">Altre visualizzazioni, ad esempio l'elenco delle versioni, contengono anche un'icona che indica che l'approvazione è in sospeso.</span><span class="sxs-lookup"><span data-stu-id="a85e4-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="a85e4-317">L'elemento popup per questa icona specifica il nome dell'ambiente e altri dettagli relativi alla distribuzione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="a85e4-318">È facile per un amministratore vedere lo stato di avanzamento complessivo delle versioni e quali versioni sono in attesa di approvazione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="a85e4-319">Monitorare le distribuzioni</span><span class="sxs-lookup"><span data-stu-id="a85e4-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="a85e4-320">Nella pagina di riepilogo di **Release-2** selezionare **Log**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="a85e4-321">Durante una distribuzione, in questa pagina è visualizzato il log attivo dell'agente.</span><span class="sxs-lookup"><span data-stu-id="a85e4-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="a85e4-322">Nel riquadro sinistro è riportato lo stato di ogni operazione della distribuzione per ogni ambiente.</span><span class="sxs-lookup"><span data-stu-id="a85e4-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="a85e4-323">Selezionare l'icona della persona nella colonna **Azione** per un'approvazione pre-distribuzione o post-distribuzione per verificare chi ha approvato (o rifiutato) la distribuzione e visualizzare il relativo messaggio.</span><span class="sxs-lookup"><span data-stu-id="a85e4-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="a85e4-324">Al termine della distribuzione, nel riquadro destro viene visualizzato l'intero file di log.</span><span class="sxs-lookup"><span data-stu-id="a85e4-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="a85e4-325">Selezionare un **passaggio** nel riquadro sinistro per visualizzare il file di log per quel passaggio specifico, ad esempio l'**inizializzazione del processo**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="a85e4-326">La possibilità di visualizzare i singoli log semplifica il monitoraggio e il debug delle parti della distribuzione complessiva.</span><span class="sxs-lookup"><span data-stu-id="a85e4-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="a85e4-327">**Salvare** il file di log per un passaggio o **scaricare tutti i log in un file zip**.</span><span class="sxs-lookup"><span data-stu-id="a85e4-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="a85e4-328">Aprire la scheda **Riepilogo** per visualizzare le informazioni generali sulla versione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="a85e4-329">Questa vista contiene i dettagli relativi alla build, gli ambienti in cui è stata distribuita, lo stato della distribuzione e altre informazioni sulla versione.</span><span class="sxs-lookup"><span data-stu-id="a85e4-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="a85e4-330">Selezionare un collegamento all'ambiente (**Azure** o **hub di Azure Stack**) per visualizzare informazioni sulle distribuzioni esistenti e in sospeso in un ambiente specifico.</span><span class="sxs-lookup"><span data-stu-id="a85e4-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="a85e4-331">Usare queste viste come metodo rapido per verificare che la stessa build sia stata distribuita in entrambi gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="a85e4-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="a85e4-332">Aprire l'**app di produzione distribuita** in un browser.</span><span class="sxs-lookup"><span data-stu-id="a85e4-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="a85e4-333">Ad esempio, per il sito Web Servizi app di Azure aprire l'URL `https://[your-app-name\].azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="a85e4-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="a85e4-334">L'integrazione di Azure e dell'hub di Azure Stack offre una soluzione scalabile tra cloud</span><span class="sxs-lookup"><span data-stu-id="a85e4-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="a85e4-335">Un servizio multi-cloud flessibile e affidabile offre sicurezza dei dati, backup e ridondanza, disponibilità costante e rapida, archiviazione e distribuzione scalabili e routing conforme all'area geografica.</span><span class="sxs-lookup"><span data-stu-id="a85e4-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="a85e4-336">Questo processo attivato manualmente garantisce un passaggio affidabile ed efficiente del carico tra le app Web ospitate e la disponibilità immediata dei dati cruciali.</span><span class="sxs-lookup"><span data-stu-id="a85e4-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a85e4-337">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="a85e4-337">Next steps</span></span>

- <span data-ttu-id="a85e4-338">Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="a85e4-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
