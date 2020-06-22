---
title: Eseguire il training del modello di Machine Learning al modello perimetrale
description: Informazioni su come eseguire il training del modello di Machine Learning al perimetro con Azure e hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911101"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="86cc7-103">Eseguire il training del modello di Machine Learning al modello perimetrale</span><span class="sxs-lookup"><span data-stu-id="86cc7-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="86cc7-104">Generare modelli di Machine Learning (ML) portatili da dati che esistono solo in locale.</span><span class="sxs-lookup"><span data-stu-id="86cc7-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="86cc7-105">Contesto e problema</span><span class="sxs-lookup"><span data-stu-id="86cc7-105">Context and problem</span></span>

<span data-ttu-id="86cc7-106">Molte organizzazioni vogliono sbloccare informazioni approfondite dai dati locali o legacy usando gli strumenti che i loro data scientist conoscono.</span><span class="sxs-lookup"><span data-stu-id="86cc7-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="86cc7-107">[Azure Machine Learning](/azure/machine-learning/) offre strumenti nativi del cloud per eseguire il training, l'ottimizzazione e la distribuzione di modelli di apprendimento avanzato e ml.</span><span class="sxs-lookup"><span data-stu-id="86cc7-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="86cc7-108">Tuttavia, alcuni dati sono troppo grandi per l'invio al cloud o non possono essere inviati al cloud per motivi normativi.</span><span class="sxs-lookup"><span data-stu-id="86cc7-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="86cc7-109">Con questo modello, i data scientist possono usare Azure Machine Learning per eseguire il training di modelli usando dati e calcolo locali.</span><span class="sxs-lookup"><span data-stu-id="86cc7-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="86cc7-110">Soluzione</span><span class="sxs-lookup"><span data-stu-id="86cc7-110">Solution</span></span>

<span data-ttu-id="86cc7-111">Il training al modello perimetrale usa una macchina virtuale (VM) in esecuzione nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="86cc7-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="86cc7-112">La macchina virtuale viene registrata come destinazione di calcolo in Azure ML, consentendo l'accesso ai dati disponibili solo in locale.</span><span class="sxs-lookup"><span data-stu-id="86cc7-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="86cc7-113">In questo caso i dati vengono archiviati nell'archivio BLOB di Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="86cc7-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="86cc7-114">Una volta eseguito il training del modello, questo viene registrato con Azure ML, in contenitori e aggiunto a un Container Registry di Azure per la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="86cc7-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="86cc7-115">Per questa iterazione del modello, la macchina virtuale di training dell'hub Azure Stack deve essere raggiungibile tramite la rete Internet pubblica.</span><span class="sxs-lookup"><span data-stu-id="86cc7-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="86cc7-116">[![Training del modello ML nell'architettura perimetrale](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="86cc7-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="86cc7-117">Ecco come funziona il modello:</span><span class="sxs-lookup"><span data-stu-id="86cc7-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="86cc7-118">La VM dell'hub Azure Stack viene distribuita e registrata come destinazione di calcolo con Azure ML.</span><span class="sxs-lookup"><span data-stu-id="86cc7-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="86cc7-119">In Azure ML viene creato un esperimento che usa la macchina virtuale Azure Stack Hub come destinazione di calcolo.</span><span class="sxs-lookup"><span data-stu-id="86cc7-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="86cc7-120">Una volta eseguito il training del modello, questo viene registrato e in contenitori.</span><span class="sxs-lookup"><span data-stu-id="86cc7-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="86cc7-121">Il modello può ora essere distribuito in percorsi locali o nel cloud.</span><span class="sxs-lookup"><span data-stu-id="86cc7-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="86cc7-122">Componenti</span><span class="sxs-lookup"><span data-stu-id="86cc7-122">Components</span></span>

<span data-ttu-id="86cc7-123">Questa soluzione USA i componenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="86cc7-123">This solution uses the following components:</span></span>

