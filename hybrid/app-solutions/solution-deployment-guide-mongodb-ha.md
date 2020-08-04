---
title: Distribuire una soluzione MongoDB a disponibilità elevata in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire una soluzione MongoDB a disponibilità elevata in Azure e nell'hub di Azure Stack
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f6064aaa1087a3c0cfc26e09371e81752c777edb
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477270"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Distribuire una soluzione MongoDB a disponibilità elevata in Azure e nell'hub di Azure Stack

Questo articolo illustra una distribuzione automatizzata di un cluster MongoDB a disponibilità elevata di base con un sito di ripristino di emergenza tra due ambienti dell'hub di Azure Stack. Per altre informazioni su MongoDB e sulla disponibilità elevata, vedere [Membri del set di repliche](https://docs.mongodb.com/manual/core/replica-set-members/).

In questa soluzione verrà creato un ambiente di esempio per:

> [!div class="checklist"]
> - Orchestrare una distribuzione tra due hub di Azure Stack.
> - Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.
> - Distribuire un cluster MongoDB a disponibilità elevata di base con un sito di ripristino di emergenza.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Architettura per MongoDB con l'hub di Azure Stack

![architettura MongoDB a disponibilità elevata nell'hub di Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Prerequisiti per MongoDB con l'hub di Azure Stack

- Due sistemi integrati dell'hub di Azure Stack connessi (hub di Azure Stack). Questa distribuzione non funziona nell'Azure Stack Development Kit (ASDK). Per altre informazioni sull'hub di Azure Stack, vedere [Cos'è l'hub di Azure Stack?](https://azure.microsoft.com/products/azure-stack/hub/)
  - Una sottoscrizione tenant in ogni hub di Azure Stack. 
  - **Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub di Azure Stack.**
- Un'entità servizio Azure Active Directory (Azure AD) in possesso delle autorizzazioni per la sottoscrizione tenant in ogni hub di Azure Stack. Potrebbe essere necessario creare due entità servizio se gli hub di Azure Stack vengono distribuiti in tenant di Azure AD diversi. Per informazioni su come creare un'entità servizio per l'hub di Azure Stack, vedere [Usare un'identità dell'app per accedere alle risorse dell'hub di Azure Stack](/azure-stack/user/azure-stack-create-service-principals).
  - **Prendere nota di ID applicazione, segreto client e nome tenant di ogni entità servizio (xxxxx.onmicrosoft.com).**
- Ubuntu 16.04 è stato diffuso a ogni marketplace dell'hub di Azure Stack. Per altre informazioni sulla diffusione del marketplace, vedere [Scaricare gli elementi del marketplace nell'hub di Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- [Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.

## <a name="get-the-docker-image"></a>Ottenere l'immagine Docker

Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.

1. Verificare che Docker per Windows usi i contenitori di Windows.
2. Eseguire il comando seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Distribuire i cluster

1. Dopo aver eseguito correttamente il pull dell'immagine del contenitore, avviare l'immagine.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Avviato il contenitore, gli verrà assegnato un terminale di PowerShell con privilegi elevati. Modificare le directory per ottenere lo script di distribuzione.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Eseguire la distribuzione. Specificare le credenziali e i nomi di risorse, dove necessario. HA si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster a disponibilità elevata. DR si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.

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

4. Digitare `Y` per consentire l'installazione del provider NuGet, che avvierà i moduli del profilo di API "2018-03-01-hybrid" da installare.

5. Le risorse a disponibilità elevata verranno distribuite per prime. Monitorare la distribuzione e attenderne il completamento. Dopo aver ricevuto il messaggio che indica il completamento della distribuzione a disponibilità elevata, è possibile controllare il portale dell'hub di Azure Stack a disponibilità elevata per visualizzare le risorse distribuite.

6. Continuare con la distribuzione delle risorse di ripristino di emergenza e decidere se si vuole abilitare un jumpbox nell'hub di Azure Stack di ripristino di emergenza per interagire con il cluster.

7. Attendere il completamento della distribuzione della risorsa di ripristino di emergenza.

8. Al termine della distribuzione della risorsa di ripristino di emergenza, uscire dal contenitore.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Passaggi successivi

- Se è stata abilitata la VM jumpbox nell'hub di Azure Stack di ripristino di emergenza, è possibile connettersi tramite SSH e interagire con il cluster di MongoDB installando l'interfaccia della riga di comando di Mongo. Per altre informazioni sull'interazione con MongoDB, vedere [Mongo Shell](https://docs.mongodb.com/manual/mongo/).
- Per altre informazioni sulle app per il cloud ibrido, vedere [Soluzioni per il cloud ibrido.](https://aka.ms/azsdevtutorials)
- Modificare il codice in questo esempio in [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
