---
title: Distribuire la soluzione di rilevamento del footfall basata sull'intelligenza artificiale in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire una soluzione di rilevamento del footfall basata sull'intelligenza artificiale per analizzare il traffico dei visitatori nei punti vendita al dettaglio usando Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895365"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="d9a48-103">Distribuire una soluzione di rilevamento del footfall basata sull'intelligenza artificiale usando Azure e l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="d9a48-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d9a48-104">Questo articolo descrive come distribuire una soluzione basata sull'intelligenza artificiale che genera informazioni dettagliate da azioni reali usando Azure, l'hub di Azure Stack e il Custom Vision AI Dev Kit.</span><span class="sxs-lookup"><span data-stu-id="d9a48-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="d9a48-105">In questa soluzione si apprenderà come:</span><span class="sxs-lookup"><span data-stu-id="d9a48-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d9a48-106">Distribuire Cloud Native Application Bundle (CNAB) perimetrali.</span><span class="sxs-lookup"><span data-stu-id="d9a48-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="d9a48-107">Distribuire un'app che si estende oltre i limiti del cloud.</span><span class="sxs-lookup"><span data-stu-id="d9a48-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="d9a48-108">Usare il Custom Vision AI Dev Kit per l'inferenza perimetrale.</span><span class="sxs-lookup"><span data-stu-id="d9a48-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="d9a48-109">![Diagramma dei concetti sulle app ibride](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d9a48-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d9a48-110">L'hub di Microsoft Azure Stack è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d9a48-111">L'hub di Azure Stack offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="d9a48-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d9a48-112">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="d9a48-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d9a48-113">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="d9a48-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d9a48-114">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="d9a48-114">Prerequisites</span></span>

