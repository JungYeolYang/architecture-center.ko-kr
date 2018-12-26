---
title: 기본 웹앱 응용 프로그램
titleSuffix: Azure Reference Architectures
description: Azure에서 실행되는 기본 웹 애플리케이션에 권장하는 아키텍처입니다.
author: MikeWasson
ms.date: 12/12/2017
ms.custom: seodec18
ms.openlocfilehash: 17750a57835f017d13eb205a7b4e821a5834b741
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120257"
---
# <a name="run-a-basic-web-application-in-azure"></a><span data-ttu-id="5af48-103">Azure의 기본 웹 애플리케이션 실행</span><span class="sxs-lookup"><span data-stu-id="5af48-103">Run a basic web application in Azure</span></span>

<span data-ttu-id="5af48-104">이 참조 아키텍처는 [Azure App Service][app-service] 및 [Azure SQL Database][sql-db]를 사용하는 웹 애플리케이션에 관한 검증된 사례를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-104">This reference architecture shows proven practices for a web application that uses [Azure App Service][app-service] and [Azure SQL Database][sql-db].</span></span> <span data-ttu-id="5af48-105">[**이 솔루션을 배포합니다**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="5af48-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Azure의 기본 웹 애플리케이션을 위한 참조 아키텍처](./images/basic-web-app.png)

<span data-ttu-id="5af48-107">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="5af48-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="5af48-108">아키텍처</span><span class="sxs-lookup"><span data-stu-id="5af48-108">Architecture</span></span>

> [!NOTE]
> <span data-ttu-id="5af48-109">이 아키텍처는 응용 프로그램 개발에 초점을 두지 않으며 특정 응용 프로그램 프레임워크를 가정하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-109">This architecture does not focus on application development, and does not assume any particular application framework.</span></span> <span data-ttu-id="5af48-110">다양한 Azure 서비스가 어떻게 연결되는지 이해하는 것이 목표입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-110">The goal is to understand how various Azure services fit together.</span></span>
>

<span data-ttu-id="5af48-111">이 아키텍처의 구성 요소는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-111">The architecture has the following components:</span></span>

- <span data-ttu-id="5af48-112">**리소스 그룹**.</span><span class="sxs-lookup"><span data-stu-id="5af48-112">**Resource group**.</span></span> <span data-ttu-id="5af48-113">[리소스 그룹](/azure/azure-resource-manager/resource-group-overview)은 Azure 리소스에 대한 논리적 컨테이너입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-113">A [resource group](/azure/azure-resource-manager/resource-group-overview) is a logical container for Azure resources.</span></span>

- <span data-ttu-id="5af48-114">**App Service 앱**.</span><span class="sxs-lookup"><span data-stu-id="5af48-114">**App Service app**.</span></span> <span data-ttu-id="5af48-115">[Azure App Service][app-service]는 클라우드 응용 프로그램을 만들고 배포하기 위한 완전히 관리되는 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-115">[Azure App Service][app-service] is a fully managed platform for creating and deploying cloud applications.</span></span>

- <span data-ttu-id="5af48-116">**App Service 계획**.</span><span class="sxs-lookup"><span data-stu-id="5af48-116">**App Service plan**.</span></span> <span data-ttu-id="5af48-117">[App Service 계획][app-service-plans]은 앱을 호스트하는 관리되는 VM(가상 머신)을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-117">An [App Service plan][app-service-plans] provides the managed virtual machines (VMs) that host your app.</span></span> <span data-ttu-id="5af48-118">계획과 연결된 모든 앱은 같은 VM 인스턴스에서 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-118">All apps associated with a plan run on the same VM instances.</span></span>