| <span data-ttu-id="86cc7-124">Livello</span><span class="sxs-lookup"><span data-stu-id="86cc7-124">Layer</span></span> | <span data-ttu-id="86cc7-125">Componente</span><span class="sxs-lookup"><span data-stu-id="86cc7-125">Component</span></span> | <span data-ttu-id="86cc7-126">Descrizione</span><span class="sxs-lookup"><span data-stu-id="86cc7-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="86cc7-127">Azure</span><span class="sxs-lookup"><span data-stu-id="86cc7-127">Azure</span></span> | <span data-ttu-id="86cc7-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="86cc7-128">Azure Machine Learning</span></span> | <span data-ttu-id="86cc7-129">[Azure Machine Learning](/azure/machine-learning/) orchestra il training del modello ml.</span><span class="sxs-lookup"><span data-stu-id="86cc7-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="86cc7-130">Registro Azure Container</span><span class="sxs-lookup"><span data-stu-id="86cc7-130">Azure Container Registry</span></span> | <span data-ttu-id="86cc7-131">Azure ML inserisce il modello in un contenitore e lo archivia in un [container Registry di Azure](/azure/container-registry/) per la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="86cc7-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="86cc7-132">Hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="86cc7-132">Azure Stack Hub</span></span> | <span data-ttu-id="86cc7-133">Servizio app</span><span class="sxs-lookup"><span data-stu-id="86cc7-133">App Service</span></span> | <span data-ttu-id="86cc7-134">[Azure stack Hub con il servizio app](/azure-stack/operator/azure-stack-app-service-overview) fornisce la base per i componenti al perimetro.</span><span class="sxs-lookup"><span data-stu-id="86cc7-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="86cc7-135">Calcolo</span><span class="sxs-lookup"><span data-stu-id="86cc7-135">Compute</span></span> | <span data-ttu-id="86cc7-136">Per eseguire il training del modello ML viene usata una macchina virtuale Azure Stack Hub che esegue Ubuntu con Docker.</span><span class="sxs-lookup"><span data-stu-id="86cc7-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="86cc7-137">Archiviazione</span><span class="sxs-lookup"><span data-stu-id="86cc7-137">Storage</span></span> | <span data-ttu-id="86cc7-138">I dati privati possono essere ospitati nell'archivio BLOB dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="86cc7-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="86cc7-139">Considerazioni e problemi</span><span class="sxs-lookup"><span data-stu-id="86cc7-139">Issues and considerations</span></span>

<span data-ttu-id="86cc7-140">Quando si decide come implementare questa soluzione, tenere presente quanto segue:</span><span class="sxs-lookup"><span data-stu-id="86cc7-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="86cc7-141">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="86cc7-141">Scalability</span></span>

<span data-ttu-id="86cc7-142">Per consentire la scalabilità di questa soluzione, è necessario creare una macchina virtuale di dimensioni appropriate nell'hub Azure Stack per il training.</span><span class="sxs-lookup"><span data-stu-id="86cc7-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="86cc7-143">Disponibilità</span><span class="sxs-lookup"><span data-stu-id="86cc7-143">Availability</span></span>

<span data-ttu-id="86cc7-144">Assicurarsi che gli script di training e la macchina virtuale dell'hub Azure Stack dispongano dell'accesso ai dati locali usati per il training.</span><span class="sxs-lookup"><span data-stu-id="86cc7-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="86cc7-145">Gestione</span><span class="sxs-lookup"><span data-stu-id="86cc7-145">Manageability</span></span>

<span data-ttu-id="86cc7-146">Assicurarsi che i modelli e gli esperimenti siano registrati in modo appropriato, con versione e con tag per evitare confusione durante la distribuzione del modello.</span><span class="sxs-lookup"><span data-stu-id="86cc7-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="86cc7-147">Sicurezza</span><span class="sxs-lookup"><span data-stu-id="86cc7-147">Security</span></span>

<span data-ttu-id="86cc7-148">Questo modello consente ad Azure ML di accedere ai dati sensibili in locale.</span><span class="sxs-lookup"><span data-stu-id="86cc7-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="86cc7-149">Assicurarsi che l'account usato per SSH nella macchina virtuale Azure Stack Hub disponga di una password complessa e che gli script di training non mantengano o caricano i dati nel cloud.</span><span class="sxs-lookup"><span data-stu-id="86cc7-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="86cc7-150">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="86cc7-150">Next steps</span></span>

<span data-ttu-id="86cc7-151">Per ulteriori informazioni sugli argomenti introdotti in questo articolo:</span><span class="sxs-lookup"><span data-stu-id="86cc7-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="86cc7-152">Vedere la [documentazione di Azure Machine Learning](/azure/machine-learning) per una panoramica di ml e argomenti correlati.</span><span class="sxs-lookup"><span data-stu-id="86cc7-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="86cc7-153">Vedere [container Registry di Azure](/azure/container-registry/) per informazioni su come creare, archiviare e gestire immagini per le distribuzioni di contenitori.</span><span class="sxs-lookup"><span data-stu-id="86cc7-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="86cc7-154">Per altre informazioni sul provider di risorse e su come eseguire la distribuzione, vedere [servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-overview) .</span><span class="sxs-lookup"><span data-stu-id="86cc7-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="86cc7-155">Per ulteriori informazioni sulle procedure consigliate e per ottenere risposte alle domande aggiuntive, vedere [considerazioni sulla progettazione di applicazioni ibride](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="86cc7-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="86cc7-156">Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="86cc7-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="86cc7-157">Quando si è pronti per testare l'esempio di soluzione, continuare con il [modello Train ml nella Guida alla distribuzione di Edge](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="86cc7-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="86cc7-158">Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.</span><span class="sxs-lookup"><span data-stu-id="86cc7-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
