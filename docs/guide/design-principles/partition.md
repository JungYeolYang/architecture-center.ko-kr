---
title: 한도에 맞춘 분할
titleSuffix: Azure Application Architecture Guide
description: 분할을 사용하여 데이터베이스, 네트워크 및 계산 한도를 해결합니다.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 76590d2a0c16df9d599e7d4a856a84b5e3bdcec8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486015"
---
# <a name="partition-around-limits"></a><span data-ttu-id="52cf8-103">한도에 맞춘 분할</span><span class="sxs-lookup"><span data-stu-id="52cf8-103">Partition around limits</span></span>

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a><span data-ttu-id="52cf8-104">분할을 사용하여 데이터베이스, 네트워크 및 계산 한도를 해결합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-104">Use partitioning to work around database, network, and compute limits</span></span>

<span data-ttu-id="52cf8-105">클라우드에서 모든 서비스는 규모 확장 능력이 제한됩니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-105">In the cloud, all services have limits in their ability to scale up.</span></span> <span data-ttu-id="52cf8-106">Azure 서비스 제한 사항은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건][azure-limits]에 설명되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-106">Azure service limits are documented in [Azure subscription and service limits, quotas, and constraints][azure-limits].</span></span> <span data-ttu-id="52cf8-107">제한에는 코어 수, 데이터베이스 크기, 쿼리 처리량 및 네트워크 처리량이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-107">Limits include number of cores, database size, query throughput, and network throughput.</span></span> <span data-ttu-id="52cf8-108">시스템 충분히 커지면, 이러한 제한 중 하나 이상에 도달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-108">If your system grows sufficiently large, you may hit one or more of these limits.</span></span> <span data-ttu-id="52cf8-109">이러한 제한을 해결하려면 분할을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-109">Use partitioning to work around these limits.</span></span>

<span data-ttu-id="52cf8-110">다음과 같이 시스템을 분할하는 방법에는 여러 가지가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-110">There are many ways to partition a system, such as:</span></span>

- <span data-ttu-id="52cf8-111">데이터베이스 크기, 데이터 I/O 또는 동시 세션 수에 대한 제한을 방지하기 위해 데이터베이스를 분할합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-111">Partition a database to avoid limits on database size, data I/O, or number of concurrent sessions.</span></span>

- <span data-ttu-id="52cf8-112">요청 수 또는 동시 연결 수를 제한하지 않도록 큐 또는 메시지 버스를 분할합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-112">Partition a queue or message bus to avoid limits on the number of requests or the number of concurrent connections.</span></span>

- <span data-ttu-id="52cf8-113">App Service 계획당 인스턴스 수를 제한하지 않도록 App Service 웹앱을 분할합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-113">Partition an App Service web app to avoid limits on the number of instances per App Service plan.</span></span>

<span data-ttu-id="52cf8-114">데이터베이스는 *수평적*, *수직적* 또는 *기능적*에 따라 분할할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-114">A database can be partitioned *horizontally*, *vertically*, or *functionally*.</span></span>

- <span data-ttu-id="52cf8-115">행 분할이라고도 하는 수평적 분할에서 각 파티션은 전체 데이터 집합의 하위 집합에 대한 데이터를 보유합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-115">In horizontal partitioning, also called sharding, each partition holds data for a subset of the total data set.</span></span> <span data-ttu-id="52cf8-116">파티션은 동일한 데이터 스키마를 공유합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-116">The partitions share the same data schema.</span></span> <span data-ttu-id="52cf8-117">예를 들어, 이름이 A&ndash;M으로 시작하는 고객과 N&ndash;Z로 시작하는 고객은 다른 파티션에 속합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-117">For example, customers whose names start with A&ndash;M go into one partition, N&ndash;Z into another partition.</span></span>

- <span data-ttu-id="52cf8-118">수직적 분할에서 각 파티션은 데이터 저장소의 항목에 대한 필드 하위 집합을 보유합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-118">In vertical partitioning, each partition holds a subset of the fields for the items in the data store.</span></span> <span data-ttu-id="52cf8-119">예를 들어, 자주 액세스되는 필드와 덜 자주 액세스되는 필드를 다른 파티션에 둡니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-119">For example, put frequently accessed fields in one partition, and less frequently accessed fields in another.</span></span>

- <span data-ttu-id="52cf8-120">기능적 파티션에서는 시스템의 제한된 컨텍스트별로 데이터가 사용되는 방법에 따라 데이터를 분할합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-120">In functional partitioning, data is partitioned according to how it is used by each bounded context in the system.</span></span> <span data-ttu-id="52cf8-121">예를 들어, 청구서 데이터와 제품 재고 데이터를 다른 파티션에 둡니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-121">For example, store invoice data in one partition and product inventory data in another.</span></span> <span data-ttu-id="52cf8-122">스키마는 서로 독립적입니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-122">The schemas are independent.</span></span>

