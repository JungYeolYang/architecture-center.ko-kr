---
title: 컨테이너 기반 작업에 대한 CI/CD 파이프라인
titleSuffix: Azure Example Scenarios
description: Jenkins, Azure Container Registry, Azure Kubernetes Service, Cosmos DB 및 Grafana를 사용하여 Node.js 웹앱에 대한 DevOps 파이프라인을 빌드합니다.
author: iainfoulds
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-devops-with-aks.png
ms.openlocfilehash: 9be4f828c96c4ac321acf9d9719d0ef465fb35cf
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641181"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a><span data-ttu-id="f807c-103">컨테이너 기반 작업에 대한 CI/CD 파이프라인</span><span class="sxs-lookup"><span data-stu-id="f807c-103">CI/CD pipeline for container-based workloads</span></span>

<span data-ttu-id="f807c-104">이 예제 시나리오는 컨테이너 및 DevOps 흐름을 사용하여 애플리케이션 개발을 현대화하려는 비즈니스에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-104">This example scenario is applicable to businesses that want to modernize application development by using containers and DevOps workflows.</span></span> <span data-ttu-id="f807c-105">이 시나리오에서는 Jenkins에서 Node.js 웹앱을 Azure Container Registry 및 Azure Kubernetes Service에 구축하고 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-105">In this scenario, a Node.js web app is built and deployed by Jenkins into an Azure Container Registry and Azure Kubernetes Service.</span></span> <span data-ttu-id="f807c-106">전역 분산 데이터베이스 계층에는 Azure Cosmos DB가 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-106">For a globally distributed database tier, Azure Cosmos DB is used.</span></span> <span data-ttu-id="f807c-107">Azure Monitor는 애플리케이션 성능을 모니터링하고 문제를 해결하기 위해 Grafana 인스턴스 및 대시보드와 통합됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-107">To monitor and troubleshoot application performance, Azure Monitor integrates with a Grafana instance and dashboard.</span></span>

<span data-ttu-id="f807c-108">예제 애플리케이션 시나리오에는 자동화된 개발 환경 제공, 새 코드 커밋의 유효성 검사 및 준비 또는 프로덕션 환경으로 새 배포 푸시가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-108">Example application scenarios include providing an automated development environment, validating new code commits, and pushing new deployments into staging or production environments.</span></span> <span data-ttu-id="f807c-109">일반적으로 기업에서는 애플리케이션과 업데이트를 수동으로 빌드 및 컴파일하고, 대규모의 모놀리식 코드베이스를 유지해야 했습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-109">Traditionally, businesses had to manually build and compile applications and updates, and maintain a large, monolithic code base.</span></span> <span data-ttu-id="f807c-110">CI(지속적인 통합) 및 CD(지속적인 배포)를 사용하는 애플리케이션 개발에 대한 현대적인 접근 방식을 사용하면, 서비스를 더 빠르게 빌드, 테스트 및 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-110">With a modern approach to application development that uses continuous integration (CI) and continuous deployment (CD), you can more quickly build, test, and deploy services.</span></span> <span data-ttu-id="f807c-111">이러한 현대적인 접근 방식을 통해 고객에게 애플리케이션과 업데이트를 더 빨리 릴리스하고, 변화하는 비즈니스 요구 사항에 더 민첩하게 대응할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-111">This modern approach lets you release applications and updates to your customers faster, and respond to changing business demands in a more agile manner.</span></span>

<span data-ttu-id="f807c-112">Azure Kubernetes Service, Container Registry 및 Cosmos DB와 같은 Azure 서비스를 사용하면, 최신의 애플리케이션 개발 기술과 도구를 사용하여 고가용성 구현 프로세스를 간소화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-112">By using Azure services such as Azure Kubernetes Service, Container Registry, and Cosmos DB, companies can use the latest in application development techniques and tools to simplify the process of implementing high availability.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="f807c-113">관련 사용 사례</span><span class="sxs-lookup"><span data-stu-id="f807c-113">Relevant use cases</span></span>

