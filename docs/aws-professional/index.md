---
title: AWS 전문가를 위한 Azure
titleSuffix: Azure Architecture Center
description: Microsoft Azure 계정, 플랫폼 및 서비스의 기본 사항에 대해 알아봅니다. 또한 AWS 및 Azure 플랫폼 간의 주요 유사점과 차이점에 대해 알아봅니다. Azure에서 AWS 환경을 활용합니다.
keywords: AWS 전문가, Azure 비교, AWS 비교, azure와 aws의 차이점, azure 및 aws
author: lbrader
ms.date: 09/19/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 6e44953cccf39cc40be775f4043bcf8b1b890dae
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641164"
---
# <a name="azure-for-aws-professionals"></a>AWS 전문가를 위한 Azure

이 문서는 AWS(Amazon Web Services) 전문가가 Microsoft Azure 계정, 플랫폼 및 서비스의 기본 사항을 이해하는 데 도움을 줍니다. 또한 AWS 및 Azure 플랫폼 간의 주요 유사점과 차이점에 대해 다룹니다.

다음 내용을 배웁니다.

- Azure에서 계정 및 리소스가 구성되는 방식.
- Azure에서 사용 가능한 솔루션이 구성되는 방식.
- 주요 Azure 서비스가 AWS 서비스와 다른 점.

Azure와 AWS는 시간에 관계없이 각 기능을 독립적으로 개발했기 때문에 각 기능의 구현 및 디자인이 큰 차이를 보입니다.

## <a name="overview"></a>개요

AWS와 마찬가지로, Microsoft Azure는 핵심 계산, 스토리지, 데이터베이스 및 네트워킹 서비스 세트를 중심으로 빌드됩니다. 두 플랫폼이 제공하는 제품과 서비스의 기본적인 사항은 동일한 경우가 많습니다. AWS와 Azure 모두 Windows 또는 Linux 호스트를 기반으로 고가용성 솔루션을 빌드할 수 있습니다. 따라서 Linux 및 OSS 기술을 사용하여 개발하는 데 익숙한 분들은 어느 플랫폼으로도 작업을 수행할 수 있습니다.

두 플랫폼의 기능은 유사하지만, 기능을 제공하는 리소스는 종종 다르게 구성됩니다. 솔루션을 빌드하는 데 필요한 서비스 간에 항상 정확한 일대일 관계가 성립하는 것은 아닙니다. 특정 서비스가 한 플랫폼에만 제공되고 다른 플랫폼에는 제공되지 않는 경우가 있습니다. [Azure 및 AWS 서비스 비교 차트](./services.md)를 참조하세요.

## <a name="accounts-and-subscriptions"></a>계정 및 구독

조직의 규모와 요구 사항에 따라 여러 가격 책정 옵션을 사용하여 Azure 서비스를 구입할 수 있습니다. 자세한 내용은 [가격 책정 개요](https://azure.microsoft.com/pricing/) 페이지를 참조하세요.

[Azure 구독](/azure/virtual-machines/linux/infrastructure-example)은 대금 청구 및 권한 관리 책임이 있는 소유자가 할당된 리소스 그룹입니다. AWS 계정 하에서 생성되는 모든 리소스가 해당 계정에 연결되는 AWS와는 달리, 구독이 소유자 계정에 독립적으로 존재하며 필요에 따라 새로운 소유자에게 할당할 수 있습니다.

<!-- markdownlint-disable MD033 -->

![AWS 계정과 Azure 구독의 구조 및 소유권 비교](./images/azure-aws-account-compare.png "AWS 계정과 Azure 구독의 구조 및 소유권 비교")
<br/>*AWS 계정과 Azure 구독의 구조 및 소유권 비교*
<br/><br/>

<!-- markdownlint-enable MD033 -->

구독에는 세 가지 유형의 관리자 계정이 할당됩니다.

- **계정 관리자** 구독에 사용된 리소스 요금이 청구되는 구독 소유자 및 계정입니다. 구독 소유권을 양도하여 계정 관리자를 변경하는 것만 가능합니다.

- **서비스 관리자** 이 계정은 구독에서 리소스를 만들고 관리할 수 있는 권한을 갖고 있지만 대금 청구를 처리하지는 않습니다. 기본적으로 계정 관리자 및 서비스 관리자는 동일한 계정에 할당됩니다. 계정 관리자는 구독의 기술 및 운영적 측면을 관리할 별도의 사용자를 서비스 관리자 계정에 할당할 수 있습니다. 구독당 서비스 관리자는 한 명만 있습니다.

- **공동 관리자** 한 구독에 공동 관리자 계정을 여러 개 할당할 수 있습니다. 공동 관리자는 서비스 관리자를 변경할 수 없지만 구독 리소스와 사용자를 완전하게 제어할 수 있습니다.

AWS에서 IAM 사용자 및 그룹에 권한을 부여하는 방법과 비슷하게, 구독 수준 아래에서 사용자 역할 및 개별 권한을 특정 리소스에 할당할 수도 있습니다. Azure에서는 모든 사용자 계정이 Microsoft 계정 또는 조직 계정(Azure Active Directory를 통해 관리되는 계정)과 연결됩니다.

AWS 계정과 마찬가지로, 구독의 기본 서비스 할당량 및 제한이 있습니다. 전체 제한 목록은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건](/azure/azure-subscription-service-limits)을 참조하세요.
[관리 포털에서 지원 요청을 제출](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/)하여 이러한 제한을 최대값까지 높일 수 있습니다.

