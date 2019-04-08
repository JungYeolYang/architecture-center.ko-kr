---
title: 규모 확장을 위한 디자인
titleSuffix: Azure Application Architecture Guide
description: 클라우드 애플리케이션은 수평적 크기 조정에 맞게 설계되어야 합니다.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 32ea1e7dc732c819783ad2fc06dbcffd75685d23
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483737"
---
# <a name="design-to-scale-out"></a><span data-ttu-id="b7916-103">규모 확장을 위한 디자인</span><span class="sxs-lookup"><span data-stu-id="b7916-103">Design to scale out</span></span>

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a><span data-ttu-id="b7916-104">수평적으로 애플리케이션 크기가 조정될 수 있도록 애플리케이션 디자인</span><span class="sxs-lookup"><span data-stu-id="b7916-104">Design your application so that it can scale horizontally</span></span>

<span data-ttu-id="b7916-105">클라우드의 주요 장점은 탄력적인 크기 조정으로, 필요한 만큼 용량을 사용하고 부하 증가에 따라 확장하고 추가 용량이 필요하지 않을 때 축소할 수 있다는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-105">A primary advantage of the cloud is elastic scaling &mdash; the ability to use as much capacity as you need, scaling out as load increases, and scaling in when the extra capacity is not needed.</span></span> <span data-ttu-id="b7916-106">수요에 따라 새 인스턴스를 추가하거나 제거하여 규모 확장이 가능하도록 애플리케이션을 디자인합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-106">Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

## <a name="recommendations"></a><span data-ttu-id="b7916-107">권장 사항</span><span class="sxs-lookup"><span data-stu-id="b7916-107">Recommendations</span></span>

<span data-ttu-id="b7916-108">**인스턴스 연결 유지 방지**.</span><span class="sxs-lookup"><span data-stu-id="b7916-108">**Avoid instance stickiness**.</span></span> <span data-ttu-id="b7916-109">연결 유지 또는 *세션 선호도*는 동일한 클라이언트의 요청이 항상 동일한 서버로 라우트되는 경우입니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-109">Stickiness, or *session affinity*, is when requests from the same client are always routed to the same server.</span></span> <span data-ttu-id="b7916-110">연결 유지는 애플리케이션의 확장 기능을 제한합니다. 예를 들어 높은 볼륨의 사용자 트래픽이 인스턴스에 분배되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-110">Stickiness limits the application's ability to scale out. For example, traffic from a high-volume user will not be distributed across instances.</span></span> <span data-ttu-id="b7916-111">연결 유지의 원인에는 메모리에 세션 상태 저장, 암호화에 컴퓨터 특정 키 사용 등이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-111">Causes of stickiness include storing session state in memory, and using machine-specific keys for encryption.</span></span> <span data-ttu-id="b7916-112">모든 인스턴스가 요청을 처리할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-112">Make sure that any instance can handle any request.</span></span>

<span data-ttu-id="b7916-113">**병목 상태 식별**.</span><span class="sxs-lookup"><span data-stu-id="b7916-113">**Identify bottlenecks**.</span></span> <span data-ttu-id="b7916-114">확장을 통해 모든 성능 문제를 해결할 수 있는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-114">Scaling out isn't a magic fix for every performance issue.</span></span> <span data-ttu-id="b7916-115">예를 들어 백 엔드 데이터베이스가 병목 상태인 경우 웹 서버를 더 추가해도 도움이 되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-115">For example, if your backend database is the bottleneck, it won't help to add more web servers.</span></span> <span data-ttu-id="b7916-116">문제 지점에 인스턴스를 더 추가하기 전에 시스템의 병목 상태를 먼저 식별하고 해결하세요.</span><span class="sxs-lookup"><span data-stu-id="b7916-116">Identify and resolve the bottlenecks in the system first, before throwing more instances at the problem.</span></span> <span data-ttu-id="b7916-117">일반적으로 시스템의 상태 저장 부분이 병목 상태의 주요 원인입니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-117">Stateful parts of the system are the most likely cause of bottlenecks.</span></span>

<span data-ttu-id="b7916-118">**확장성 요구 사항별로 워크로드 분해**.</span><span class="sxs-lookup"><span data-stu-id="b7916-118">**Decompose workloads by scalability requirements.**</span></span>  <span data-ttu-id="b7916-119">애플리케이션은 종종 크기 조정 요구 사항이 각기 다른 여러 워크로드로 구성되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-119">Applications often consist of multiple workloads, with different requirements for scaling.</span></span> <span data-ttu-id="b7916-120">예를 들어 애플리케이션에 공용 사이트와 별도의 관리 사이트가 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-120">For example, an application might have a public-facing site and a separate administration site.</span></span> <span data-ttu-id="b7916-121">공용 사이트의 트래픽은 갑자기 급증할 수 있는 반면 관리 사이트의 부하는 더 적고 예측 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-121">The public site may experience sudden surges in traffic, while the administration site has a smaller, more predictable load.</span></span>