<span data-ttu-id="f807c-114">관련된 다른 사용 사례는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-114">Other relevant use cases include:</span></span>

- <span data-ttu-id="f807c-115">애플리케이션 개발 사례를 마이크로 서비스, 컨테이너 기반 접근 방식으로 현대화</span><span class="sxs-lookup"><span data-stu-id="f807c-115">Modernizing application development practices to a microservice, container-based approach.</span></span>
- <span data-ttu-id="f807c-116">애플리케이션 개발 및 배포 수명 주기 가속화</span><span class="sxs-lookup"><span data-stu-id="f807c-116">Speeding up application development and deployment lifecycles.</span></span>
- <span data-ttu-id="f807c-117">유효성 검사를 위해 테스트 또는 수용 환경에 대한 배포 자동화</span><span class="sxs-lookup"><span data-stu-id="f807c-117">Automating deployments to test or acceptance environments for validation.</span></span>

## <a name="architecture"></a><span data-ttu-id="f807c-118">아키텍처</span><span class="sxs-lookup"><span data-stu-id="f807c-118">Architecture</span></span>

![Jenkins, Azure Container Registry 및 Azure Kubernetes Service를 사용하는 DevOps 시나리오와 관련된 Azure 구성 요소 아키텍처에 대한 개요][architecture]

<span data-ttu-id="f807c-120">이 시나리오에서는 Node.js 웹 애플리케이션 및 데이터베이스 백 엔드용 DevOps 파이프라인에 대해 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-120">This scenario covers a DevOps pipeline for a Node.js web application and database back end.</span></span> <span data-ttu-id="f807c-121">시나리오를 통한 데이터 흐름은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-121">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="f807c-122">개발자는 Node.js 웹 애플리케이션 소스 코드를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-122">A developer makes changes to the Node.js web application source code.</span></span>
2. <span data-ttu-id="f807c-123">코드 변경은 GitHub와 같은 소스 제어 리포지토리에 커밋됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-123">The code change is committed to a source control repository, such as GitHub.</span></span>
3. <span data-ttu-id="f807c-124">CI 프로세스를 시작하기 위해 GitHub 웹후크에서 Jenkins 프로젝트 빌드를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-124">To start the continuous integration (CI) process, a GitHub webhook triggers a Jenkins project build.</span></span>
4. <span data-ttu-id="f807c-125">Jenkins 빌드 작업은 Azure Kubernetes Service의 동적 빌드 에이전트를 사용하여 컨테이너 빌드 프로세스를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-125">The Jenkins build job uses a dynamic build agent in Azure Kubernetes Service to perform a container build process.</span></span>
5. <span data-ttu-id="f807c-126">컨테이너 이미지는 소스 제어의 코드에서 만들어진 다음, Azure Container Registry로 푸시됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-126">A container image is created from the code in source control, and is then pushed to an Azure Container Registry.</span></span>
6. <span data-ttu-id="f807c-127">Jenkins는 CD를 통해 업데이트된 이 컨테이너 이미지를 Kubernetes 클러스터에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-127">Through continuous deployment (CD), Jenkins deploys this updated container image to the Kubernetes cluster.</span></span>
7. <span data-ttu-id="f807c-128">Node.js 웹 애플리케이션은 Cosmos DB를 백 엔드로 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-128">The Node.js web application uses Cosmos DB as its back end.</span></span> <span data-ttu-id="f807c-129">Cosmos DB와 Azure Kubernetes Service는 모두 Azure Monitor에 메트릭을 보고합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-129">Both Cosmos DB and Azure Kubernetes Service report metrics to Azure Monitor.</span></span>
8. <span data-ttu-id="f807c-130">Grafana 인스턴스는 Azure Monitor의 데이터를 기반으로 하여 애플리케이션 성능의 시각적 대시보드를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-130">A Grafana instance provides visual dashboards of the application performance based on the data from Azure Monitor.</span></span>

