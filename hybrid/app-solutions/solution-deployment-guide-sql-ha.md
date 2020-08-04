---
title: Distribuire un gruppo di disponibilità di SQL Server 2016 in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire un gruppo di disponibilità di SQL Server 2016 in Azure e nell'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 85b859457b9b54a973c5fc23329b927212b60a07
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477083"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Distribuire un gruppo di disponibilità di SQL Server 2016 in Azure e nell'hub di Azure Stack

Questo articolo illustra una distribuzione automatizzata di un cluster di SQL Server 2016 Enterprise a disponibilità elevata di base con un sito di ripristino di emergenza asincrono tra due ambienti dell'hub di Azure Stack. Per altre informazioni su SQL Server 2016 e sulla disponibilità elevata, vedere [Gruppi di disponibilità Always On: una soluzione per la disponibilità elevata e il ripristino di emergenza](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

In questa soluzione si compilerà un ambiente di esempio per:

> [!div class="checklist"]
> - Orchestrare una distribuzione tra due hub di Azure Stack.
> - Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.
> - Distribuire un cluster di SQL Server 2016 Enterprise a disponibilità elevata di base con un sito di ripristino di emergenza.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure, che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="architecture-for-sql-server-2016"></a>Architettura di SQL Server 2016

![Disponibilità elevata di SQL Server 2016 nell'hub di Azure Stack](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Prerequisiti di SQL Server 2016

- Due sistemi integrati dell'hub di Azure Stack connessi (hub di Azure Stack). Questa distribuzione non funziona nell'Azure Stack Development Kit (ASDK). Per altre informazioni sull'hub di Azure Stack, vedere la [panoramica di Azure Stack](https://azure.microsoft.com/overview/azure-stack/).
- Una sottoscrizione tenant in ogni hub di Azure Stack.
  - **Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub di Azure Stack.**
- Un'entità servizio Azure Active Directory (Azure AD) in possesso delle autorizzazioni per la sottoscrizione tenant in ogni hub di Azure Stack. Potrebbe essere necessario creare due entità servizio se gli hub di Azure Stack vengono distribuiti in tenant di Azure AD diversi. Per informazioni su come creare un'entità servizio per l'hub di Azure Stack, vedere [Creare entità servizio per consentire alle app l'accesso alle risorse dell'hub di Azure Stack](/azure-stack/user/azure-stack-create-service-principals).
  - **Prendere nota di ID applicazione, segreto client e nome tenant di ogni entità servizio (xxxxx.onmicrosoft.com).**
- SQL Server 2016 Enterprise è stato diffuso in ogni marketplace dell'hub di Azure Stack. Per altre informazioni sulla diffusione nel marketplace, vedere [Scaricare gli elementi del marketplace nell'hub di Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Assicurarsi che l'organizzazione disponga delle licenze SQL appropriate.**
- [Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.

## <a name="get-the-docker-image"></a>Ottenere l'immagine Docker

Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.

1. Verificare che Docker per Windows usi i contenitori di Windows.
2. Eseguire lo script seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Distribuire il gruppo di disponibilità

1. Dopo aver eseguito correttamente il pull dell'immagine del contenitore, avviare l'immagine.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Avviato il contenitore, gli verrà assegnato un terminale di PowerShell con privilegi elevati. Modificare le directory per ottenere lo script di distribuzione.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Eseguire la distribuzione. Specificare le credenziali e i nomi delle risorse, dove necessario. HA si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster a disponibilità elevata. DR si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

5. Attendere il completamento della distribuzione delle risorse.

6. Al completamento della distribuzione della risorsa di ripristino di emergenza, uscire dal contenitore.

      ```powershell
      exit
      ```

7. Esaminare la distribuzione visualizzando le risorse nel portale di ogni hub di Azure Stack. Connettersi a una delle istanze di SQL nell'ambiente a disponibilità elevata ed esaminare il gruppo di disponibilità tramite SQL Server Management Studio (SSMS).

    ![Disponibilità elevata SQL - SQL Server 2016](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Passaggi successivi

- Usare SQL Server Management Studio per eseguire manualmente il failover del cluster. Vedere [Eseguire un failover manuale forzato di un gruppo di disponibilità Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Altre informazioni sulle app cloud ibride. Vedere [Soluzioni per il cloud ibrido.](https://aka.ms/azsdevtutorials)
- Usare i dati personalizzati o modificare il codice in questo esempio in [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