<span data-ttu-id="52cf8-123">자세한 지침은 [데이터 분할][data-partitioning-guidance]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="52cf8-123">For more detailed guidance, see [Data partitioning][data-partitioning-guidance].</span></span>

## <a name="recommendations"></a><span data-ttu-id="52cf8-124">권장 사항</span><span class="sxs-lookup"><span data-stu-id="52cf8-124">Recommendations</span></span>

<span data-ttu-id="52cf8-125">**애플리케이션의 다른 부분 분할**</span><span class="sxs-lookup"><span data-stu-id="52cf8-125">**Partition different parts of the application**.</span></span> <span data-ttu-id="52cf8-126">데이터베이는 분할에 적합한 후보임에 분명하지만, 저장소, 캐시, 큐 및 계산 인스턴스의 분할도 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-126">Databases are one obvious candidate for partitioning, but also consider storage, cache, queues, and compute instances.</span></span>

<span data-ttu-id="52cf8-127">**핫 스폿을 방지하도록 파티션 키 디자인**</span><span class="sxs-lookup"><span data-stu-id="52cf8-127">**Design the partition key to avoid hot spots**.</span></span> <span data-ttu-id="52cf8-128">데이터베이스를 분할해도, 하나의 분할된 데이터베이스가 요청의 대부분을 가져가므로 문제가 해결되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-128">If you partition a database, but one shard still gets the majority of the requests, then you haven't solved your problem.</span></span> <span data-ttu-id="52cf8-129">이상적인 목표는 부하가 모든 파티션에 골고루 분산되는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-129">Ideally, load gets distributed evenly across all the partitions.</span></span> <span data-ttu-id="52cf8-130">예를 들어, 일부 문자가 좀 더 자주 사용되므로 고객 이름의 첫 문자가 아닌 고객 ID별로 해시를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-130">For example, hash by customer ID and not the first letter of the customer name, because some letters are more frequent.</span></span> <span data-ttu-id="52cf8-131">메시지 큐를 분할할 때도 같은 원칙이 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-131">The same principle applies when partitioning a message queue.</span></span> <span data-ttu-id="52cf8-132">큐 집합에서 메시지를 균등하게 분산하는 파티션 키를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-132">Pick a partition key that leads to an even distribution of messages across the set of queues.</span></span> <span data-ttu-id="52cf8-133">자세한 내용은 [분할][sharding]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="52cf8-133">For more information, see [Sharding][sharding].</span></span>

<span data-ttu-id="52cf8-134">**Azure 구독 및 서비스 제한을 고려한 분할**</span><span class="sxs-lookup"><span data-stu-id="52cf8-134">**Partition around Azure subscription and service limits**.</span></span> <span data-ttu-id="52cf8-135">개별 구성 요소 및 서비스에는 제한이 적용되지만, 구독 및 리소스 그룹에 대해서도 제한이 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-135">Individual components and services have limits, but there are also limits for subscriptions and resource groups.</span></span> <span data-ttu-id="52cf8-136">매우 큰 애플리케이션의 경우, 이러한 제한을 고려해서 분할해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-136">For very large applications, you might need to partition around those limits.</span></span>

<span data-ttu-id="52cf8-137">**다양한 수준에서 분할**</span><span class="sxs-lookup"><span data-stu-id="52cf8-137">**Partition at different levels**.</span></span> <span data-ttu-id="52cf8-138">VM에 배포된 데이터베이스 서버를 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-138">Consider a database server deployed on a VM.</span></span> <span data-ttu-id="52cf8-139">VM에는 Azure Storage가 지원하는 VHD가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-139">The VM has a VHD that is backed by Azure Storage.</span></span> <span data-ttu-id="52cf8-140">저장소 계정은 Azure 구독에 속합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-140">The storage account belongs to an Azure subscription.</span></span> <span data-ttu-id="52cf8-141">계층의 각 단계에도 제한이 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-141">Notice that each step in the hierarchy has limits.</span></span> <span data-ttu-id="52cf8-142">데이터베이스 서버에는 연결 풀 제한이 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-142">The database server may have a connection pool limit.</span></span> <span data-ttu-id="52cf8-143">VM에는 CPU 및 네트워크 제한이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-143">VMs have CPU and network limits.</span></span> <span data-ttu-id="52cf8-144">저장소에는 IOPS 제한이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-144">Storage has IOPS limits.</span></span> <span data-ttu-id="52cf8-145">구독에는 VM 코어 수 제한이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-145">The subscription has limits on the number of VM cores.</span></span> <span data-ttu-id="52cf8-146">일반적으로 계층 구조의 하위에서 분할하는 것이 더 쉽습니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-146">Generally, it's easier to partition lower in the hierarchy.</span></span> <span data-ttu-id="52cf8-147">대규모 애플리케이션의 경우만 구독 수준에서 분할해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="52cf8-147">Only large applications should need to partition at the subscription level.</span></span>

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md