### <a name="components"></a><span data-ttu-id="f807c-131">구성 요소</span><span class="sxs-lookup"><span data-stu-id="f807c-131">Components</span></span>

- <span data-ttu-id="f807c-132">[Jenkins][jenkins]는 Azure 서비스와 통합하여 CI 및 CD를 수행할 수 있게 하는 오픈 소스 자동화 서버입니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-132">[Jenkins][jenkins] is an open-source automation server that can integrate with Azure services to enable continuous integration (CI) and continuous deployment (CD).</span></span> <span data-ttu-id="f807c-133">이 시나리오에서 Jenkins는 소스 제어에 대한 커밋에 따라 새 컨테이너 이미지를 만들도록 오케스트레이션하고, Azure Container Registry에 해당 이미지를 푸시한 다음, Azure Kubernetes Service에서 애플리케이션 인스턴스를 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-133">In this scenario, Jenkins orchestrates the creation of new container images based on commits to source control, pushes those images to Azure Container Registry, then updates application instances in Azure Kubernetes Service.</span></span>
- <span data-ttu-id="f807c-134">[Azure Linux Virtual Machines][docs-virtual-machines]는 Jenkins 및 Grafana 인스턴스를 실행하는 데 사용되는 IaaS 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-134">[Azure Linux Virtual Machines][docs-virtual-machines] is the IaaS platform used to run the Jenkins and Grafana instances.</span></span>
- <span data-ttu-id="f807c-135">[Azure Container Registry][docs-acr]는 Azure Kubernetes Service 클러스터에서 사용되는 컨테이너 이미지를 저장하고 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-135">[Azure Container Registry][docs-acr] stores and manages container images that are used by the Azure Kubernetes Service cluster.</span></span> <span data-ttu-id="f807c-136">이미지는 안전하게 저장되며, Azure 플랫폼을 통해 다른 지역으로 복제하여 배포 시간을 단축할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-136">Images are securely stored, and can be replicated to other regions by the Azure platform to speed up deployment times.</span></span>
- <span data-ttu-id="f807c-137">[Azure Kubernetes Service][docs-aks]는 컨테이너 오케스트레이션에 대한 전문 지식이 없어도 컨테이너화된 애플리케이션을 배포하고 관리할 수 있는 관리되는 Kubernetes 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-137">[Azure Kubernetes Service][docs-aks] is a managed Kubernetes platform that lets you deploy and manage containerized applications without container orchestration expertise.</span></span> <span data-ttu-id="f807c-138">호스팅되는 Kubernetes 서비스인 Azure는 상태 모니터링 및 유지 관리 같은 중요 작업을 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-138">As a hosted Kubernetes service, Azure handles critical tasks like health monitoring and maintenance for you.</span></span>
- <span data-ttu-id="f807c-139">[Azure Cosmos DB][docs-cosmos-db]는 요구 사항에 맞게 다양한 데이터베이스 및 일관성 모델 중에서 선택할 수 있는 전역으로 분산된 다중 모델 데이터베이스입니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-139">[Azure Cosmos DB][docs-cosmos-db] is a globally distributed, multi-model database that allows you to choose from various database and consistency models to suit your needs.</span></span> <span data-ttu-id="f807c-140">Cosmos DB를 사용하면 데이터를 전역으로 복제할 수 있으며, 배포 및 구성할 클러스터 관리 또는 복제 구성 요소가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-140">With Cosmos DB, your data can be globally replicated, and there is no cluster management or replication components to deploy and configure.</span></span>
- <span data-ttu-id="f807c-141">[Azure Monitor][docs-azure-monitor]는 성능을 추적하고, 보안을 유지하며, 추세를 파악하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-141">[Azure Monitor][docs-azure-monitor] helps you track performance, maintain security, and identify trends.</span></span> <span data-ttu-id="f807c-142">Monitor에서 얻은 메트릭은 Grafana와 같은 다른 리소스 및 도구에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-142">Metrics obtained by Monitor can be used by other resources and tools, such as Grafana.</span></span>
- <span data-ttu-id="f807c-143">[Grafana][grafana]는 메트릭을 쿼리하고, 시각화하고, 경고하고, 이해할 수 있는 오픈 소스 솔루션입니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-143">[Grafana][grafana] is an open-source solution to query, visualize, alert, and understand metrics.</span></span> <span data-ttu-id="f807c-144">Azure Monitor용 데이터 원본 플러그 인을 사용하면 Grafana가 Azure Kubernetes Service에서 실행되고 Cosmos DB를 사용하는 애플리케이션의 성능을 모니터링하는 시각적 대시보드를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-144">A data source plugin for Azure Monitor allows Grafana to create visual dashboards to monitor the performance of your applications running in Azure Kubernetes Service and using Cosmos DB.</span></span>

