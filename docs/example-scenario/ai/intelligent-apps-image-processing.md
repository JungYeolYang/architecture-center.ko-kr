---
title: 보험 청구에 대한 이미지 분류
titleSuffix: Azure Example Scenarios
description: Azure 애플리케이션에 이미지 처리를 빌드합니다.
author: david-stanford
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-intelligent-apps-image-processing.png
ms.openlocfilehash: 03ef9d15ec9bf64dc743657e9d1e7a7e275c5a8e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244804"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a><span data-ttu-id="3603a-103">Azure에서 보험 청구에 대한 이미지 분류</span><span class="sxs-lookup"><span data-stu-id="3603a-103">Image classification for insurance claims on Azure</span></span>

<span data-ttu-id="3603a-104">이 예제 시나리오는 이미지를 처리해야 하는 비즈니스와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-104">This scenario is relevant for businesses that need to process images.</span></span>

<span data-ttu-id="3603a-105">잠재적인 애플리케이션에는 패션 웹 사이트에 대한 이미지 분류, 보험 청구에 대한 텍스트 및 이미지 분석 또는 게임 스크린샷의 원격 분석 데이터 인식이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="3603a-106">일반적으로 회사에서는 기계 학습 모델에 대한 전문 지식을 개발하고, 모델을 학습하며, 마지막으로 사용자 지정 프로세스를 통해 이미지를 실행하여 이미지에서 데이터를 가져와야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="3603a-107">Computer Vision API 및 Azure Functions와 같은 Azure 서비스를 사용하면, 개별 서버를 관리할 필요가 없으며, 비용을 줄이고, Microsoft에서 Cognitive Services를 사용하여 이미지를 처리할 때 이미 개발한 전문 지식을 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive Services.</span></span> <span data-ttu-id="3603a-108">이 예제 시나리오에서는 특히 이미지 처리 사용 사례에 대해 다루고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-108">This example scenario specifically addresses an image-processing use case.</span></span> <span data-ttu-id="3603a-109">다양한 AI 요구 사항이 있으면 [Cognitive Services](/azure/#pivot=products&panel=ai)의 전체 제품군을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-109">If you have different AI needs, consider the full suite of [Cognitive Services](/azure/#pivot=products&panel=ai).</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="3603a-110">관련 사용 사례</span><span class="sxs-lookup"><span data-stu-id="3603a-110">Relevant use cases</span></span>

<span data-ttu-id="3603a-111">관련된 다른 사용 사례는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-111">Other relevant use cases include:</span></span>

- <span data-ttu-id="3603a-112">패션 웹 사이트에서 이미지 분류</span><span class="sxs-lookup"><span data-stu-id="3603a-112">Classifying images on a fashion website.</span></span>
- <span data-ttu-id="3603a-113">게임 스크린샷의 원격 분석 데이터 분류</span><span class="sxs-lookup"><span data-stu-id="3603a-113">Classifying telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="3603a-114">아키텍처</span><span class="sxs-lookup"><span data-stu-id="3603a-114">Architecture</span></span>

![이미지 분류를 위한 아키텍처][architecture]

<span data-ttu-id="3603a-116">이 시나리오에서는 웹 또는 모바일 애플리케이션의 백 엔드 구성 요소에 대해 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-116">This scenario covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="3603a-117">시나리오를 통한 데이터 흐름은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-117">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="3603a-118">API 레이어는 Azure Functions를 사용하여 빌드됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-118">The API layer is built using Azure Functions.</span></span> <span data-ttu-id="3603a-119">이러한 API를 통해 애플리케이션은 Cosmos DB에서 이미지를 업로드하고 데이터를 검색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-119">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>
2. <span data-ttu-id="3603a-120">이미지가 API 호출을 통해 업로드되면 Blob Storage에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-120">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>
3. <span data-ttu-id="3603a-121">Blob Storage에 새 파일이 추가되면 Azure Function으로 Event Grid 알림이 전송됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-121">Adding new files to Blob storage triggers an Event Grid notification to be sent to an Azure Function.</span></span>
4. <span data-ttu-id="3603a-122">Azure Functions에서 분석을 위해 새로 업로드된 파일에 대한 링크를 Computer Vision API로 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-122">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>
5. <span data-ttu-id="3603a-123">데이터가 Computer Vision API로부터 반환되면 Azure Functions에서 Cosmos DB에 항목을 만들어 이미지 메타데이터와 함께 분석 결과를 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-123">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis along with the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="3603a-124">구성 요소</span><span class="sxs-lookup"><span data-stu-id="3603a-124">Components</span></span>

- <span data-ttu-id="3603a-125">[Computer Vision API](/azure/cognitive-services/computer-vision/home)는 Cognitive Services 제품군의 일부이며 각 이미지에 대한 정보를 검색하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-125">[Computer Vision API](/azure/cognitive-services/computer-vision/home) is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>
- <span data-ttu-id="3603a-126">[Azure Functions](/azure/azure-functions/functions-overview)는 업로드된 이미지에 대한 이벤트 처리뿐만 아니라 웹 애플리케이션에 대한 백 엔드 API도 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-126">[Azure Functions](/azure/azure-functions/functions-overview) provides the back-end API for the web application, as well as the event processing for uploaded images.</span></span>
- <span data-ttu-id="3603a-127">[Event Grid](/azure/event-grid/overview)는 Blob Storage에 새 이미지가 업로드되면 이벤트를 트리거합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-127">[Event Grid](/azure/event-grid/overview) triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="3603a-128">그런 다음, 이미지는 Azure Functions에서 처리됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-128">The image is then processed with Azure functions.</span></span>
- <span data-ttu-id="3603a-129">[Blob Storage](/azure/storage/blobs/storage-blobs-introduction)는 웹 응용 프로그램에 업로드되는 모든 이미지 파일과 웹 응용 프로그램에서 사용하는 모든 정적 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-129">[Blob storage](/azure/storage/blobs/storage-blobs-introduction) stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>
- <span data-ttu-id="3603a-130">[Cosmos DB](/azure/cosmos-db/introduction)는 Computer Vision API에서 처리한 결과를 포함하여 업로드된 각 이미지에 대한 메타데이터를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-130">[Cosmos DB](/azure/cosmos-db/introduction) stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="3603a-131">대안</span><span class="sxs-lookup"><span data-stu-id="3603a-131">Alternatives</span></span>

- <span data-ttu-id="3603a-132">[Custom Vision Service](/azure/cognitive-services/custom-vision-service/home).</span><span class="sxs-lookup"><span data-stu-id="3603a-132">[Custom Vision Service](/azure/cognitive-services/custom-vision-service/home).</span></span> <span data-ttu-id="3603a-133">Computer Vision API는 일단의 [분류 기반 범주][cv-categories]를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-133">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="3603a-134">Computer Vision API에서 반환하지 않는 정보를 처리해야 하는 경우 사용자 지정 이미지 분류기를 만들 수 있는 Custom Vision Service를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-134">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>
- <span data-ttu-id="3603a-135">[Azure Search](/azure/search/search-what-is-azure-search).</span><span class="sxs-lookup"><span data-stu-id="3603a-135">[Azure Search](/azure/search/search-what-is-azure-search).</span></span> <span data-ttu-id="3603a-136">특정 조건을 충족하는 이미지를 찾기 위해 메타데이터를 쿼리하는 사용 사례가 있는 경우 Azure Search를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-136">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="3603a-137">현재 미리 보기에서 [인지 검색](/azure/search/cognitive-search-concept-intro)은 이 워크플로를 원활하게 통합합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-137">Currently in preview, [Cognitive search](/azure/search/cognitive-search-concept-intro) seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="3603a-138">고려 사항</span><span class="sxs-lookup"><span data-stu-id="3603a-138">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="3603a-139">확장성</span><span class="sxs-lookup"><span data-stu-id="3603a-139">Scalability</span></span>

<span data-ttu-id="3603a-140">이 시나리오에서 사용된 대부분의 구성 요소는 자동으로 크기 조정되는 관리 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-140">The majority of the components used in this example scenario are managed services that will automatically scale.</span></span> <span data-ttu-id="3603a-141">몇 가지 주목할 만한 예외: Azure Functions에는 최대 200개의 인스턴스 제한이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-141">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="3603a-142">이 제한을 초과하여 확장해야 하는 경우 여러 지역 또는 앱 계획을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-142">If you need to scale beyond this limit, consider multiple regions or app plans.</span></span>

<span data-ttu-id="3603a-143">Cosmos DB는 프로비전된 RU(요청 단위)를 기준으로 자동으로 크기가 조정되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-143">Cosmos DB doesn’t autoscale in terms of provisioned request units (RUs).</span></span> <span data-ttu-id="3603a-144">요구 사항 추정에 대한 지침은 설명서의 [요청 단위](/azure/cosmos-db/request-units)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3603a-144">For guidance on estimating your requirements see [request units](/azure/cosmos-db/request-units) in our documentation.</span></span> <span data-ttu-id="3603a-145">Cosmos DB의 크기 조정을 최대한 활용하려면 CosmosDB에서 [파티션 키](/azure/cosmos-db/partition-data)가 작동하는 원리를 이해해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-145">To fully take advantage of the scaling in Cosmos DB, understand how [partition keys](/azure/cosmos-db/partition-data) work in CosmosDB.</span></span>

<span data-ttu-id="3603a-146">NoSQL 데이터베이스는 가용성, 확장성 및 파티션에 대한 일관성을 자주 교환합니다(CAP 정리의 의미에서).</span><span class="sxs-lookup"><span data-stu-id="3603a-146">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability, and partitioning.</span></span> <span data-ttu-id="3603a-147">이 시나리오에서는 키-값 데이터 모델이 사용되며 대부분의 작업이 원자성 정의이므로 트랜잭션 일관성은 거의 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-147">In this example scenario, a key-value data model is used and transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="3603a-148">[적절한 데이터 저장소 선택](../../guide/technology-choices/data-store-overview.md)에 대한 추가 지침은 Azure 아키텍처 센터에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-148">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the Azure Architecture Center.</span></span> <span data-ttu-id="3603a-149">구현에 높은 일관성이 필요한 경우 CosmosDB에서 [일관성 수준을 선택](/azure/cosmos-db/consistency-levels)할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-149">If your implementation requires high consistency, you can [choose your consistency level](/azure/cosmos-db/consistency-levels) in CosmosDB.</span></span>

<span data-ttu-id="3603a-150">확장 가능한 솔루션 설계에 대한 일반적인 지침은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3603a-150">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="3603a-151">보안</span><span class="sxs-lookup"><span data-stu-id="3603a-151">Security</span></span>

<span data-ttu-id="3603a-152">[Azure 리소스용 관리 ID][msi]는 계정 내부의 다른 리소스에 대한 액세스를 제공하는 데 사용된 후, Azure Functions에 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-152">[Managed identities for Azure resources][msi] are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="3603a-153">이러한 ID에 필요한 리소스에만 액세스할 수 있도록 허용하여 추가적으로 사용자의 함수에(그리고 잠재적으로 고객에게 ) 노출되지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-153">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>

<span data-ttu-id="3603a-154">보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3603a-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="3603a-155">복원력</span><span class="sxs-lookup"><span data-stu-id="3603a-155">Resiliency</span></span>

<span data-ttu-id="3603a-156">이 시나리오의 모든 구성 요소가 관리되므로 모두 지역 수준에서 자동으로 복원됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-156">All of the components in this scenario are managed, so at a regional level they are all resilient automatically.</span></span>

<span data-ttu-id="3603a-157">복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 애플리케이션 디자인][resiliency]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3603a-157">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="3603a-158">가격</span><span class="sxs-lookup"><span data-stu-id="3603a-158">Pricing</span></span>