<span data-ttu-id="b7916-122">**리소스를 많이 사용하는 태스크 오프로드.**</span><span class="sxs-lookup"><span data-stu-id="b7916-122">**Offload resource-intensive tasks.**</span></span> <span data-ttu-id="b7916-123">사용자 요청을 처리하는 프런트 엔드의 부하를 최소화하기 위해 가능한 경우 많은 CPU 또는 I/O 리소스가 필요한 태스크를 [백그라운드 작업][background-jobs]으로 이동해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-123">Tasks that require a lot of CPU or I/O resources should be moved to [background jobs][background-jobs] when possible, to minimize the load on the front end that is handling user requests.</span></span>

<span data-ttu-id="b7916-124">**기본 제공 자동 크기 조정 기능 사용**.</span><span class="sxs-lookup"><span data-stu-id="b7916-124">**Use built-in autoscaling features**.</span></span> <span data-ttu-id="b7916-125">기본적으로 자동 크기 조정을 지원하는 Azure 컴퓨팅 서비스가 많습니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-125">Many Azure compute services have built-in support for autoscaling.</span></span> <span data-ttu-id="b7916-126">애플리케이션의 워크로드가 예측 가능하고 정기적인 경우 일정에 따라 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-126">If the application has a predictable, regular workload, scale out on a schedule.</span></span> <span data-ttu-id="b7916-127">예를 들어 업무 시간 중에 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-127">For example, scale out during business hours.</span></span> <span data-ttu-id="b7916-128">워크로드가 예측 불가능한 경우 CPU 또는 요청 큐 길이와 같은 성능 지표를 사용하여 자동 크기 조정을 트리거합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-128">Otherwise, if the workload is not predictable, use performance metrics such as CPU or request queue length to trigger autoscaling.</span></span> <span data-ttu-id="b7916-129">자동 크기 조정 모범 사례는 [자동 크기 조정][autoscaling]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b7916-129">For autoscaling best practices, see [Autoscaling][autoscaling].</span></span>

<span data-ttu-id="b7916-130">**중요 워크로드에 대해 적극적인 자동 크기 조정 고려**.</span><span class="sxs-lookup"><span data-stu-id="b7916-130">**Consider aggressive autoscaling for critical workloads**.</span></span> <span data-ttu-id="b7916-131">중요 워크로드의 경우 수요를 미리 예측하려고 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-131">For critical workloads, you want to keep ahead of demand.</span></span> <span data-ttu-id="b7916-132">부하가 높은 경우 새 인스턴스를 빠르게 추가하여 추가 트래픽을 처리한 다음 점차 다시 축소하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-132">It's better to add new instances quickly under heavy load to handle the additional traffic, and then gradually scale back.</span></span>

<span data-ttu-id="b7916-133">**규모 감축 디자인**.</span><span class="sxs-lookup"><span data-stu-id="b7916-133">**Design for scale in**.</span></span>  <span data-ttu-id="b7916-134">탄력적 크기 조정을 사용하는 경우 인스턴스가 제거되면 애플리케이션의 크기가 축소됩니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-134">Remember that with elastic scale, the application will have periods of scale in, when instances get removed.</span></span> <span data-ttu-id="b7916-135">애플리케이션이 제거되는 인스턴스를 정상적으로 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-135">The application must gracefully handle instances being removed.</span></span> <span data-ttu-id="b7916-136">다음은 규모 감축을 처리하는 몇 가지 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-136">Here are some ways to handle scalein:</span></span>

- <span data-ttu-id="b7916-137">종료 이벤트(사용 가능한 경우)를 수신 대기하고 정상적으로 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-137">Listen for shutdown events (when available) and shut down cleanly.</span></span>
- <span data-ttu-id="b7916-138">서비스의 클라이언트/소비자가 일시적인 결함 처리 및 다시 시도를 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-138">Clients/consumers of a service should support transient fault handling and retry.</span></span>
- <span data-ttu-id="b7916-139">장기 실행 태스크의 경우 검사점 또는 [파이프 및 필터][pipes-filters-pattern] 패턴을 사용하여 작업을 분할하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-139">For long-running tasks, consider breaking up the work, using checkpoints or the [Pipes and Filters][pipes-filters-pattern] pattern.</span></span>
- <span data-ttu-id="b7916-140">처리 중에 인스턴스가 제거될 경우 다른 인스턴스가 작업을 선택할 수 있도록 큐에 작업 항목을 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="b7916-140">Put work items on a queue so that another instance can pick up the work, if an instance is removed in the middle of processing.</span></span>

<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