### <a name="alternatives"></a><span data-ttu-id="f807c-145">대안</span><span class="sxs-lookup"><span data-stu-id="f807c-145">Alternatives</span></span>

- <span data-ttu-id="f807c-146">[Azure Pipelines][azure-pipelines]를 사용하면 모든 앱에 대한 지속적인 통합(CI), 테스트 및 배포(CD) 파이프라인을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-146">[Azure Pipelines][azure-pipelines] help you implement a continuous integration (CI), test, and deployment (CD) pipeline for any app.</span></span>
- <span data-ttu-id="f807c-147">[Kubernetes][kubernetes]는 클러스터에 대해 더 자세히 제어하려는 경우에 관리 서비스 대신 Azure VM에서 직접 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-147">[Kubernetes][kubernetes] can be run directly on Azure VMs instead of via a managed service if you would like more control over the cluster.</span></span>
- <span data-ttu-id="f807c-148">[Service Fabric][service-fabric]은 AKS를 대체할 수 있는 대체 컨테이너 오케스트레이터입니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-148">[Service Fabric][service-fabric] is another alternate container orchestrator that can replace AKS.</span></span>

## <a name="considerations"></a><span data-ttu-id="f807c-149">고려 사항</span><span class="sxs-lookup"><span data-stu-id="f807c-149">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="f807c-150">가용성</span><span class="sxs-lookup"><span data-stu-id="f807c-150">Availability</span></span>

<span data-ttu-id="f807c-151">애플리케이션 성능을 모니터링하고 문제를 보고하기 위해 이 시나리오에서는 시각적 대시보드에서 Azure Monitor와 Grafana를 결합합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-151">To monitor your application performance and report on issues, this scenario combines Azure Monitor with Grafana for visual dashboards.</span></span> <span data-ttu-id="f807c-152">이러한 도구를 사용하면 코드를 업데이트해야 하는 성능 문제를 모니터링하고 문제를 해결할 수 있습니다. 그런 다음, CI/CD 파이프라인을 통해 모두 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-152">These tools let you monitor and troubleshoot performance issues that may require code updates, which can all then be deployed with the CI/CD pipeline.</span></span>

<span data-ttu-id="f807c-153">Azure Kubernetes Service 클러스터의 일부인 부하 분산 장치는 애플리케이션을 실행하는 하나 이상의 컨테이너(포드)에 애플리케이션 트래픽을 분산시킵니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-153">As part of the Azure Kubernetes Service cluster, a load balancer distributes application traffic to one or more containers (pods) that run your application.</span></span> <span data-ttu-id="f807c-154">Kubernetes에서 컨테이너화된 애플리케이션을 실행하는 이 접근 방식은 고객에게 고가용성 인프라를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-154">This approach to running containerized applications in Kubernetes provides a highly available infrastructure for your customers.</span></span>