### <a name="see-also"></a>참고 항목

- [Azure 관리자 역할을 추가 또는 변경하는 방법](/azure/billing/billing-add-change-azure-subscription-administrator)

- [Azure 청구 송장 및 일간 사용 현황 데이터를 다운로드하는 방법](/azure/billing/billing-download-azure-invoice-daily-usage-date)

## <a name="resource-management"></a>리소스 관리

Azure에서 말하는 "리소스"라는 용어는 AWS와 똑같은 의미로 사용됩니다. 즉, 모든 컴퓨팅 인스턴스, 스토리지 개체, 네트워킹 디바이스 또는 플랫폼 내에서 만들거나 구성할 수 있는 기타 엔터티를 의미합니다.

Azure 리소스는 두 가지 모델, 즉 [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) 또는 기존 Azure [클래식 배포 모델](/azure/azure-resource-manager/resource-manager-deployment-model) 중 하나를 사용하여 배포 및 관리됩니다. 모든 새 리소스는 Resource Manager 모델을 사용하여 만듭니다.

### <a name="resource-groups"></a>리소스 그룹

Azure와 AWS 둘 다 VM, 저장소, 가상 네트워킹 디바이스 등의 리소스를 구성하는 "리소스 그룹"이라는 엔터티를 갖고 있습니다. 그러나 [Azure 리소스 그룹](/azure/virtual-machines/windows/infrastructure-example)을 AWS 리소스 그룹과 직접 비교하는 것은 무리입니다.

AWS는 한 리소스를 여러 리소스 그룹에 태깅할 수 있지만 Azure 리소스는 항상 한 리소스 그룹에만 연결됩니다. 한 리소스 그룹에서 생성된 리소스를 다른 그룹으로 이동할 수 있지만, 한 번에 한 리소스 그룹에만 소속될 수 있습니다. 리소스 그룹은 Azure Resource Manager가 사용하는 기본적인 그룹화 방법입니다.

[태그](/azure/azure-resource-manager/resource-group-using-tags)를 사용하여 리소스를 구성할 수도 있습니다. 태그는 리소스 그룹 멤버 자격에 관계없이 구독의 리소스를 그룹화할 수 있는 키-값 쌍입니다.

### <a name="management-interfaces"></a>관리 인터페이스

Azure는 리소스를 관리하는 여러 방법을 제공합니다.

- [웹 인터페이스](/azure/azure-resource-manager/resource-group-portal).
    AWS 대시보드와 마찬가지로, Azure Portal은 Azure 리소스에 대한 완전한 웹 기반 관리 인터페이스를 제공합니다.

- [REST API](/rest/api/).
    Azure Resource Manager REST API는 Azure Portal에서 사용 가능한 대부분의 기능에 대해 프로그래밍 방식의 액세스를 제공합니다.