<span data-ttu-id="d9a48-115">Prima di iniziare a usare questa guida alla distribuzione:</span><span class="sxs-lookup"><span data-stu-id="d9a48-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="d9a48-116">Rivedere l'argomento [Modello di rilevamento del footfall](pattern-retail-footfall-detection.md).</span><span class="sxs-lookup"><span data-stu-id="d9a48-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="d9a48-117">Ottenere un accesso utente ad Azure Stack Development Kit (ASDK) o un'istanza di sistema integrato dell'hub di Azure Stack, con i requisiti seguenti:</span><span class="sxs-lookup"><span data-stu-id="d9a48-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="d9a48-118">Installazione del [servizio app di Azure in un provider di risorse dell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-overview).</span><span class="sxs-lookup"><span data-stu-id="d9a48-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview) installed.</span></span> <span data-ttu-id="d9a48-119">È necessario disporre dell'accesso operatore all'istanza dell'hub di Azure Stack o dell'accesso amministratore per poter eseguire l'installazione.</span><span class="sxs-lookup"><span data-stu-id="d9a48-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="d9a48-120">Sottoscrizione a un'offerta che offre il servizio app e la quota di archiviazione.</span><span class="sxs-lookup"><span data-stu-id="d9a48-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="d9a48-121">Per creare un'offerta, è necessario disporre dell'accesso operatore.</span><span class="sxs-lookup"><span data-stu-id="d9a48-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="d9a48-122">Ottenere l'accesso a una sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="d9a48-123">Se non si dispone di una sottoscrizione di Azure, iscriversi per creare un [account di prova gratuito](https://azure.microsoft.com/free/) prima di iniziare.</span><span class="sxs-lookup"><span data-stu-id="d9a48-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="d9a48-124">Creare due entità servizio nella directory:</span><span class="sxs-lookup"><span data-stu-id="d9a48-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="d9a48-125">Una configurata per usare le risorse di Azure, con accesso all'ambito della sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="d9a48-126">Una configurata per usare le risorse dell'hub di Azure Stack, con accesso all'ambito della sottoscrizione dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d9a48-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="d9a48-127">Per altre informazioni sulla creazione di entità servizio e sull'autorizzazione dell'accesso, vedere [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="d9a48-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals).</span></span> <span data-ttu-id="d9a48-128">Per usare l'interfaccia della riga di comando di Azure, vedere [Creare un'entità servizio di Azure con l'interfaccia della riga di comando di Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span><span class="sxs-lookup"><span data-stu-id="d9a48-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="d9a48-129">Distribuire Servizi cognitivi di Azure in Azure o nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d9a48-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="d9a48-130">Leggere prima [altre informazioni su Servizi cognitivi](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="d9a48-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="d9a48-131">Visitare quindi [Distribuire Servizi cognitivi di Azure nell'hub di Azure Stack](/azure-stack/user/azure-stack-solution-template-cognitive-services) per distribuire Servizi cognitivi nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d9a48-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="d9a48-132">È prima necessario iscriversi per accedere all'anteprima.</span><span class="sxs-lookup"><span data-stu-id="d9a48-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="d9a48-133">Clonare o scaricare una versione del Custom Vision AI Dev Kit non configurata di Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="d9a48-134">Per informazioni dettagliate, vedere [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="d9a48-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="d9a48-135">Creare un account Power BI.</span><span class="sxs-lookup"><span data-stu-id="d9a48-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="d9a48-136">Chiave di sottoscrizione per l'API Viso di Servizi cognitivi di Azure e URL dell'endpoint.</span><span class="sxs-lookup"><span data-stu-id="d9a48-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="d9a48-137">È possibile ottenere sia la chiave sia l'URL nella versione di [valutazione di Servizi cognitivi](https://azure.microsoft.com/try/cognitive-services/?api=face-api).</span><span class="sxs-lookup"><span data-stu-id="d9a48-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="d9a48-138">In alternativa, seguire le istruzioni riportate in [Creare un account di Servizi cognitivi](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="d9a48-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="d9a48-139">Installare le risorse di sviluppo seguenti:</span><span class="sxs-lookup"><span data-stu-id="d9a48-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="d9a48-140">Interfaccia della riga di comando di Azure 2.0</span><span class="sxs-lookup"><span data-stu-id="d9a48-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [<span data-ttu-id="d9a48-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="d9a48-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="d9a48-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="d9a48-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="d9a48-143">Porter consente di distribuire app cloud usando i manifesti di bundle CNAB disponibili.</span><span class="sxs-lookup"><span data-stu-id="d9a48-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="d9a48-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="d9a48-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="d9a48-145">Azure IoT Tools per Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="d9a48-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="d9a48-146">Estensione Python per Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="d9a48-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="d9a48-147">Python</span><span class="sxs-lookup"><span data-stu-id="d9a48-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="d9a48-148">Distribuire l'app cloud ibrida</span><span class="sxs-lookup"><span data-stu-id="d9a48-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="d9a48-149">Usare l'interfaccia della riga di comando di Porter per generare un set di credenziali, quindi distribuire l'app cloud.</span><span class="sxs-lookup"><span data-stu-id="d9a48-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="d9a48-150">Clonare o scaricare il codice di esempio della soluzione da https://github.com/azure-samples/azure-intelligent-edge-patterns.</span><span class="sxs-lookup"><span data-stu-id="d9a48-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="d9a48-151">Porter genererà un set di credenziali che consentono di automatizzare la distribuzione dell'app.</span><span class="sxs-lookup"><span data-stu-id="d9a48-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="d9a48-152">Prima di eseguire il comando di generazione delle credenziali, assicurarsi che siano disponibili gli elementi seguenti:</span><span class="sxs-lookup"><span data-stu-id="d9a48-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="d9a48-153">Entità servizio per l'accesso alle risorse di Azure, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.</span><span class="sxs-lookup"><span data-stu-id="d9a48-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="d9a48-154">ID sottoscrizione della sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="d9a48-155">Entità servizio per l'accesso alle risorse dell'hub di Azure Stack, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.</span><span class="sxs-lookup"><span data-stu-id="d9a48-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="d9a48-156">ID sottoscrizione della sottoscrizione dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d9a48-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="d9a48-157">Chiave per l'API Viso di Servizi cognitivi di Azure e URL dell'endpoint della risorsa.</span><span class="sxs-lookup"><span data-stu-id="d9a48-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="d9a48-158">Eseguire il processo di generazione delle credenziali di Porter e seguire le istruzioni:</span><span class="sxs-lookup"><span data-stu-id="d9a48-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="d9a48-159">Per l'esecuzione, Porter richiede anche un set di parametri.</span><span class="sxs-lookup"><span data-stu-id="d9a48-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="d9a48-160">Creare un file di testo per i parametri e immettere le coppie nome-valore seguenti.</span><span class="sxs-lookup"><span data-stu-id="d9a48-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="d9a48-161">Rivolgersi all'amministratore dell'hub di Azure Stack per richiedere assistenza su qualsiasi valore necessario.</span><span class="sxs-lookup"><span data-stu-id="d9a48-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="d9a48-162">Il valore `resource suffix` viene usato per assicurare che le risorse della distribuzione abbiano nomi univoci in Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="d9a48-163">Deve essere una stringa univoca di lettere e numeri e non deve superare gli 8 caratteri di lunghezza.</span><span class="sxs-lookup"><span data-stu-id="d9a48-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="d9a48-164">Salvare il file di testo e prendere nota del relativo percorso.</span><span class="sxs-lookup"><span data-stu-id="d9a48-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="d9a48-165">A questo punto è possibile distribuire l'app cloud ibrida usando Porter.</span><span class="sxs-lookup"><span data-stu-id="d9a48-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="d9a48-166">Eseguire il comando di installazione e osservare come le risorse vengono distribuite in Azure e nell'hub di Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="d9a48-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="d9a48-167">Al termine della distribuzione, prendere nota dei valori seguenti:</span><span class="sxs-lookup"><span data-stu-id="d9a48-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="d9a48-168">Stringa di connessione della fotocamera.</span><span class="sxs-lookup"><span data-stu-id="d9a48-168">The camera's connection string.</span></span>
    - <span data-ttu-id="d9a48-169">Stringa di connessione dell'account di archiviazione dell'immagine.</span><span class="sxs-lookup"><span data-stu-id="d9a48-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="d9a48-170">Nomi dei gruppi di risorse.</span><span class="sxs-lookup"><span data-stu-id="d9a48-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="d9a48-171">Preparare il Custom Vision AI DevKit</span><span class="sxs-lookup"><span data-stu-id="d9a48-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="d9a48-172">A questo punto configurare il Custom Vision AI DevKit come illustrato nell'[avvio rapido di Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="d9a48-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="d9a48-173">È anche possibile configurare e testare la fotocamera, usando la stringa di connessione ottenuta nel passaggio precedente.</span><span class="sxs-lookup"><span data-stu-id="d9a48-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="d9a48-174">Distribuire l'app della fotocamera</span><span class="sxs-lookup"><span data-stu-id="d9a48-174">Deploy the camera app</span></span>

<span data-ttu-id="d9a48-175">Usare l'interfaccia della riga di comando di Porter per generare un set di credenziali, quindi distribuire l'app della fotocamera.</span><span class="sxs-lookup"><span data-stu-id="d9a48-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="d9a48-176">Porter genererà un set di credenziali che consentono di automatizzare la distribuzione dell'app.</span><span class="sxs-lookup"><span data-stu-id="d9a48-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="d9a48-177">Prima di eseguire il comando di generazione delle credenziali, assicurarsi che siano disponibili gli elementi seguenti:</span><span class="sxs-lookup"><span data-stu-id="d9a48-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="d9a48-178">Entità servizio per l'accesso alle risorse di Azure, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.</span><span class="sxs-lookup"><span data-stu-id="d9a48-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="d9a48-179">ID sottoscrizione della sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="d9a48-180">Stringa di connessione dell'account di archiviazione dell'immagine ottenuta durante la distribuzione dell'app cloud.</span><span class="sxs-lookup"><span data-stu-id="d9a48-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="d9a48-181">Eseguire il processo di generazione delle credenziali di Porter e seguire le istruzioni:</span><span class="sxs-lookup"><span data-stu-id="d9a48-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="d9a48-182">Per l'esecuzione, Porter richiede anche un set di parametri.</span><span class="sxs-lookup"><span data-stu-id="d9a48-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="d9a48-183">Creare un file di testo per i parametri e immettere il testo seguente.</span><span class="sxs-lookup"><span data-stu-id="d9a48-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="d9a48-184">Rivolgersi all'amministratore dell'hub di Azure Stack se non sono noti alcuni dei valori richiesti.</span><span class="sxs-lookup"><span data-stu-id="d9a48-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="d9a48-185">Il valore `deployment suffix` viene usato per assicurare che le risorse della distribuzione abbiano nomi univoci in Azure.</span><span class="sxs-lookup"><span data-stu-id="d9a48-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="d9a48-186">Deve essere una stringa univoca di lettere e numeri e non deve superare gli 8 caratteri di lunghezza.</span><span class="sxs-lookup"><span data-stu-id="d9a48-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="d9a48-187">Salvare il file di testo e prendere nota del relativo percorso.</span><span class="sxs-lookup"><span data-stu-id="d9a48-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="d9a48-188">A questo punto è possibile distribuire l'app della fotocamera usando Porter.</span><span class="sxs-lookup"><span data-stu-id="d9a48-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="d9a48-189">Eseguire il comando di installazione e osservare come viene distribuito il servizio IoT Edge.</span><span class="sxs-lookup"><span data-stu-id="d9a48-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="d9a48-190">Verificare che la distribuzione della fotocamera sia completata visualizzando il feed della fotocamera in `https://<camera-ip>:3000/`, dove `<camara-ip>` corrisponde all'indirizzo IP della fotocamera.</span><span class="sxs-lookup"><span data-stu-id="d9a48-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="d9a48-191">Questo passaggio può richiedere fino a 10 minuti.</span><span class="sxs-lookup"><span data-stu-id="d9a48-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="d9a48-192">Configurare Analisi di flusso di Azure</span><span class="sxs-lookup"><span data-stu-id="d9a48-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="d9a48-193">Ora che i dati vengono trasmessi ad Analisi di flusso di Azure dalla fotocamera, è necessario creare manualmente un'autorizzazione perché possano comunicare con Power BI.</span><span class="sxs-lookup"><span data-stu-id="d9a48-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="d9a48-194">Nel portale di Azure aprire **Tutte le risorse**, quindi selezionare il processo *process-footfall\[suffissoutente\]* .</span><span class="sxs-lookup"><span data-stu-id="d9a48-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="d9a48-195">Nella sezione **Topologia processo** del riquadro del processo di Analisi di flusso selezionare l'opzione **Output**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="d9a48-196">Selezionare il sink di output **traffic-output**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="d9a48-197">Selezionare **Rinnova autorizzazione** e accedere all'account di Power BI.</span><span class="sxs-lookup"><span data-stu-id="d9a48-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Richiesta di rinnovo dell'autorizzazione in Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="d9a48-199">Salvare le impostazioni di output.</span><span class="sxs-lookup"><span data-stu-id="d9a48-199">Save the output settings.</span></span>

6. <span data-ttu-id="d9a48-200">Passare al riquadro **Panoramica** e selezionare **Avvia** per iniziare a inviare i dati a Power BI.</span><span class="sxs-lookup"><span data-stu-id="d9a48-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="d9a48-201">Selezionare **Ora** come orario di avvio per l'output del processo e selezionare **Avvia**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="d9a48-202">È possibile visualizzare lo stato del processo nella barra di notifica.</span><span class="sxs-lookup"><span data-stu-id="d9a48-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="d9a48-203">Creare un dashboard di Power BI</span><span class="sxs-lookup"><span data-stu-id="d9a48-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="d9a48-204">Al termine del processo, passare a [Power BI](https://powerbi.com/) e accedere con l'account aziendale o dell'istituto di istruzione.</span><span class="sxs-lookup"><span data-stu-id="d9a48-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="d9a48-205">Se la query del processo di Analisi di flusso restituisce risultati, il set di dati *footfall-dataset* creato è presente nella scheda **Set di dati**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="d9a48-206">Nell'area di lavoro di Power BI selezionare **+ Crea** per creare un nuovo dashboard denominato *Footfall Analysis.*</span><span class="sxs-lookup"><span data-stu-id="d9a48-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="d9a48-207">Nella parte superiore della finestra selezionare **Aggiungi riquadro**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="d9a48-208">Selezionare quindi **Dati in streaming personalizzati** e **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="d9a48-209">Scegliere **footfall-dataset** in **Set di dati personali**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="d9a48-210">Selezionare **Scheda** nell'elenco a discesa **Tipo di visualizzazione** e aggiungere **age** a **Campi**.</span><span class="sxs-lookup"><span data-stu-id="d9a48-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="d9a48-211">Selezionare **Avanti** per immettere un nome per il riquadro e quindi selezionare **Applica** per creare il riquadro.</span><span class="sxs-lookup"><span data-stu-id="d9a48-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="d9a48-212">È possibile aggiungere altri campi e schede in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="d9a48-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="d9a48-213">Testare la soluzione</span><span class="sxs-lookup"><span data-stu-id="d9a48-213">Test Your Solution</span></span>

<span data-ttu-id="d9a48-214">Osservare come i dati nelle schede create in Power BI cambiano a seconda delle diverse persone che passano di fronte alla fotocamera.</span><span class="sxs-lookup"><span data-stu-id="d9a48-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="d9a48-215">È possibile che le inferenze siano visualizzate fino a 20 secondi dopo essere state registrate.</span><span class="sxs-lookup"><span data-stu-id="d9a48-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="d9a48-216">Rimuovere la soluzione</span><span class="sxs-lookup"><span data-stu-id="d9a48-216">Remove Your Solution</span></span>

<span data-ttu-id="d9a48-217">Se si vuole rimuovere la soluzione, eseguire i comandi seguenti tramite Porter, usando gli stessi file di parametri creati per la distribuzione:</span><span class="sxs-lookup"><span data-stu-id="d9a48-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="d9a48-218">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="d9a48-218">Next steps</span></span>

- <span data-ttu-id="d9a48-219">Altre informazioni sulle [considerazioni per la progettazione di app ibride](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="d9a48-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="d9a48-220">Esaminare e proporre miglioramenti del [codice di questo esempio in GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="d9a48-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
