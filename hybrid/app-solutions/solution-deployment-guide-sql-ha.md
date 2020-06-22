---
title: Distribuire un gruppo di disponibilità SQL Server 2016 in Azure e nell'hub Azure Stack
description: Informazioni su come distribuire un gruppo di disponibilità SQL Server 2016 in Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911045"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Distribuire un gruppo di disponibilità SQL Server 2016 in Azure e nell'hub Azure Stack

Questo articolo illustra in modo dettagliato la distribuzione automatica di un cluster Enterprise a disponibilità elevata SQL Server 2016 con un sito di ripristino di emergenza asincrono in due ambienti hub Azure Stack. Per altre informazioni su SQL Server 2016 e sulla disponibilità elevata, vedere [gruppi di disponibilità always on: una soluzione di disponibilità elevata e ripristino di emergenza](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

In questa soluzione verrà compilato un ambiente di esempio per:

> [!div class="checklist"]
> - Orchestrare una distribuzione tra due hub Azure Stack.
> - Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.
> - Distribuire un cluster aziendale di base a disponibilità elevata SQL Server 2016 con un sito di ripristino di emergenza.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

## <a name="architecture-for-sql-server-2016"></a>Architettura per SQL Server 2016

![SQL Server 2016 Hub Azure Stack a disponibilità elevata SQL](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Prerequisiti per SQL Server 2016

- Due sistemi integrati di Azure Stack hub collegati (hub Azure Stack). Questa distribuzione non funziona nel Azure Stack Development Kit (Gabriele). Per altre informazioni sull'hub Azure Stack, vedere la [Panoramica di Azure stack](https://azure.microsoft.com/overview/azure-stack/).
- Una sottoscrizione tenant in ogni hub Azure Stack.
  - **Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub Azure Stack.**
- Un'entità servizio Azure Active Directory (Azure AD) che dispone delle autorizzazioni per la sottoscrizione tenant in ogni hub Azure Stack. Potrebbe essere necessario creare due entità servizio se gli hub Azure Stack vengono distribuiti in tenant Azure AD diversi. Per informazioni su come creare un'entità servizio per Azure Stack Hub, vedere [creare entità servizio per consentire alle app di accedere alle risorse dell'hub Azure stack](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).
  - **Prendere nota dell'ID applicazione di ogni entità servizio, del segreto client e del nome del tenant (xxxxx.onmicrosoft.com).**
- SQL Server 2016 Enterprise syndicated per ogni Marketplace di Azure Stack Hub. Per altre informazioni sulla diffusione del Marketplace, vedere [scaricare gli elementi del Marketplace nell'Hub Azure stack](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Assicurarsi che l'organizzazione disponga delle licenze SQL appropriate.**
- [Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.

## <a name="get-the-docker-image"></a>Ottenere l'immagine Docker

Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.

1. Assicurarsi che Docker per Windows usi i contenitori di Windows.
2. Eseguire lo script seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Distribuire il gruppo di disponibilità

1. Una volta eseguito il pull dell'immagine del contenitore, avviare l'immagine.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Una volta avviato il contenitore, verrà assegnato un terminale di PowerShell con privilegi elevati nel contenitore. Modificare le directory per ottenere lo script di distribuzione.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Eseguire la distribuzione. Fornire le credenziali e i nomi delle risorse laddove necessario. HA si riferisce all'hub Azure Stack in cui verrà distribuito il cluster a disponibilità elevata. Il ripristino di emergenza fa riferimento all'hub Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.

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

4. Digitare `Y` per consentire l'installazione del provider NuGet, che consente di avviare i moduli del profilo API "2018-03-01-Hybrid" da installare.

5. Attendere il completamento della distribuzione delle risorse.

6. Una volta completata la distribuzione delle risorse di ripristino di emergenza, uscire dal contenitore.

      ```powershell
      exit
      ```

7. Esaminare la distribuzione visualizzando le risorse nel portale di ogni hub Azure Stack. Connettersi a una delle istanze di SQL nell'ambiente a disponibilità elevata e ispezionare il gruppo di disponibilità tramite SQL Server Management Studio (SSMS).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Passaggi successivi

- Usare SQL Server Management Studio per eseguire manualmente il failover del cluster. Vedere [eseguire un failover manuale forzato di un gruppo di disponibilità always on (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Scopri di più sulle app cloud ibride. Vedere [soluzioni cloud ibride.](https://aka.ms/azsdevtutorials)
- Usare i propri dati o modificare il codice in questo esempio su [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