- <span data-ttu-id="5af48-119">**배포 슬롯**.</span><span class="sxs-lookup"><span data-stu-id="5af48-119">**Deployment slots**.</span></span>  <span data-ttu-id="5af48-120">[배포 슬롯][deployment-slots]을 사용하면 배포를 스테이징한 다음 프로덕션 배포로 교환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-120">A [deployment slot][deployment-slots] lets you stage a deployment and then swap it with the production deployment.</span></span> <span data-ttu-id="5af48-121">따라서 프로덕션으로 바로 배포하지 않아도 됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-121">That way, you avoid deploying directly into production.</span></span> <span data-ttu-id="5af48-122">관련 권장 사항은 [관리 효율성](#manageability-considerations) 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-122">See the [Manageability](#manageability-considerations) section for specific recommendations.</span></span>

- <span data-ttu-id="5af48-123">**IP 주소**.</span><span class="sxs-lookup"><span data-stu-id="5af48-123">**IP address**.</span></span> <span data-ttu-id="5af48-124">App Service 앱에는 공용 IP 주소와 도메인 이름이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-124">The App Service app has a public IP address and a domain name.</span></span> <span data-ttu-id="5af48-125">도메인 이름은 `contoso.azurewebsites.net`와 같이 `azurewebsites.net`의 하위 도메인입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-125">The domain name is a subdomain of `azurewebsites.net`, such as `contoso.azurewebsites.net`.</span></span>

- <span data-ttu-id="5af48-126">**Azure DNS**.</span><span class="sxs-lookup"><span data-stu-id="5af48-126">**Azure DNS**.</span></span> <span data-ttu-id="5af48-127">[Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-127">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="5af48-128">Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-128">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span> <span data-ttu-id="5af48-129">사용자 지정 도메인 이름(예: `contoso.com`)을 사용하려면 사용자 지정 도메인 이름을 IP 주소에 매핑하는 DNS 레코드를 작성합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-129">To use a custom domain name (such as `contoso.com`) create DNS records that map the custom domain name to the IP address.</span></span> <span data-ttu-id="5af48-130">자세한 내용은 [Azure App Service에서 사용자 지정 도메인 이름 구성][custom-domain-name]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-130">For more information, see [Configure a custom domain name in Azure App Service][custom-domain-name].</span></span>

- <span data-ttu-id="5af48-131">**Azure SQL Database**.</span><span class="sxs-lookup"><span data-stu-id="5af48-131">**Azure SQL Database**.</span></span> <span data-ttu-id="5af48-132">[SQL Database][sql-db]는 클라우드에서 실행되는 관계형 DaaS(Database-as-a-Service)입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-132">[SQL Database][sql-db] is a relational database-as-a-service in the cloud.</span></span> <span data-ttu-id="5af48-133">SQL Database는 해당 코드 베이스를 Microsoft SQL Server 데이터베이스 엔진과 공유합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-133">SQL Database shares its code base with the Microsoft SQL Server database engine.</span></span> <span data-ttu-id="5af48-134">애플리케이션 요구 사항에 따라 [Azure Database for MySQL](/azure/mysql) 또는 [Azure Database for PostgreSQL](/azure/postgresql)을 사용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-134">Depending on your application requirements, you can also use [Azure Database for MySQL](/azure/mysql) or [Azure Database for PostgreSQL](/azure/postgresql).</span></span> <span data-ttu-id="5af48-135">이 기능은 각각 오픈 소스 MySQL Server 및 Postgres 데이터베이스 엔진에 기반하여 완전히 관리되는 데이터베이스 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-135">These are fully managed database services, based on the open source MySQL Server and Postgres database engines, respectively.</span></span>

- <span data-ttu-id="5af48-136">**논리 서버**.</span><span class="sxs-lookup"><span data-stu-id="5af48-136">**Logical server**.</span></span> <span data-ttu-id="5af48-137">Azure SQL Database에서 논리 서버는 데이터베이스를 호스트합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-137">In Azure SQL Database, a logical server hosts your databases.</span></span> <span data-ttu-id="5af48-138">논리 서버당 데이터베이스를 여럿 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-138">You can create multiple databases per logical server.</span></span>

- <span data-ttu-id="5af48-139">**Azure Storage**.</span><span class="sxs-lookup"><span data-stu-id="5af48-139">**Azure Storage**.</span></span> <span data-ttu-id="5af48-140">메타데이터를 저장할 Blob 컨테이너가 있는 Azure Storage 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-140">Create an Azure storage account with a blob container to store diagnostic logs.</span></span>

- <span data-ttu-id="5af48-141">**Azure AD(Azure Active Directory)**.</span><span class="sxs-lookup"><span data-stu-id="5af48-141">**Azure Active Directory (Azure AD)**.</span></span> <span data-ttu-id="5af48-142">인증을 위해 Azure AD 또는 다른 ID 공급자를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-142">Use Azure AD or another identity provider for authentication.</span></span>

## <a name="recommendations"></a><span data-ttu-id="5af48-143">권장 사항</span><span class="sxs-lookup"><span data-stu-id="5af48-143">Recommendations</span></span>

<span data-ttu-id="5af48-144">개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-144">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="5af48-145">이 섹션의 권장 사항을 시작점으로 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-145">Use the recommendations in this section as a starting point.</span></span>

### <a name="app-service-plan"></a><span data-ttu-id="5af48-146">App Service 계획</span><span class="sxs-lookup"><span data-stu-id="5af48-146">App Service plan</span></span>

<span data-ttu-id="5af48-147">규모 확장, 자동 크기 조정 및 SSL(Secure Sockets Layer)을 지원하는 표준 또는 프리미엄 계층을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-147">Use the Standard or Premium tiers, because they support scale out, autoscale, and secure sockets layer (SSL).</span></span> <span data-ttu-id="5af48-148">각 계층은 코어 및 메모리 수가 다른 여러 *인스턴스 크기*를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-148">Each tier supports several *instance sizes* that differ by number of cores and memory.</span></span> <span data-ttu-id="5af48-149">계획을 만든 후 계층이나 인스턴스 크기를 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-149">You can change the tier or instance size after you create a plan.</span></span> <span data-ttu-id="5af48-150">App Service 계획에 대한 자세한 내용은 [App Service 가격][app-service-plans-tiers]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-150">For more information about App Service plans, see [App Service Pricing][app-service-plans-tiers].</span></span>

<span data-ttu-id="5af48-151">앱이 중지된 경우에도 App Service 계획의 인스턴스에 대해 요금이 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-151">You are charged for the instances in the App Service plan, even if the app is stopped.</span></span> <span data-ttu-id="5af48-152">사용하지 않는 계획(예: 테스트 배포)은 삭제하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-152">Make sure to delete plans that you aren't using (for example, test deployments).</span></span>

### <a name="sql-database"></a><span data-ttu-id="5af48-153">SQL Database</span><span class="sxs-lookup"><span data-stu-id="5af48-153">SQL Database</span></span>

<span data-ttu-id="5af48-154">SQL Database의 [V12 버전][sql-db-v12]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-154">Use the [V12 version][sql-db-v12] of SQL Database.</span></span> <span data-ttu-id="5af48-155">SQL Database는 기본, 표준 및 프리미엄 [서비스 계층][sql-db-service-tiers]을 지원하며, 각 계층 내에서 [DTU(데이터베이스 트랜잭션 단위)][sql-dtu]로 측정되는 성능 수준이 다양합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-155">SQL Database supports Basic, Standard, and Premium [service tiers][sql-db-service-tiers], with multiple performance levels within each tier measured in [Database Transaction Units (DTUs)][sql-dtu].</span></span> <span data-ttu-id="5af48-156">용량 계획을 수행하고 요구 사항에 맞은 계층과 성능 수준을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-156">Perform capacity planning and choose a tier and performance level that meets your requirements.</span></span>

### <a name="region"></a><span data-ttu-id="5af48-157">지역</span><span class="sxs-lookup"><span data-stu-id="5af48-157">Region</span></span>

<span data-ttu-id="5af48-158">네트워크 대기 시간을 최소화하려면 App Service 계획과 SQL Database를 같은 지역에 프로비전합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-158">Provision the App Service plan and the SQL Database in the same region to minimize network latency.</span></span> <span data-ttu-id="5af48-159">일반적으로 사용자에게 가장 가까운 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-159">Generally, choose the region closest to your users.</span></span>

<span data-ttu-id="5af48-160">리소스 그룹에는 배포 메타데이터가 저장되는 위치를 지정하는 지역도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-160">The resource group also has a region, which specifies where deployment metadata is stored.</span></span> <span data-ttu-id="5af48-161">리소스 그룹과 해당 리소스를 같은 지역에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-161">Put the resource group and its resources in the same region.</span></span> <span data-ttu-id="5af48-162">이렇게 하면 배포 중 가용성을 향상할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-162">This can improve availability during deployment.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="5af48-163">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="5af48-163">Scalability considerations</span></span>

<span data-ttu-id="5af48-164">Azure App Service의 주요 이점은 부하에 따라 응용 프로그램을 확장할 수 있다는 점입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-164">A major benefit of Azure App Service is the ability to scale your application based on load.</span></span> <span data-ttu-id="5af48-165">다음은 응용 프로그램 확장을 계획할 때 염두할 몇 가지 고려 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-165">Here are some considerations to keep in mind when planning to scale your application.</span></span>

### <a name="scaling-the-app-service-app"></a><span data-ttu-id="5af48-166">App Service 앱 크기 조정</span><span class="sxs-lookup"><span data-stu-id="5af48-166">Scaling the App Service app</span></span>

<span data-ttu-id="5af48-167">App Service 앱은 확장하는 방법에는 다음 두 가지가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-167">There are two ways to scale an App Service app:</span></span>

- <span data-ttu-id="5af48-168">*강화* 즉, 인스턴스 크기를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-168">*Scale up*, which means changing the instance size.</span></span> <span data-ttu-id="5af48-169">인스턴스 크기에 따라 각 VM 인스턴스의 메모리, 코어 수 및 저장소가 결정됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-169">The instance size determines the memory, number of cores, and storage on each VM instance.</span></span> <span data-ttu-id="5af48-170">인스턴스 크기나 계획 계층을 변경하여 수동으로 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-170">You can scale up manually by changing the instance size or the plan tier.</span></span>

- <span data-ttu-id="5af48-171">*규모 확장* 즉, 증가된 부하를 처리할 인스턴스를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-171">*Scale out*, which means adding instances to handle increased load.</span></span> <span data-ttu-id="5af48-172">각 가격 책정 계층에는 최대 인스턴스 수가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-172">Each pricing tier has a maximum number of instances.</span></span>

  <span data-ttu-id="5af48-173">인스턴스 수를 변경하여 수동으로 확장하거나 [자동 크기 조정][web-app-autoscale]을 사용하여 Azure에서 일정 및/또는 성능 메트릭에 따라 인스턴스를 자동으로 추가하거나 제거하도록 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-173">You can scale out manually by changing the instance count, or use [autoscaling][web-app-autoscale] to have Azure automatically add or remove instances based on a schedule and/or performance metrics.</span></span> <span data-ttu-id="5af48-174">각 확장 작업은 빠르게, 대개 몇 초 내에 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-174">Each scale operation happens quickly&mdash;typically within seconds.</span></span>

  <span data-ttu-id="5af48-175">자동 크기 조정을 사용하려면 최소 및 최대 인스턴스 수를 정의하는 자동 크기 조정 *프로필*을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-175">To enable autoscaling, create an autoscale *profile* that defines the minimum and maximum number of instances.</span></span> <span data-ttu-id="5af48-176">프로필을 예약할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-176">Profiles can be scheduled.</span></span> <span data-ttu-id="5af48-177">예를 들어 평일과 주말의 프로필을 별개로 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-177">For example, you might create separate profiles for weekdays and weekends.</span></span> <span data-ttu-id="5af48-178">프로필에 인스턴스를 추가하거나 제거할 시기에 대한 규칙을 포함할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-178">Optionally, a profile contains rules for when to add or remove instances.</span></span> <span data-ttu-id="5af48-179">(예: (CPU 사용량이 5분 동안 70% 이상인 경우 두 개의 인스턴스를 추가).</span><span class="sxs-lookup"><span data-stu-id="5af48-179">(Example: Add two instances if CPU usage is above 70% for 5 minutes.)</span></span>

<span data-ttu-id="5af48-180">웹앱 확장에 대한 권장 사항:</span><span class="sxs-lookup"><span data-stu-id="5af48-180">Recommendations for scaling a web app:</span></span>

- <span data-ttu-id="5af48-181">규모를 확장하거나 축소하면 응용 프로그램이 다시 시작될 수 있으므로 가능한 한 피합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-181">As much as possible, avoid scaling up and down, because it may trigger an application restart.</span></span> <span data-ttu-id="5af48-182">그 대신에 일반적인 부하에서 성능 요구 사항에 맞는 계층 및 크기를 선택한 다음 규모 확장하여 트래픽 볼륨의 변화를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-182">Instead, select a tier and size that meet your performance requirements under typical load and then scale out the instances to handle changes in traffic volume.</span></span>
- <span data-ttu-id="5af48-183">자동 크기 조정을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-183">Enable autoscaling.</span></span> <span data-ttu-id="5af48-184">응용 프로그램의 워크로드가 예측 가능하고 정기적이면 인스턴스 수를 미리 예약하는 프로필을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-184">If your application has a predictable, regular workload, create profiles to schedule the instance counts ahead of time.</span></span> <span data-ttu-id="5af48-185">워크로드를 예측할 수 없는 경우에는 규칙 기반 자동 크기 조정을 사용하여 발생하는 부하 변경에 대처합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-185">If the workload is not predictable, use rule-based autoscaling to react to changes in load as they occur.</span></span> <span data-ttu-id="5af48-186">이 두 가지 방법을 결합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-186">You can combine both approaches.</span></span>
- <span data-ttu-id="5af48-187">CPU 사용량은 일반적으로 자동 크기 조정 규칙에 적합한 메트릭입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-187">CPU usage is generally a good metric for autoscale rules.</span></span> <span data-ttu-id="5af48-188">하지만 응용 프로그램 부하 테스트를 수행하고, 잠재적인 병목 상태를 식별하고 이 데이터를 기준으로 자동 크기 조정 규칙을 만들어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-188">However, you should load test your application, identify potential bottlenecks, and base your autoscale rules on that data.</span></span>
- <span data-ttu-id="5af48-189">자동 크기 조정 규칙에는 확장 작업 완료 후 새 확장 작업을 시작하기 전에 대기하는 간격을 의미하는 *휴지* 기간이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-189">Autoscale rules include a *cool-down* period, which is the interval to wait after a scale action has completed before starting a new scale action.</span></span> <span data-ttu-id="5af48-190">휴지 기간을 이용하여 시스템을 다시 확장하기 전에 안정화합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-190">The cool-down period lets the system stabilize before scaling again.</span></span> <span data-ttu-id="5af48-191">인스턴스 추가의 경우 휴지 기간을 짧게 설정하고 인스턴스 제거의 경우 휴지 기간을 길게 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-191">Set a shorter cool-down period for adding instances, and a longer cool-down period for removing instances.</span></span> <span data-ttu-id="5af48-192">예를 들어 인스턴스를 추가하려면 5분을 설정하지만 인스턴스를 제거하려면 60분을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-192">For example, set 5 minutes to add an instance, but 60 minutes to remove an instance.</span></span> <span data-ttu-id="5af48-193">부하가 높은 경우 새 인스턴스를 빠르게 추가하여 추가 트래픽을 처리한 다음 점차 다시 축소하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-193">It's better to add new instances quickly under heavy load to handle the additional traffic, and then gradually scale back.</span></span>

### <a name="scaling-sql-database"></a><span data-ttu-id="5af48-194">SQL Database 크기 조정</span><span class="sxs-lookup"><span data-stu-id="5af48-194">Scaling SQL Database</span></span>

<span data-ttu-id="5af48-195">SQL Database에 대해 더 높은 서비스 계층이나 성능 수준이 필요한 경우 응용 프로그램 가동을 중지하지 않고 개별 데이터베이스를 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-195">If you need a higher service tier or performance level for SQL Database, you can scale up individual databases with no application downtime.</span></span> <span data-ttu-id="5af48-196">자세한 내용은 [SQL Database 옵션 및 성능: 각 서비스 계층에서 사용할 수 있는 기능 이해][sql-db-scale]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-196">For more information, see [SQL Database options and performance: Understand what's available in each service tier][sql-db-scale].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="5af48-197">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="5af48-197">Availability considerations</span></span>

<span data-ttu-id="5af48-198">이 문서 작성 시점을 기준으로 App Service의 SLA(서비스 수준 계약)는 99.95%이고 SQL Database의 SLA는 기본, 표준 및 프리미엄 계층에 대해 99.99%입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-198">At the time of writing, the service level agreement (SLA) for App Service is 99.95% and the SLA for SQL Database is 99.99% for Basic, Standard, and Premium tiers.</span></span>

> [!NOTE]
> <span data-ttu-id="5af48-199">App Service SLA는 단일 인스턴스와 여러 인스턴스 모두에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-199">The App Service SLA applies to both single and multiple instances.</span></span>
>

### <a name="backups"></a><span data-ttu-id="5af48-200">Backup</span><span class="sxs-lookup"><span data-stu-id="5af48-200">Backups</span></span>

<span data-ttu-id="5af48-201">데이터 손실이 발생하면 SQL Database에서는 지정 시간 복원과 지역 복원을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-201">In the event of data loss, SQL Database provides point-in-time restore and geo-restore.</span></span> <span data-ttu-id="5af48-202">이러한 기능은 모든 계층에서 제공되며 자동으로 사용하도록 설정됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-202">These features are available in all tiers and are automatically enabled.</span></span> <span data-ttu-id="5af48-203">백업을 예약하거나 관리할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-203">You don't need to schedule or manage the backups.</span></span>

- <span data-ttu-id="5af48-204">지정 시간 복원을 사용하면 데이터베이스를 이전 시간으로 되돌려 [사용자 오류를 복구][sql-human-error]할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-204">Use point-in-time restore to [recover from human error][sql-human-error] by returning the database to an earlier point in time.</span></span>
- <span data-ttu-id="5af48-205">지역 복원을 사용하면 지역 중복 백업에서 데이터베이스를 복원하여 [서비스 중단에 따른 손상을 복구][sql-outage-recovery]할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-205">Use geo-restore to [recover from a service outage][sql-outage-recovery] by restoring a database from a geo-redundant backup.</span></span>

<span data-ttu-id="5af48-206">자세한 내용은 [SQL Database의 클라우드 무중단 업무 방식 및 데이터베이스 재해 복구][sql-backup]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-206">For more information, see [Cloud business continuity and database disaster recovery with SQL Database][sql-backup].</span></span>

<span data-ttu-id="5af48-207">App Service에서는 응용 프로그램 파일에 대한 [백업 및 복원][web-app-backup] 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-207">App Service provides a [backup and restore][web-app-backup] feature for your application files.</span></span> <span data-ttu-id="5af48-208">그러나 백업된 파일에는 앱 설정이 일반 텍스트로 포함되며 연결 문자열과 같은 비밀이 포함될 수 있다는 점에 유의합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-208">However, be aware that the backed-up files include app settings in plain text and these may include secrets, such as connection strings.</span></span> <span data-ttu-id="5af48-209">SQL Database를 백업하기 위해 App Service 백업 기능을 사용하지 마세요. 이 경우 데이터베이스를 SQL .bacpac 파일로 내보내 [DTU][sql-dtu]를 소모하기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-209">Avoid using the App Service backup feature to back up your SQL databases because it exports the database to a SQL .bacpac file, consuming [DTUs][sql-dtu].</span></span> <span data-ttu-id="5af48-210">대신 위에서 설명한 SQL Database 지정 시간 복원을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-210">Instead, use SQL Database point-in-time restore described above.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="5af48-211">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="5af48-211">Manageability considerations</span></span>

<span data-ttu-id="5af48-212">프로덕션, 개발 및 테스트 환경에 대해 별도의 리소스 그룹을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-212">Create separate resource groups for production, development, and test environments.</span></span> <span data-ttu-id="5af48-213">이렇게 하면 배포 관리, 테스트 배포 삭제, 액세스 권한 할당 등이 더 간단해집니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-213">This makes it easier to manage deployments, delete test deployments, and assign access rights.</span></span>

<span data-ttu-id="5af48-214">리소스를 리소스 그룹에 할당할 때는 다음 사항을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-214">When assigning resources to resource groups, consider the following:</span></span>

- <span data-ttu-id="5af48-215">수명 주기.</span><span class="sxs-lookup"><span data-stu-id="5af48-215">Lifecycle.</span></span> <span data-ttu-id="5af48-216">일반적으로 수명 주기가 동일한 리소스를 동일한 리소스 그룹에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-216">In general, put resources with the same lifecycle into the same resource group.</span></span>
- <span data-ttu-id="5af48-217">액세스.</span><span class="sxs-lookup"><span data-stu-id="5af48-217">Access.</span></span> <span data-ttu-id="5af48-218">[RBAC(역할 기반 액세스 제어)][rbac]를 사용하여 그룹의 리소스에 액세스 정책을 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-218">You can use [role-based access control][rbac] (RBAC) to apply access policies to the resources in a group.</span></span>
- <span data-ttu-id="5af48-219">청구.</span><span class="sxs-lookup"><span data-stu-id="5af48-219">Billing.</span></span> <span data-ttu-id="5af48-220">리소스 그룹에 대한 롤업 비용을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-220">You can view the rolled-up costs for the resource group.</span></span>

<span data-ttu-id="5af48-221">자세한 내용은 [Azure Resource Manager 개요](/azure/azure-resource-manager/resource-group-overview)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-221">For more information, see [Azure Resource Manager overview](/azure/azure-resource-manager/resource-group-overview).</span></span>

### <a name="deployment"></a><span data-ttu-id="5af48-222">배포</span><span class="sxs-lookup"><span data-stu-id="5af48-222">Deployment</span></span>

<span data-ttu-id="5af48-223">배포는 다음 두 단계로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-223">Deployment involves two steps:</span></span>

1. <span data-ttu-id="5af48-224">Azure 리소스 프로비전.</span><span class="sxs-lookup"><span data-stu-id="5af48-224">Provisioning the Azure resources.</span></span> <span data-ttu-id="5af48-225">이 단계에는 [Azure Resource Manager 템플릿][arm-template]을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-225">We recommend that you use [Azure Resource Manager templates][arm-template] for this step.</span></span> <span data-ttu-id="5af48-226">템플릿을 사용하면 PowerShell이나 Azure CLI(명령줄 인터페이스)를 통해 배포를 쉽게 자동화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-226">Templates make it easier to automate deployments via PowerShell or the Azure command line interface (CLI).</span></span>
2. <span data-ttu-id="5af48-227">응용 프로그램(코드, 이진 파일 및 콘텐츠 파일) 배포.</span><span class="sxs-lookup"><span data-stu-id="5af48-227">Deploying the application (code, binaries, and content files).</span></span> <span data-ttu-id="5af48-228">로컬 Git 리포지토리에서 배포하거나, Visual Studio를 사용하여 배포하거나, 클라우드 기반 소스 제어를 통해 연속 배포하는 등 여러 옵션이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-228">You have several options, including deploying from a local Git repository, using Visual Studio, or continuous deployment from cloud-based source control.</span></span> <span data-ttu-id="5af48-229">[Azure App Service에 앱 배포][deploy]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-229">See [Deploy your app to Azure App Service][deploy].</span></span>

<span data-ttu-id="5af48-230">App Service App에는 라이브 프로덕션 사이트를 나타내는 `production`이라는 배포 슬롯 하나가 항상 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-230">An App Service app always has one deployment slot named `production`, which represents the live production site.</span></span> <span data-ttu-id="5af48-231">업데이트를 배포하기 위한 스테이징 슬롯을 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-231">We recommend creating a staging slot for deploying updates.</span></span> <span data-ttu-id="5af48-232">스테이징 슬롯을 사용하면 다음과 같은 이점이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-232">The benefits of using a staging slot include:</span></span>

- <span data-ttu-id="5af48-233">프로덕션으로 교환하기 전에 배포가 성공했는지 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-233">You can verify the deployment succeeded, before swapping it into production.</span></span>
- <span data-ttu-id="5af48-234">스테이징 슬롯에 배포하면 모든 인스턴스가 프로덕션으로 교환되기 전에 확실히 준비됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-234">Deploying to a staging slot ensures that all instances are warmed up before being swapped into production.</span></span> <span data-ttu-id="5af48-235">많은 애플리케이션에는 중요한 준비 및 콜드 부팅 시간이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-235">Many applications have a significant warmup and cold-start time.</span></span>

<span data-ttu-id="5af48-236">또한 마지막으로 성공한 올바른 배포를 보유할 세 번째 슬롯을 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-236">We also recommend creating a third slot to hold the last-known-good deployment.</span></span> <span data-ttu-id="5af48-237">스테이징과 프로덕션을 교환한 후 이전 프로덕션 배포(이제는 스테이징에 있음)를 마지막으로 성공한 올바른 슬롯으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-237">After you swap staging and production, move the previous production deployment (which is now in staging) into the last-known-good slot.</span></span> <span data-ttu-id="5af48-238">이렇게 하면 나중에 문제를 발견하는 경우 마지막으로 성공한 올바른 버전으로 신속하게 되돌릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-238">That way, if you discover a problem later, you can quickly revert to the last-known-good version.</span></span>

<span data-ttu-id="5af48-239">![[1]][1]</span><span class="sxs-lookup"><span data-stu-id="5af48-239">![[1]][1]</span></span>

<span data-ttu-id="5af48-240">이전 버전으로 되돌리는 경우에는 데이터베이스 스키마 변경 내용이 이전 버전과 호환되는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-240">If you revert to a previous version, make sure any database schema changes are backward compatible.</span></span>

<span data-ttu-id="5af48-241">동일한 App Service 계획 내의 모든 앱이 동일한 VM 인스턴스를 공유하므로 프로덕션 배포의 슬롯을 테스트에 사용하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-241">Don't use slots on your production deployment for testing because all apps within the same App Service plan share the same VM instances.</span></span> <span data-ttu-id="5af48-242">예를 들어 부하 테스트는 라이브 프로덕션 사이트의 성능을 떨어뜨릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-242">For example, load tests might degrade the live production site.</span></span> <span data-ttu-id="5af48-243">대신 프로덕션과 테스트에 대한 App Service 계획을 별도로 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-243">Instead, create separate App Service plans for production and test.</span></span> <span data-ttu-id="5af48-244">테스트 배포를 별도의 계획에 배치하여 프로덕션 버전과 격리합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-244">By putting test deployments into a separate plan, you isolate them from the production version.</span></span>

### <a name="configuration"></a><span data-ttu-id="5af48-245">구성</span><span class="sxs-lookup"><span data-stu-id="5af48-245">Configuration</span></span>

<span data-ttu-id="5af48-246">구성 설정을 [앱 설정][app-settings]으로 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-246">Store configuration settings as [app settings][app-settings].</span></span> <span data-ttu-id="5af48-247">앱 설정을 리소스 관리자 템플릿에서 정의하거나 PowerShell을 사용하여 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-247">Define the app settings in your Resource Manager templates, or using PowerShell.</span></span> <span data-ttu-id="5af48-248">런타임 시 앱 설정을 응용 프로그램에 환경 변수로 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-248">At runtime, app settings are available to the application as environment variables.</span></span>

<span data-ttu-id="5af48-249">암호, 액세스 키 또는 연결 문자열을 소스 제어로 체크 인하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-249">Never check passwords, access keys, or connection strings into source control.</span></span> <span data-ttu-id="5af48-250">대신 이러한 값을 배포 스크립트에 매개 변수로 전달하여 앱 설정으로 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-250">Instead, pass these as parameters to a deployment script that stores these values as app settings.</span></span>

<span data-ttu-id="5af48-251">배포 슬롯을 교환하면 앱 설정이 기본적으로 교환됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-251">When you swap a deployment slot, the app settings are swapped by default.</span></span> <span data-ttu-id="5af48-252">프로덕션 및 스테이징에 서로 다른 설정이 필요하면 슬롯에 고정되어 교환되지 않는 앱 설정을 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-252">If you need different settings for production and staging, you can create app settings that stick to a slot and don't get swapped.</span></span>

### <a name="diagnostics-and-monitoring"></a><span data-ttu-id="5af48-253">진단 및 모니터링</span><span class="sxs-lookup"><span data-stu-id="5af48-253">Diagnostics and monitoring</span></span>

<span data-ttu-id="5af48-254">응용 프로그램 로깅 및 웹 서버 로깅을 포함하여 [진단 로깅][diagnostic-logs]을 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-254">Enable [diagnostics logging][diagnostic-logs], including application logging and web server logging.</span></span> <span data-ttu-id="5af48-255">Blob Storage를 사용하도록 로깅을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-255">Configure logging to use Blob storage.</span></span> <span data-ttu-id="5af48-256">성능을 위해 진단 로그를 저장할 별도의 저장소 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-256">For performance reasons, create a separate storage account for diagnostic logs.</span></span> <span data-ttu-id="5af48-257">로그와 애플리케이션 데이터에 동일한 저장소 계정을 사용하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-257">Don't use the same storage account for logs and application data.</span></span> <span data-ttu-id="5af48-258">로깅에 대한 자세한 내용은 [모니터링 및 진단 지침][monitoring-guidance]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-258">For more detailed guidance on logging, see [Monitoring and diagnostics guidance][monitoring-guidance].</span></span>

<span data-ttu-id="5af48-259">[New Relic][ new-relic] 또는 [Application Insights][app-insights] 같은 서비스를 사용하여 응용 프로그램 성능 및 부하를 받을 때의 동작을 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-259">Use a service such as [New Relic][new-relic] or [Application Insights][app-insights] to monitor application performance and behavior under load.</span></span> <span data-ttu-id="5af48-260">Application Insights에 대한 [데이터 속도 제한][app-insights-data-rate]을 알아둡니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-260">Be aware of the [data rate limits][app-insights-data-rate] for Application Insights.</span></span>

<span data-ttu-id="5af48-261">[Azure DevOps][azure-devops] 또는 [Visual Studio Team Foundation Server][tfs]와 같은 도구를 사용하여 부하 테스트를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-261">Perform load testing, using a tool such as [Azure DevOps][azure-devops] or [Visual Studio Team Foundation Server][tfs].</span></span> <span data-ttu-id="5af48-262">클라우드 응용 프로그램의 성능 분석에 대한 개요는 [Performance Analysis Primer][perf-analysis]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-262">For a general overview of performance analysis in cloud applications, see [Performance Analysis Primer][perf-analysis].</span></span>

<span data-ttu-id="5af48-263">응용 프로그램 문제 해결 팁:</span><span class="sxs-lookup"><span data-stu-id="5af48-263">Tips for troubleshooting your application:</span></span>

- <span data-ttu-id="5af48-264">Azure Portal에 [문제 해결 블레이드][troubleshoot-blade]를 사용하여 일반적인 문제에 대한 해결 방법을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-264">Use the [troubleshoot blade][troubleshoot-blade] in the Azure portal to find solutions to common problems.</span></span>
- <span data-ttu-id="5af48-265">[로그 스트리밍][web-app-log-stream]을 사용하도록 설정하여 로그 정보를 거의 실시간으로 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-265">Enable [log streaming][web-app-log-stream] to see logging information in near-real time.</span></span>
- <span data-ttu-id="5af48-266">[Kudu 대시보드][ kudu]에는 응용 프로그램을 모니터링하고 디버깅하기 위한 여러 도구가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-266">The [Kudu dashboard][kudu] has several tools for monitoring and debugging your application.</span></span> <span data-ttu-id="5af48-267">자세한 내용은 [Azure Websites online tools you should know about][kudu](알아 두면 도움이 되는 Azure 웹 사이트 온라인 도구)(블로그 게시물)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-267">For more information, see [Azure Websites online tools you should know about][kudu] (blog post).</span></span> <span data-ttu-id="5af48-268">Azure Portal에서 Kudu 대시보드에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-268">You can reach the Kudu dashboard from the Azure portal.</span></span> <span data-ttu-id="5af48-269">앱의 블레이드를 열고 **도구**, **Kudu**를 차례로 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-269">Open the blade for your app and click **Tools**, then click **Kudu**.</span></span>
- <span data-ttu-id="5af48-270">Visual Studio를 사용하는 경우 [Visual Studio를 사용하여 Azure App Service에서 웹앱 문제 해결][troubleshoot-web-app]에서 디버깅 및 문제 해결 팁을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-270">If you use Visual Studio, see the article [Troubleshoot a web app in Azure App Service using Visual Studio][troubleshoot-web-app] for debugging and troubleshooting tips.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="5af48-271">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="5af48-271">Security considerations</span></span>

<span data-ttu-id="5af48-272">이 섹션에는 이 문서에 설명된 Azure 서비스와 관련된 보안 고려 사항이 나와 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-272">This section lists security considerations that are specific to the Azure services described in this article.</span></span> <span data-ttu-id="5af48-273">보안 모범 사례가 완전히 다 나와 있는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-273">It's not a complete list of security best practices.</span></span> <span data-ttu-id="5af48-274">몇 가지 추가 보안 고려 사항은 [Azure App Service에서 앱 보안][app-service-security]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-274">For some additional security considerations, see [Secure an app in Azure App Service][app-service-security].</span></span>

### <a name="sql-database-auditing"></a><span data-ttu-id="5af48-275">SQL Database 감사</span><span class="sxs-lookup"><span data-stu-id="5af48-275">SQL Database auditing</span></span>

<span data-ttu-id="5af48-276">감사는 규정 준수를 유지 관리하고, 비즈니스 문제나 의심스러운 보안 위반을 나타낼 수 있는 불일치 및 이상 활동을 파악하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-276">Auditing can help you maintain regulatory compliance and get insight into discrepancies and irregularities that could indicate business concerns or suspected security violations.</span></span> <span data-ttu-id="5af48-277">[SQL Database 감사 시작][sql-audit]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-277">See [Get started with SQL database auditing][sql-audit].</span></span>

### <a name="deployment-slots"></a><span data-ttu-id="5af48-278">배포 슬롯</span><span class="sxs-lookup"><span data-stu-id="5af48-278">Deployment slots</span></span>

<span data-ttu-id="5af48-279">각 배포 슬롯에는 공용 IP 주소가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-279">Each deployment slot has a public IP address.</span></span> <span data-ttu-id="5af48-280">개발 및 DevOps 팀의 구성원만 해당 엔드포인트에 연결할 수 있도록 [Azure Active Directory 로그인][aad-auth]을 사용하여 프로덕션이 아닌 슬롯을 보호합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-280">Secure the nonproduction slots using [Azure Active Directory login][aad-auth] so that only members of your development and DevOps teams can reach those endpoints.</span></span>

### <a name="logging"></a><span data-ttu-id="5af48-281">로깅</span><span class="sxs-lookup"><span data-stu-id="5af48-281">Logging</span></span>

<span data-ttu-id="5af48-282">사용자 암호 또는 ID 사기에 사용될 수 있는 기타 정보는 절대로 기록해서는 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-282">Logs should never record users' passwords or other information that might be used to commit identity fraud.</span></span> <span data-ttu-id="5af48-283">데이터를 저장하기 전에 데이터에서 이러한 세부 정보를 지웁니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-283">Scrub those details from the data before storing it.</span></span>

### <a name="ssl"></a><span data-ttu-id="5af48-284">SSL</span><span class="sxs-lookup"><span data-stu-id="5af48-284">SSL</span></span>

<span data-ttu-id="5af48-285">App Service 앱에서는 추가 비용 없이 `azurewebsites.net`의 하위 도메인의 SSL 엔드포인트를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-285">An App Service app includes an SSL endpoint on a subdomain of `azurewebsites.net` at no additional cost.</span></span> <span data-ttu-id="5af48-286">SSL 엔드포인트에는 `*.azurewebsites.net` 도메인에 대한 와일드카드 인증서가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-286">The SSL endpoint includes a wildcard certificate for the `*.azurewebsites.net` domain.</span></span> <span data-ttu-id="5af48-287">사용자 지정 도메인 이름을 사용하는 경우 사용자 지정 도메인과 일치하는 인증서를 제공해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-287">If you use a custom domain name, you must provide a certificate that matches the custom domain.</span></span> <span data-ttu-id="5af48-288">가장 간단한 방법은 Azure Portal에서 직접 인증서를 구입하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-288">The simplest approach is to buy a certificate directly through the Azure portal.</span></span> <span data-ttu-id="5af48-289">다른 인증 기관에서 인증서를 가져올 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-289">You can also import certificates from other certificate authorities.</span></span> <span data-ttu-id="5af48-290">자세한 내용은 [Azure App Service에 대한 SSL 인증서 구입 및 구성][ssl-cert]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-290">For more information, see [Buy and Configure an SSL Certificate for your Azure App Service][ssl-cert].</span></span>

<span data-ttu-id="5af48-291">보안을 극대화하려면 앱에서 HTTP 요청을 리디렉션하여 HTTPS를 적용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-291">As a security best practice, your app should enforce HTTPS by redirecting HTTP requests.</span></span> <span data-ttu-id="5af48-292">응용 프로그램 내부에서 이를 구현하거나 로[Azure App Service에서 앱에 HTTPS 사용][ssl-redirect]에 설명된 대로 URL 다시 쓰기 규칙을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-292">You can implement this inside your application or use a URL rewrite rule as described in [Enable HTTPS for an app in Azure App Service][ssl-redirect].</span></span>

### <a name="authentication"></a><span data-ttu-id="5af48-293">인증</span><span class="sxs-lookup"><span data-stu-id="5af48-293">Authentication</span></span>

<span data-ttu-id="5af48-294">Azure AD, Facebook, Google 또는 Twitter 같은 IDP(ID 공급자)를 통해 인증하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-294">We recommend authenticating through an identity provider (IDP), such as Azure AD, Facebook, Google, or Twitter.</span></span> <span data-ttu-id="5af48-295">인증 흐름에 OAuth 2 또는 OIDC(OpenID Connect)를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-295">Use OAuth 2 or OpenID Connect (OIDC) for the authentication flow.</span></span> <span data-ttu-id="5af48-296">Azure AD는 사용자 및 그룹을 관리하고, 응용 프로그램 역할을 만들고, 온-프레미스 ID를 통합하고, Office 365, 비즈니스용 Skype 같은 백 엔드 서비스를 사용할 수 있는 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-296">Azure AD provides functionality to manage users and groups, create application roles, integrate your on-premises identities, and consume backend services such as Office 365 and Skype for Business.</span></span>

<span data-ttu-id="5af48-297">응용 프로그램에서 사용자 로그인 및 자격 증명을 직접 관리하도록 하지 마세요. 이 경우 잠재적인 공격 노출 영역이 생길 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-297">Avoid having the application manage user logins and credentials directly, as it creates a potential attack surface.</span></span>  <span data-ttu-id="5af48-298">최소한 메일 확인, 암호 복구 및 다단계 인증을 사용하도록 해야 하고 암호 강도를 확인하고, 암호 해시를 안전하게 저장해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-298">At a minimum, you would need to have email confirmation, password recovery, and multi-factor authentication; validate password strength; and store password hashes securely.</span></span> <span data-ttu-id="5af48-299">대형 ID 공급자는 이러한 기능을 모두 자동으로 처리하며 보안 방법을 지속적으로 모니터링하고 개선합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-299">The large identity providers handle all of those things for you, and are constantly monitoring and improving their security practices.</span></span>

<span data-ttu-id="5af48-300">[App Service 인증][app-service-auth]을 사용하여 OAuth/OIDC 인증 흐름을 구현하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-300">Consider using [App Service authentication][app-service-auth] to implement the OAuth/OIDC authentication flow.</span></span> <span data-ttu-id="5af48-301">App Service 인증의 이점은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-301">The benefits of App Service authentication include:</span></span>

- <span data-ttu-id="5af48-302">구성하기 쉽습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-302">Easy to configure.</span></span>
- <span data-ttu-id="5af48-303">단순 인증 시나리오의 경우 코드가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-303">No code is required for simple authentication scenarios.</span></span>
- <span data-ttu-id="5af48-304">OAuth 액세스 토큰을 사용한 위임된 권한 부여를 통해 사용자 대신 리소스를 사용하도록 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-304">Supports delegated authorization using OAuth access tokens to consume resources on behalf of the user.</span></span>
- <span data-ttu-id="5af48-305">토큰 캐시를 기본 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-305">Provides a built-in token cache.</span></span>

<span data-ttu-id="5af48-306">App Service 인증의 몇 가지 제한 사항은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-306">Some limitations of App Service authentication:</span></span>

- <span data-ttu-id="5af48-307">사용자 지정 옵션이 제한되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-307">Limited customization options.</span></span>
- <span data-ttu-id="5af48-308">위임된 권한 부여는 로그인 세션당 하나의 백 엔드 리소스로 제한됩니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-308">Delegated authorization is restricted to one backend resource per login session.</span></span>
- <span data-ttu-id="5af48-309">둘 이상의 IDP를 사용하는 경우 홈 영역 검색을 위한 기본 메커니즘이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-309">If you use more than one IDP, there is no built-in mechanism for home realm discovery.</span></span>
- <span data-ttu-id="5af48-310">다중 테넌트 시나리오의 경우 응용 프로그램에서 토큰 발급자의 유효성을 검사하는 논리를 구현해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-310">For multi-tenant scenarios, the application must implement the logic to validate the token issuer.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="5af48-311">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="5af48-311">Deploy the solution</span></span>

<span data-ttu-id="5af48-312">이 아키텍처에 대한 예제 Resource Manager 템플릿은 [GitHub에서 사용할 수 있습니다][paas-basic-arm-template].</span><span class="sxs-lookup"><span data-stu-id="5af48-312">An example Resource Manager template for this architecture is [available on GitHub][paas-basic-arm-template].</span></span>

<span data-ttu-id="5af48-313">PowerShell을 사용하여 템플릿을 배포하려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="5af48-313">To deploy the template using PowerShell, run the following commands:</span></span>

```powershell
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

<span data-ttu-id="5af48-314">자세한 내용은 [Azure 리소스 관리자 템플릿을 사용하여 리소스 배포][deploy-arm-template]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="5af48-314">For more information, see [Deploy resources with Azure Resource Manager templates][deploy-arm-template].</span></span>

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-devops]: /azure/devops/
[azure-dns]: /azure/dns/dns-overview
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: https://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/managed-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[tfs]: /tfs/index
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[1]: ./images/paas-basic-web-app-staging-slots.png "프로덕션 및 스테이징 배포에 대한 슬롯 교환"
