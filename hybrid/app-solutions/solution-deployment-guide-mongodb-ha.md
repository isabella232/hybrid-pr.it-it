---
title: Distribuire una soluzione MongoDB a disponibilità elevata in Azure e in hub Azure Stack
description: Informazioni su come distribuire una soluzione MongoDB a disponibilità elevata in Azure e in hub Azure Stack
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b34ba7c10ff5f658d645923ae8b6de2fb2607ccb
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911451"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Distribuire una soluzione MongoDB a disponibilità elevata in Azure e in hub Azure Stack

Questo articolo illustra in modo dettagliato la distribuzione automatica di un cluster MongoDB di base a disponibilità elevata con un sito di ripristino di emergenza in due ambienti Azure Stack Hub. Per altre informazioni su MongoDB e sulla disponibilità elevata, vedere [membri del set di repliche](https://docs.mongodb.com/manual/core/replica-set-members/).

In questa soluzione verrà creato un ambiente di esempio per:

> [!div class="checklist"]
> - Orchestrare una distribuzione tra due hub Azure Stack.
> - Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.
> - Distribuire un cluster MongoDB di base a disponibilità elevata con un sito di ripristino di emergenza.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Architettura per MongoDB con hub Azure Stack

![architettura MongoDB a disponibilità elevata nell'hub Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Prerequisiti per MongoDB con hub Azure Stack

- Due sistemi integrati di Azure Stack hub collegati (hub Azure Stack). Questa distribuzione non funziona nel Azure Stack Development Kit (Gabriele). Per altre informazioni sull'hub Azure Stack, vedere [che cos'è Azure stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)
  - Una sottoscrizione tenant in ogni hub Azure Stack. 
  - **Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub Azure Stack.**
- Un'entità servizio Azure Active Directory (Azure AD) che dispone delle autorizzazioni per la sottoscrizione tenant in ogni hub Azure Stack. Potrebbe essere necessario creare due entità servizio se gli hub Azure Stack vengono distribuiti in tenant Azure AD diversi. Per informazioni su come creare un'entità servizio per Azure Stack Hub, vedere [usare un'identità dell'app per accedere alle risorse dell'hub Azure stack](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).
  - **Prendere nota dell'ID applicazione di ogni entità servizio, del segreto client e del nome del tenant (xxxxx.onmicrosoft.com).**
- Ubuntu 16,04 è stato diffuso a ogni Marketplace dell'hub Azure Stack. Per altre informazioni sulla diffusione del Marketplace, vedere [scaricare gli elementi del Marketplace nell'Hub Azure stack](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- [Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.

## <a name="get-the-docker-image"></a>Ottenere l'immagine Docker

Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.

1. Assicurarsi che Docker per Windows usi i contenitori di Windows.
2. Eseguire il comando seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Distribuire i cluster

1. Una volta eseguito il pull dell'immagine del contenitore, avviare l'immagine.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Una volta avviato il contenitore, verrà assegnato un terminale di PowerShell con privilegi elevati nel contenitore. Modificare le directory per ottenere lo script di distribuzione.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Eseguire la distribuzione. Fornire le credenziali e i nomi delle risorse laddove necessario. HA si riferisce all'hub Azure Stack in cui verrà distribuito il cluster a disponibilità elevata. Il ripristino di emergenza fa riferimento all'hub Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. Digitare `Y` per consentire l'installazione del provider NuGet, che consente di avviare i moduli del profilo API "2018-03-01-Hybrid" da installare.

5. Le risorse a disponibilità elevata vengono distribuite per prime. Monitorare la distribuzione e attenderne il completamento. Una volta che il messaggio indica che la distribuzione a disponibilità elevata è stata completata, è possibile controllare il portale dell'hub Azure Stack HA per visualizzare le risorse distribuite.

6. Continuare con la distribuzione delle risorse di ripristino di emergenza e decidere se si vuole abilitare una Jump box nell'hub Azure Stack di ripristino di emergenza per interagire con il cluster.

7. Attendere il completamento della distribuzione delle risorse di ripristino di emergenza.

8. Al termine della distribuzione delle risorse di ripristino di emergenza, uscire dal contenitore.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Passaggi successivi

- Se è stata abilitata la macchina virtuale jump box nell'hub Azure Stack di ripristino di emergenza, è possibile connettersi tramite SSH e interagire con il cluster MongoDB installando l'interfaccia della riga di comando di Mongo. Per ulteriori informazioni sull'interazione con MongoDB, vedere [la shell di Mongo](https://docs.mongodb.com/manual/mongo/).
- Per altre informazioni sulle app cloud ibride, vedere [soluzioni cloud ibride.](https://aka.ms/azsdevtutorials)
- Modificare il codice in questo esempio su [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