<span data-ttu-id="3603a-159">이 시나리오를 실행하는 데 들어가는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-159">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="3603a-160">특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 트래픽에 맞게 적절한 변수를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-160">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="3603a-161">트래픽 양을 기준으로 다음 세 가지 샘플 비용 프로필을 제공했습니다(모든 이미지가 100kb 크기라고 가정).</span><span class="sxs-lookup"><span data-stu-id="3603a-161">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100 kb in size):</span></span>

- <span data-ttu-id="3603a-162">[소형][small-pricing]: 이 가격 책정 예제는 매월&lt; 5000개 이미지 처리와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-162">[Small][small-pricing]: this pricing example correlates to processing &lt; 5000 images a month.</span></span>
- <span data-ttu-id="3603a-163">[중형][medium-pricing]: 이 가격 책정 예제는 매월 500,000개 이미지 처리와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-163">[Medium][medium-pricing]: this pricing example correlates to processing 500,000 images a month.</span></span>
- <span data-ttu-id="3603a-164">[대형][large-pricing]: 이 가격 책정 예제는 매월 5천만 개 이미지 처리와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-164">[Large][large-pricing]: this pricing example correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="3603a-165">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="3603a-165">Related resources</span></span>

<span data-ttu-id="3603a-166">단계별 학습 경로는 [Azure에서 서버리스 웹앱 빌드][serverless]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3603a-166">For a guided learning path, see [Build a serverless web app in Azure][serverless].</span></span>

<span data-ttu-id="3603a-167">프로덕션 환경에서 이 예제 시나리오를 배포하기 전에 [Azure Functions에서 성능 및 안정성 최적화][functions-best-practices]를 위한 권장 사례를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="3603a-167">Before deploying this example scenario in a production environment, review recommended practices for [optimizing the performance and reliability of Azure Functions][functions-best-practices].</span></span>

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