- [명령줄](/azure/azure-resource-manager/cli-azure-resource-manager).
    Azure CLI 2.0 도구는 Azure 리소스를 만들고 관리할 수 있는 명령줄 인터페이스를 제공합니다. Azure CLI는 [Windows, Linux 및 Mac OS](https://aka.ms/azurecli2)에서 사용할 수 있습니다.

- [PowerShell](/azure/azure-resource-manager/powershell-azure-resource-manager).
    PowerShell용 Azure 모듈을 사용하면 스크립트를 사용하여 자동화 관리 작업을 실행할 수 있습니다. PowerShell은 [Windows, Linux 및 Mac OS](https://github.com/PowerShell/PowerShell)에서 사용할 수 있습니다.

- [템플릿](/azure/azure-resource-manager/resource-group-authoring-templates).
    Azure Resource Manager 템플릿은 AWS CloudFormation 서비스와 비슷한 JSON 템플릿 기반 리소스 관리 기능을 제공합니다.

이러한 각 인터페이스에서 리소스 그룹은 Azure 리소스를 만들고 배포하고 수정하는 데 있어서 핵심적인 역할을 합니다. CloudFormation 배포 시 "스택"이 AWS 리소스 그룹화에서 수행하는 역할과 비슷합니다.

이러한 인터페이스의 구문 및 구조는 AWS와 다르지만, 비슷한 기능을 제공합니다. 또한 AWS에 사용되는 [Hashicorp's Terraform](https://www.terraform.io/docs/providers/azurerm/) 및 [Netflix Spinnaker](https://www.spinnaker.io/) 같은 여러 타사 관리 도구가 Azure에서도 제공됩니다.

<!-- markdownlint-disable MD024 -->

### <a name="see-also"></a>참고 항목

- [Azure 리소스 그룹 지침](/azure/azure-resource-manager/resource-group-overview#resource-groups)

## <a name="regions-and-zones-high-availability"></a>지역 및 영역(고가용성)

오류는 해당 영향의 범위가 다를 수 있습니다. 실패한 디스크와 같은 일부 하드웨어 오류는 단일 호스트 컴퓨터에 영향을 줄 수 있습니다. 실패한 네트워크 스위치는 전체 서버 랙에 영향을 줄 수 있습니다. 데이터 센터의 전원 손실 등 전체 데이터 센터를 방해하는 오류는 일반적이지 않습니다. 가끔 전체 지역을 사용할 수 없게 될 수 있습니다.

중복성을 통해 애플리케이션을 복원력 있게 만들 수 있습니다. 그러나 애플리케이션을 디자인할 때 이러한 중복성에 대해 계획해야 합니다. 또한 필요한 중복성 수준은 비즈니스 요구 사항에 따라 달라집니다. &mdash; 지역 가동 중단으로부터 보호하기 위해 일부 애플리케이션에 지역 간 중복성이 필요합니다. 일반적으로 중복성과 안정성 및 비용과 복잡성 간에 균형을 조절해야 합니다.

AWS에서는 한 영역이 두 개 이상의 가용성 영역으로 나뉩니다. 가용성 영역은 지리적 지역에 물리적으로 격리된 데이터 센터와 일치합니다. Azure에는 **가용성 집합**, **가용성 영역** 및 **쌍을 이루는 지역** 등 각 애플리케이션 오류 수준에서 중복되는 애플리케이션을 만드는 여러 기능이 있습니다.

![중복](../resiliency/images/redundancy.svg)

다음 표에서는 각 옵션을 요약합니다.

| &nbsp; | 가용성 집합 | 가용성 영역 | 쌍을 이루는 지역 |
|--------|------------------|-------------------|---------------|
| 오류의 범위 | 랙 | 데이터 센터 | 지역 |
| 요청 라우팅 | Load Balancer | 영역 간 부하 분산 장치 | Traffic Manager |
| 네트워크 대기 시간 | 매우 낮음 | 낮음 | 중간부터 높음 |
| 가상 네트워킹  | VNet | VNet | 지역 간 VNet 피어링 |

### <a name="availability-sets"></a>가용성 집합

디스크 또는 네트워크 전환이 실패한 경우 하드웨어 오류로부터 보호하려면 가용성 집합에 둘 이상의 VM을 배포합니다. 가용성 집합은 공통 전원 소스 및 네트워크 스위치를 공유하는 두 개 이상의 *장애 도메인*으로 구성됩니다. 가용성 집합의 VM은 장애 도메인에 분산되어 있으므로 하드웨어 오류가 하나의 장애 도메인에 영향을 주는 경우 네트워크 트래픽은 다른 오류 도메인에서 VM을 라우팅할 수 있습니다. 가용성 집합에 대한 자세한 내용은 [Azure에서 Windows 가상 머신의 가용성 관리](/azure/virtual-machines/windows/manage-availability)를 참조하세요.

가용성 집합에 추가되는 VM 인스턴스에는 [업데이트 도메인](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)이 할당됩니다. 업데이트 도메인은 동시에 계획된 유지 관리 이벤트를 수행하도록 설정된 VM 그룹입니다. VM을 여러 업데이트 도메인에 분산하면 계획된 업데이트 및 패치 이벤트가 지정된 시간에 이러한 VM의 하위 집합에만 영향을 줍니다.

각 역할의 한 인스턴스가 정상적으로 작동하려면 애플리케이션의 인스턴스 역할을 통해 가용성 집합을 구성해야 합니다. 예를 들어 3계층 웹 애플리케이션에서는 프런트 엔드, 애플리케이션 및 데이터 계층에 대한 별도의 가용성 집합을 만듭니다.

![각 애플리케이션 역할에 대한 Azure 가용성 집합](./images/three-tier-example.png "각 애플리케이션 역할에 대한 Azure 가용성 집합")

### <a name="availability-zones"></a>가용성 영역

[가용성 영역](/azure/availability-zones/az-overview)은 Azure 지역 내에서 물리적으로 별도의 영역입니다. 각 가용성 영역에는 고유한 소스, 네트워크 및 냉각 장치가 있습니다. 가용성 영역 간에 VM을 배포하면 데이터 센터 전체의 오류로부터 애플리케이션을 보호할 수 있습니다.

### <a name="paired-regions"></a>쌍을 이루는 지역

지역 가동 중단으로부터 애플리케이션을 보호하려면 인터넷 트래픽을 서로 다른 지역에 배포하는 [Azure Traffic Manager][traffic-manager]를 사용하여 애플리케이션을 여러 지역에 배포할 수 있습니다. 각 Azure 지역은 다른 지역과 쌍을 이룹니다. 이러한 지역은 함께 [지역 쌍][paired-regions]을 구성합니다. 브라질 남부를 제외하고 지역 쌍은 세금 및 법률 집행 관할 구역의 데이터 상주 요구 사항을 충족하기 위해 동일한 지리적 위치 내에 위치합니다.

데이터 센터가 물리적으로 떨어져 있지만 비교적 가까운 영역에 있는 가용성 영역과는 달리, 쌍을 이루는 지역은 일반적으로 480킬로미터 이상 떨어져 있습니다. 이는 대규모 재해가 발생하더라도 쌍을 이루는 지역의 한 쪽 지역만 영향을 받게 하려는 의도입니다. 인접한 쌍은 데이터베이스 및 저장소 서비스 데이터를 동기화할 때 설정할 수 있으며, 쌍을 이루는 지역 중 한 번에 한 영역에서만 플랫폼 업데이트가 수행되도록 구성됩니다.

Azure [지역 중복 저장소](/azure/storage/common/storage-redundancy-grs)는 적절한 쌍을 이루는 지역에 자동으로 백업됩니다. 그 외 리소스의 경우 쌍을 이루는 지역을 사용하여 완전한 이중화 솔루션을 만든다는 것은 두 영역 모두에 솔루션 전체 복사본을 만든다는 의미입니다.

### <a name="see-also"></a>참고 항목

- [Azure에서 가상 머신의 영역 및 가용성](/azure/virtual-machines/linux/regions-and-availability)

- [Azure 애플리케이션의 고가용성](../resiliency/high-availability-azure-applications.md)

- [Azure 애플리케이션의 오류 및 재해 복구](../reliability/disaster-recovery.md)

- [Azure에서 Linux 가상 머신에 대한 계획된 유지 관리](/azure/virtual-machines/linux/maintenance-and-updates)

## <a name="services"></a>Services

플랫폼 간 서비스 맵핑 방법 목록은 [AWS에서 Azure 서비스 비교](./services.md)를 참조하십시오.

지역에 따라 일부 Azure 제품 및 서비스가 제공되지 않을 수도 있습니다. 자세한 내용은 [지역별 제품](https://azure.microsoft.com/global-infrastructure/services/) 페이지를 참조하세요. [서비스 수준 계약](https://azure.microsoft.com/support/legal/sla/) 페이지에서 각 Azure 제품 및 서비스의 작동 시간 보장 및 가동 중지 시간 크레딧 정책을 확인할 수 있습니다.

다음 단원에서는 AWS 및 Azure 플랫폼에서 자주 사용되는 기능과 서비스가 서로 어떻게 다른지 간략하게 설명합니다.

### <a name="compute-services"></a>컴퓨팅 서비스

#### <a name="ec2-instances-and-azure-virtual-machines"></a>EC2 인스턴스 및 Azure 가상 머신

AWS 인스턴스 형식과 Azure 가상 머신 크기는 비슷한 방식으로 분류되지만 RAM, CPU 및 저장소 기능에서 차이가 있습니다.

- [Amazon EC2 인스턴스 형식](https://aws.amazon.com/ec2/instance-types/)

- [Azure에서 가상 머신 크기(Windows)](/azure/virtual-machines/windows/sizes)

- [Azure에서 가상 머신 크기(Linux)](/azure/virtual-machines/linux/sizes)

초 단위로 요금이 청구되는 AWS와 마찬가지로 Azure 주문형 VM도 초 단위로 요금이 청구됩니다.

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>VM 디스크용 EBS 및 Azure Storage

Blob Storage에 상주하는 [데이터 디스크](/azure/virtual-machines/linux/about-disks-and-vhds)가 내구성이 우수한 Azure VM용 데이터 스토리지를 제공합니다. EC2 인스턴스가 EBS(Elastic Block Store)에 디스크 볼륨을 저장하는 방식과 비슷합니다. [Azure 임시 저장소](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) 역시 EC2 인스턴스 저장소(사용 후 삭제 저장소라고도 함)와 동일하게 대기 시간이 짧은 임시 읽기-쓰기 저장소를 VM에 제공합니다.

[Azure Premium Storage](/azure/virtual-machines/windows/premium-storage)를 사용하여 고성능 디스크 IO를 지원합니다. 이것은 AWS에서 제공하는 프로비전된 IOPS 스토리지 옵션과 비슷합니다.

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda, Azure Functions, Azure Web-Jobs 및 Azure Logic Apps

[Azure Functions](https://azure.microsoft.com/services/functions/)는 서버 없는 주문형 코드를 제공한다는 측면에서 AWS Lambda와 가장 비슷합니다. 그러나 Lambda 기능은 다른 Azure 서비스와도 겹칩니다.

- [WebJobs](/azure/app-service/web-sites-create-web-jobs)을 사용하면 예약된 작업 또는 지속적으로 실행되는 백그라운드 작업을 만들 수 있습니다.

- [Logic Apps](https://azure.microsoft.com/services/logic-apps/)에서는 통신, 통합 및 비즈니스 규칙 관리 서비스를 제공합니다.

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>자동 크기 조정, Azure VM 크기 조정 및 Azure App Service

Azure의 자동 크기 조정은 다음 두 서비스에서 처리합니다.

- [VM 확장 세트](/azure/virtual-machine-scale-sets/overview)를 사용하면 일련의 동일한 VM을 배포하고 관리할 수 있습니다. 인스턴스 수는 성능 요구 사항에 따라 자동으로 조정할 수 있습니다.

- [App Service 자동 크기 조정](/azure/app-service/web-sites-scale)에서는 Azure App Service 솔루션의 크기를 자동으로 조정하는 기능을 제공합니다.

#### <a name="container-service"></a>컨테이너 서비스

[Azure Kubernetes Service](/azure/aks/intro-kubernetes)는 Kubernetes를 통해 관리형 Docker 컨테이너를 지원합니다.

#### <a name="other-compute-services"></a>기타 컴퓨팅 서비스

Azure는 AWS와 약간 차이가 있는 여러 컴퓨팅 서비스를 제공합니다.

- [Azure Batch](/azure/batch/batch-technical-overview)를 사용하면 확장 가능한 가상 머신 컬렉션에서 계산 집약적인 작업을 관리할 수 있습니다.

- [Service Fabric](/azure/service-fabric/service-fabric-overview)은 확장 가능한 [마이크로서비스](/azure/service-fabric/service-fabric-overview-microservices) 솔루션을 개발하고 호스팅할 수 있는 플랫폼입니다.

#### <a name="see-also"></a>참고 항목

- [포털을 사용하여 Azure에서 Linux VM 만들기](/azure/virtual-machines/linux/quick-create-portal)

- [Azure 참조 아키텍처: Azure에서 Linux VM 실행](/azure/architecture/reference-architectures/n-tier/linux-vm)

- [Azure App Service에서 Node.js 웹앱 시작](/azure/app-service/app-service-web-get-started-nodejs)

- [Azure 참조 아키텍처: 기본 웹 애플리케이션](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app)

- [첫 번째 Azure Function 만들기](/azure/azure-functions/functions-create-first-azure-function)

### <a name="storage"></a>Storage

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS 및 Azure Storage

AWS 플랫폼에서 클라우드 저장소는 주로 세 가지 서비스로 분류됩니다.

- **S3(Simple Storage Services)** 인터넷 액세스가 가능한 API를 통해 데이터를 사용할 수 있는 기본 개체 스토리지입니다.

- **EBS(Elastic Block Store)** 단일 VM 액세스를 위한 블록 수준 스토리지입니다.

- **EFS(Elastic File System)** EC2 인스턴스 수천 개의 공유 스토리지로 사용되는 파일 스토리지입니다.

Azure Storage에서는 구독에 바인딩된 [스토리지 계정](/azure/storage/common/storage-quickstart-create-account)을 사용하여 다음과 같은 스토리지 서비스를 만들고 관리할 수 있습니다.

- [Blob Storage](/azure/storage/common/storage-quickstart-create-account)는 문서, 미디어 파일 또는 애플리케이션 설치 프로그램과 같은 모든 종류의 텍스트 또는 이진 데이터를 저장합니다. 개인 액세스에 대해 Blob Storage를 설정하거나 인터넷에 공개적으로 콘텐츠를 공유할 수 있습니다. Blob Storage는 AWS S3 및 EBS와 동일한 용도로 사용됩니다.
- [Table Storage](/azure/cosmos-db/table-storage-how-to-use-nodejs)는 구조화된 데이터 세트를 저장합니다. Table Storage는 신속한 개발과 대량 데이터에 대한 빠른 액세스를 가능하게 하는 NoSQL 키-특성 데이터 저장소입니다. AWS의 SimpleDB 및 DynamoDB 서비스와 비슷합니다.

- [Queue Storage](/azure/storage/queues/storage-nodejs-how-to-use-queues)는 워크플로 처리 및 클라우드 서비스 구성 요소 사이의 통신을 위한 메시지를 제공합니다.

- [File Storage](/azure/storage/files/storage-java-how-to-use-file-storage)는 표준 SMB(서버 메시지 블록) 프로토콜을 사용하는 기존 애플리케이션을 위한 공유 스토리지를 제공합니다. File Storage는 AWS 플랫폼의 EFS와 비슷한 방식으로 사용됩니다.

#### <a name="glacier-and-azure-storage"></a>Glacier 및 Azure Storage

[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier)는 AWS Glacier 스토리지 서비스에 해당합니다. 180일 이상 저장되면서 거의 액세스하지 않으며 몇 시간 동안의 검색 대기 시간도 용인할 수 있는 데이터를 대상으로 합니다.

자주 액세스하지 않지만 액세스할 때 즉시 사용 가능해야 하는 데이터의 경우 [Azure Cool Blob Storage 계층](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier)에서 표준 Blob 스토리지보다 저렴한 스토리지를 제공합니다. 이 저장소 계층은 AWS S3 - 자주 액세스하지 않는 저장소 서비스에 해당합니다.

#### <a name="see-also"></a>참고 항목

- [Microsoft Azure Storage 성능 및 확장성 검사 목록](/azure/storage/common/storage-performance-checklist)

- [Azure Storage 보안 가이드](/azure/storage/common/storage-security-guide)

- [CDN(콘텐츠 전송 네트워크) 사용에 대한 모범 사례](/azure/architecture/best-practices/cdn)

### <a name="networking"></a>네트워킹

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>Elastic Load Balancing, Azure Load Balancer 및 Azure Application Gateway

두 Elastic Load Balancing 서비스에 해당하는 Azure 서비스는 다음과 같습니다.

- [Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - AWS Classic Load Balancer와 동일한 기능을 제공하며, 네트워크 수준에서 여러 VM의 트래픽을 분산할 수 있습니다. 장애 조치(failover) 기능도 제공합니다.

- [Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - AWS Application Load Balancer와 비슷한 애플리케이션 수준 규칙 기반 라우팅을 제공합니다.

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53, Azure DNS 및 Azure Traffic Manager

AWS의 Route 53은 DNS 이름 관리 및 DNS 수준 트래픽 라우팅과 장애 조치(failover) 서비스를 모두 제공합니다. Azure에서 이러한 작업이 다음 두 서비스를 통해 처리됩니다.

- [Azure DNS](https://azure.microsoft.com/documentation/services/dns/)는 도메인 및 DNS 관리를 제공합니다.

- [Traffic Manager][traffic-manager]는 DNS 수준 트래픽 라우팅, 부하 분산 및 장애 조치(failover) 기능을 제공합니다.

#### <a name="direct-connect-and-azure-expressroute"></a>Direct Connect 및 Azure ExpressRoute

Azure는 [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) 서비스를 통해 AWS와 비슷한 사이트 간 전용 연결을 제공합니다. ExpressRoute를 사용하면 전용 개인 네트워크 연결을 사용하여 로컬 네트워크를 Azure 리소스에 직접 연결할 수 있습니다. 또한 Azure는 기존의 [사이트 간 VPN 연결](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)을 좀 더 저렴한 가격에 제공합니다.

#### <a name="see-also"></a>참고 항목

- [Azure Portal을 사용하여 가상 네트워크 만들기](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

- [Azure Virtual Network 계획 및 디자인](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

- [Azure 네트워크 보안 모범 사례](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>데이터베이스 서비스

#### <a name="rds-and-azure-relational-database-services"></a>RDS 및 Azure 관계형 데이터베이스 서비스

Azure의 AWS RDS(관계형 데이터베이스 서비스)에 해당하는 몇 가지 다른 관계형 데이터베이스 서비스를 제공합니다.

- [SQL Database](/azure/sql-database/sql-database-technical-overview)
- [Azure Database for MySQL](/azure/mysql/overview)
- [Azure Database for PostgreSQL](/azure/postgresql/overview)

[SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/) 및 [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) 같은 다른 데이터베이스 엔진은 Azure VM 인스턴스를 사용하여 배포할 수 있습니다.

AWS RDS의 비용은 CPU, RAM, 저장소, 네트워크 대역폭 등 인스턴스에서 사용하는 하드웨어 리소스의 양에 따라 결정됩니다. Azure 데이터베이스 서비스의 비용은 데이터베이스 크기, 동시 연결 수 및 처리량 수준에 따라 결정됩니다.

#### <a name="see-also"></a>참고 항목

- [Azure SQL Database 자습서](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

- [Azure Portal로 Azure SQL Database에 대한 지역에서 복제 구성](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

- [Azure Cosmos DB 소개: NoSQL JSON 데이터베이스](/azure/cosmos-db/sql-api-introduction)

- [Node.js에서 Azure Table Storage를 사용하는 방법](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>보안 및 ID

#### <a name="directory-service-and-azure-active-directory"></a>디렉터리 서비스 및 Azure Active Directory

Azure는 디렉터리 서비스를 다음과 같은 제품으로 나눠 놓았습니다.

- [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - 클라우드 기반 디렉터리 및 ID 관리 서비스입니다.

- [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - 파트너 관리형 ID로 회사 애플리케이션에 액세스할 수 있습니다.

- [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - 소비자 지향 애플리케이션에서 Single Sign-On 및 사용자 관리를 지원하는 서비스입니다.

- [Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - 호스팅된 도메인 컨트롤러 서비스로, Active Directory 호환 도메인 가입 및 사용자 관리 기능을 허용합니다.

#### <a name="web-application-firewall"></a>웹 애플리케이션 방화벽

[Application Gateway 웹 애플리케이션 방화벽](/azure/application-gateway/waf-overview) 외에도 [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) 같은 타사 공급업체의 웹 애플리케이션 방화벽을 사용할 수 있습니다.

#### <a name="see-also"></a>참고 항목

- [Microsoft Azure 보안 시작](/azure/security)

- [Azure Identity Management 및 액세스 제어 보안 모범 사례](/azure/security/azure-security-identity-management-best-practices)

### <a name="application-and-messaging-services"></a>애플리케이션 및 메시지 서비스

#### <a name="simple-email-service"></a>Simple Email Service

AWS는 알림, 트랜잭션 또는 마케팅 전자 메일을 보낼 수 있는 SES(Simple Email Service)를 제공합니다. Azure에서는 [SendGrid](https://sendgrid.com/partners/azure/) 같은 타사 솔루션이 이메일 서비스를 제공합니다.

#### <a name="simple-queueing-service"></a>Simple Queueing Service

AWS SQS(AWS Simple Queueing Service)는 AWS 플랫폼 내부의 애플리케이션, 서비스 및 장치를 연결하는 메시지 시스템을 제공합니다. Azure에도 비슷한 기능을 제공하는 두 가지 서비스가 있습니다.

- [Queue Storage](/azure/storage/queues/storage-nodejs-how-to-use-queues): Azure 플랫폼 내부의 애플리케이션 구성 요소 간 통신을 허용하는 클라우드 메시지 서비스입니다.

- [Service Bus](https://azure.microsoft.com/services/service-bus/): 애플리케이션, 서비스 및 디바이스를 연결하는 보다 강력한 메시지 시스템입니다. 관련 [Service Bus Relay](/azure/service-bus-relay/relay-what-is-it)를 사용하면 Service Bus가 원격으로 호스팅되는 애플리케이션 및 서비스에도 연결할 수 있습니다.

#### <a name="device-farm"></a>Device Farm

AWS Device Farm은 디바이스 간 테스트 서비스를 제공합니다. Azure에서는 [Xamarin Test Cloud](https://www.xamarin.com/test-cloud)가 모바일 디바이스에 이와 비슷한 디바이스 간 프런트 엔드 테스트를 제공합니다.

프런트 엔드 테스트 외에도 [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/)가 Linux 및 Windows 환경을 위한 백 엔드 테스트 리소스를 제공합니다.

#### <a name="see-also"></a>참고 항목

- [Node.js에서 Queue Storage를 사용하는 방법](/azure/storage/queues/storage-nodejs-how-to-use-queues)

- [Service Bus 큐를 사용하는 방법](/azure/service-bus-messaging/service-bus-nodejs-how-to-use-queues)

### <a name="analytics-and-big-data"></a>분석 및 빅 데이터

[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/)는 대량의 데이터를 캡처, 구성, 분석 및 시각화할 수 있도록 디자인된 Azure의 제품 및 서비스 패키지입니다. Cortana 제품군은 다음 서비스로 구성되어 있습니다.

- [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - Hadoop, Spark, Storm 또는 HBase를 포함하고 있는 관리형 Apache 배포입니다.

- [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - 데이터 오케스트레이션 및 데이터 파이프라인 기능을 제공합니다.

- [SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - 대규모 관계형 데이터 스토리지입니다.

- [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - 빅 데이터 분석 워크로드에 최적화된 대규모 스토리지입니다.

- [Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - 데이터에 대한 예측 분석을 빌드하고 적용하는 데 사용됩니다.

- [Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - 실시간으로 데이터를 분석합니다.

- [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - Data Lake Store에 사용하도록 최적화된 대규모 분석 서비스입니다.

- [PowerBI](https://powerbi.microsoft.com/) - 전원 데이터 시각화를 사용합니다.

#### <a name="see-also"></a>참고 항목

- [Cortana Intelligence 갤러리](https://gallery.cortanaintelligence.com/)

- [Microsoft 빅 데이터 솔루션의 이해](https://msdn.microsoft.com/library/dn749804.aspx)

- [Azure Data Lake 및 Azure HDInsight 블로그](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>사물 인터넷

#### <a name="see-also"></a>참고 항목

- [Azure IoT Hub 시작](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

- [IoT Hub 및 Event Hubs의 비교](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>모바일 서비스

#### <a name="notifications"></a>공지

Notification Hubs는 SMS 또는 전자 메일 메시지 보내기를 지원하지 않으므로 이러한 기능을 제공하는 타사 서비스가 필요합니다.

#### <a name="see-also"></a>참고 항목

- [Android 앱 만들기](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

- [Azure Mobile Apps의 인증 및 권한 부여](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

- [Azure Notification Hubs를 사용하여 푸시 알림 보내기](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>관리 및 모니터링

#### <a name="see-also"></a>참고 항목

- [모니터링 및 진단 지침](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

- [Azure Resource Manager 템플릿 생성 모범 사례](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

- [Azure Resource Manager 빠른 시작 템플릿](https://azure.microsoft.com/documentation/templates/)

## <a name="next-steps"></a>다음 단계

- [Azure 시작](https://azure.microsoft.com/get-started/)

- [Azure 솔루션 아키텍처](https://azure.microsoft.com/solutions/architecture/)

- [Azure 참조 아키텍처](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/

<!-- markdownlint-enable MD024 -->