### <a name="scalability"></a><span data-ttu-id="f807c-155">확장성</span><span class="sxs-lookup"><span data-stu-id="f807c-155">Scalability</span></span>

<span data-ttu-id="f807c-156">Azure Kubernetes Service를 사용하면 애플리케이션의 요구 사항에 맞게 클러스터 노드 수를 크기 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-156">Azure Kubernetes Service lets you scale the number of cluster nodes to meet the demands of your applications.</span></span> <span data-ttu-id="f807c-157">애플리케이션이 증가함에 따라 서비스를 실행하는 Kubernetes 노드의 수를 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-157">As your application increases, you can scale out the number of Kubernetes nodes that run your service.</span></span>

<span data-ttu-id="f807c-158">애플리케이션 데이터는 전역으로 크기 조정할 수 있는 전역 분산형 다중 모델 데이터베이스인 Azure Cosmos DB에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-158">Application data is stored in Azure Cosmos DB, a globally distributed, multi-model database that can scale globally.</span></span> <span data-ttu-id="f807c-159">Cosmos DB는 기존 데이터베이스 구성 요소와 마찬가지로 인프라의 크기를 조정하기 위한 요구 사항을 추상화하고, 고객의 요구를 충족하기 위해 Cosmos DB를 전역적으로 복제하도록 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-159">Cosmos DB abstracts the need to scale your infrastructure as with traditional database components, and you can choose to replicate your Cosmos DB globally to meet the demands of your customers.</span></span>

<span data-ttu-id="f807c-160">다른 확장성 항목에 대해서는 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f807c-160">For other scalability topics, see the [scalability checklist][scalability] available in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="f807c-161">보안</span><span class="sxs-lookup"><span data-stu-id="f807c-161">Security</span></span>

<span data-ttu-id="f807c-162">공격 공간을 최소화하기 위해 이 시나리오에서는 Jenkins VM 인스턴스가 HTTP를 통해 노출되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-162">To minimize the attack footprint, this scenario does not expose the Jenkins VM instance over HTTP.</span></span> <span data-ttu-id="f807c-163">Jenkins와 상호 작용해야 하는 모든 관리 작업의 경우 로컬 컴퓨터에서 SSH 터널을 사용하여 보안 원격 연결을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-163">For any management tasks that require you to interact with Jenkins, you create a secure remote connection using an SSH tunnel from your local machine.</span></span> <span data-ttu-id="f807c-164">Jenkins 및 Grafana VM 인스턴스에는 SSH 공개 키 인증만 허용됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-164">Only SSH public key authentication is allowed for the Jenkins and Grafana VM instances.</span></span> <span data-ttu-id="f807c-165">암호 기반 로그인은 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-165">Password-based logins are disabled.</span></span> <span data-ttu-id="f807c-166">자세한 내용은 [Azure에서 Jenkins 서버 실행](./jenkins.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f807c-166">For more information, see [Run a Jenkins server on Azure](./jenkins.md).</span></span>

<span data-ttu-id="f807c-167">자격 증명과 권한을 분리하기 위해 이 시나리오에서는 전용 Azure AD(Active Directory) 서비스 사용자를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-167">For separation of credentials and permissions, this scenario uses a dedicated Azure Active Directory (AD) service principal.</span></span> <span data-ttu-id="f807c-168">이 서비스 사용자에 대한 자격 증명은 Jenkins에서 보안 자격 증명 개체로 저장되어 스크립트 또는 빌드 파이프라인 내에서 직접 노출되거나 볼 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-168">The credentials for this service principal are stored as a secure credential object in Jenkins so that they are not directly exposed and visible within scripts or the build pipeline.</span></span>

<span data-ttu-id="f807c-169">보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f807c-169">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="f807c-170">복원력</span><span class="sxs-lookup"><span data-stu-id="f807c-170">Resiliency</span></span>

<span data-ttu-id="f807c-171">이 시나리오에서는 애플리케이션에 Azure Kubernetes Service를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-171">This scenario uses Azure Kubernetes Service for your application.</span></span> <span data-ttu-id="f807c-172">Kubernetes에는 문제가 있는 경우 컨테이너(포드)를 모니터링하고 다시 시작하는 복원력 있는 구성 요소가 기본적으로 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-172">Built into Kubernetes are resiliency components that monitor and restart the containers (pods) if there is an issue.</span></span> <span data-ttu-id="f807c-173">여러 Kubernetes 노드를 실행하는 것과 결합하여 애플리케이션에서 사용할 수 없는 노드 또는 노드를 허용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-173">Combined with running multiple Kubernetes nodes, your application can tolerate a pod or node being unavailable.</span></span>

<span data-ttu-id="f807c-174">복원 력 있는 솔루션 디자인에 대 한 일반적인 지침을 참조 하세요 [신뢰할 수 있는 Azure 응용 프로그램 디자인](../../reliability/index.md)합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-174">For general guidance on designing resilient solutions, see [Designing reliable Azure applications](../../reliability/index.md).</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="f807c-175">시나리오 배포</span><span class="sxs-lookup"><span data-stu-id="f807c-175">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f807c-176">필수 조건</span><span class="sxs-lookup"><span data-stu-id="f807c-176">Prerequisites</span></span>

- <span data-ttu-id="f807c-177">기존 Azure 계정이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-177">You must have an existing Azure account.</span></span> <span data-ttu-id="f807c-178">Azure 구독이 아직 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-178">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

- <span data-ttu-id="f807c-179">SSH 공개 키 쌍이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-179">You need an SSH public key pair.</span></span> <span data-ttu-id="f807c-180">공개 키 쌍을 만드는 방법에 대한 단계는 [Linux VM용 SSH 키 쌍 만들기 및 사용][sshkeydocs]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f807c-180">For steps on how to create a public key pair, see [Create and use an SSH key pair for Linux VMs][sshkeydocs].</span></span>

- <span data-ttu-id="f807c-181">서비스 및 리소스의 인증에는 Azure AD(Active Directory) 서비스 사용자가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-181">You need an Azure Active Directory (AD) service principal for the authentication of service and resources.</span></span> <span data-ttu-id="f807c-182">필요한 경우 [az ad sp create-for-rbac][createsp]를 사용하여 서비스 사용자를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-182">If needed, you can create a service principal with [az ad sp create-for-rbac][createsp]</span></span>

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    <span data-ttu-id="f807c-183">이 명령의 출력에서 나온 *appId* 및 *password*를 적어 둡니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-183">Make a note of the *appId* and *password* in the output from this command.</span></span> <span data-ttu-id="f807c-184">시나리오를 배포할 때 이러한 값을 템플릿에 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-184">You provide these values to the template when you deploy the scenario.</span></span>

### <a name="walk-through"></a><span data-ttu-id="f807c-185">연습</span><span class="sxs-lookup"><span data-stu-id="f807c-185">Walk-through</span></span>

<span data-ttu-id="f807c-186">Azure Resource Manager 템플릿을 사용하여 이 시나리오를 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-186">To deploy this scenario with an Azure Resource Manager template, perform the following steps.</span></span>

<!-- markdownlint-disable MD033 -->

1. <span data-ttu-id="f807c-187">**Azure에 배포** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-187">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="f807c-188">Azure Portal에서 템플릿 배포가 열릴 때까지 기다린 후에 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-188">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   - <span data-ttu-id="f807c-189">리소스 그룹 **새로 만들기**를 선택한 다음, 텍스트 상자에서 이름(예: *myAKSDevOpsScenario*)을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-189">Choose to **Create new** resource group, then provide a name such as *myAKSDevOpsScenario* in the text box.</span></span>
   - <span data-ttu-id="f807c-190">**위치** 드롭다운 상자에서 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-190">Select a region from the **Location** drop-down box.</span></span>
   - <span data-ttu-id="f807c-191">`az ad sp create-for-rbac` 명령에서 서비스 사용자 앱 ID 및 암호를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-191">Enter your service principal app ID and password from the `az ad sp create-for-rbac` command.</span></span>
   - <span data-ttu-id="f807c-192">Jenkins 인스턴스 및 Grafana 콘솔에 대한 사용자 이름과 보안 암호를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-192">Provide a username and secure password for the Jenkins instance and Grafana console.</span></span>
   - <span data-ttu-id="f807c-193">Linux VM에 대한 로그인을 보호하기 위한 SSH 키를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-193">Provide an SSH key to secure logins to the Linux VMs.</span></span>
   - <span data-ttu-id="f807c-194">사용 약관을 검토한 다음, **위에 명시된 사용 약관에 동의함**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-194">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   - <span data-ttu-id="f807c-195">**구매** 단추를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-195">Select the **Purchase** button.</span></span>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="f807c-196">배포가 완료되는 데 15-20분이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-196">It can take 15-20 minutes for the deployment to complete.</span></span>

## <a name="pricing"></a><span data-ttu-id="f807c-197">가격</span><span class="sxs-lookup"><span data-stu-id="f807c-197">Pricing</span></span>

<span data-ttu-id="f807c-198">이 시나리오를 실행하는 데 들어가는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-198">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="f807c-199">특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 트래픽에 맞게 적절한 변수를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-199">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="f807c-200">저장할 컨테이너 이미지 및 Kubernetes 노드의 수를 기준으로 다음 세 가지 샘플 비용 프로필을 제공하여 애플리케이션을 실행했습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-200">We have provided three sample cost profiles based on the number of container images to store and Kubernetes nodes to run your applications.</span></span>

- <span data-ttu-id="f807c-201">[소형][small-pricing]: 이 가격 책정 예제는 매월 1,000개의 컨테이너 빌드와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-201">[Small][small-pricing]: this pricing example correlates to 1000 container builds per month.</span></span>
- <span data-ttu-id="f807c-202">[중형][medium-pricing]: 이 가격 책정 예제는 매월 100,000개의 컨테이너 빌드와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-202">[Medium][medium-pricing]: this pricing example correlates to 100,000 container builds per month.</span></span>
- <span data-ttu-id="f807c-203">[대형][large-pricing]: 이 가격 책정 예제는 매월 1,000,000개의 컨테이너 빌드와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-203">[Large][large-pricing]: this pricing example correlates to 1,000,000 container builds per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="f807c-204">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="f807c-204">Related resources</span></span>

<span data-ttu-id="f807c-205">이 시나리오에서는 Azure Container Registry와 Azure Kubernetes Service를 사용하여 컨테이너 기반 애플리케이션을 저장하고 실행했습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-205">This scenario used Azure Container Registry and Azure Kubernetes Service to store and run a container-based application.</span></span> <span data-ttu-id="f807c-206">Azure Container Instances는 오케스트레이션 구성 요소를 프로비전하지 않고 컨테이너 기반 애플리케이션을 실행하는 데에도 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f807c-206">Azure Container Instances can also be used to run container-based applications, without having to provision any orchestration components.</span></span> <span data-ttu-id="f807c-207">자세한 내용은 [Azure Container Instances 개요][docs-aci]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f807c-207">For more information, see [Azure Container Instances overview][docs-aci].</span></span>

<!-- links -->
[architecture]: ./media/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[docs-aci]: /azure/container-instances/container-instances-overview
[docs-acr]: /azure/container-registry/container-registry-intro
[docs-aks]: /azure/aks/intro-kubernetes
[docs-azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[docs-cosmos-db]: /azure/cosmos-db/introduction
[docs-virtual-machines]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[azure-pipelines]: /azure/devops/pipelines
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64