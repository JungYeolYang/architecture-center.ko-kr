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
# <a name="partition-around-limits"></a>한도에 맞춘 분할

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>분할을 사용하여 데이터베이스, 네트워크 및 계산 한도를 해결합니다.

클라우드에서 모든 서비스는 규모 확장 능력이 제한됩니다. Azure 서비스 제한 사항은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건][azure-limits]에 설명되어 있습니다. 제한에는 코어 수, 데이터베이스 크기, 쿼리 처리량 및 네트워크 처리량이 포함됩니다. 시스템 충분히 커지면, 이러한 제한 중 하나 이상에 도달할 수 있습니다. 이러한 제한을 해결하려면 분할을 사용합니다.

다음과 같이 시스템을 분할하는 방법에는 여러 가지가 있습니다.

- 데이터베이스 크기, 데이터 I/O 또는 동시 세션 수에 대한 제한을 방지하기 위해 데이터베이스를 분할합니다.

- 요청 수 또는 동시 연결 수를 제한하지 않도록 큐 또는 메시지 버스를 분할합니다.

- App Service 계획당 인스턴스 수를 제한하지 않도록 App Service 웹앱을 분할합니다.

데이터베이스는 *수평적*, *수직적* 또는 *기능적*에 따라 분할할 수 있습니다.

- 행 분할이라고도 하는 수평적 분할에서 각 파티션은 전체 데이터 집합의 하위 집합에 대한 데이터를 보유합니다. 파티션은 동일한 데이터 스키마를 공유합니다. 예를 들어, 이름이 A&ndash;M으로 시작하는 고객과 N&ndash;Z로 시작하는 고객은 다른 파티션에 속합니다.

- 수직적 분할에서 각 파티션은 데이터 저장소의 항목에 대한 필드 하위 집합을 보유합니다. 예를 들어, 자주 액세스되는 필드와 덜 자주 액세스되는 필드를 다른 파티션에 둡니다.

- 기능적 파티션에서는 시스템의 제한된 컨텍스트별로 데이터가 사용되는 방법에 따라 데이터를 분할합니다. 예를 들어, 청구서 데이터와 제품 재고 데이터를 다른 파티션에 둡니다. 스키마는 서로 독립적입니다.

자세한 지침은 [데이터 분할][data-partitioning-guidance]을 참조하세요.

## <a name="recommendations"></a>권장 사항

**애플리케이션의 다른 부분 분할** 데이터베이는 분할에 적합한 후보임에 분명하지만, 스토리지, 캐시, 큐 및 컴퓨팅 인스턴스의 분할도 고려할 수 있습니다.

**핫 스폿을 방지하도록 파티션 키 디자인** 데이터베이스를 분할해도, 하나의 분할된 데이터베이스가 요청의 대부분을 가져가므로 문제가 해결되지 않습니다. 이상적인 목표는 부하가 모든 파티션에 골고루 분산되는 것입니다. 예를 들어, 일부 문자가 좀 더 자주 사용되므로 고객 이름의 첫 문자가 아닌 고객 ID별로 해시를 수행합니다. 메시지 큐를 분할할 때도 같은 원칙이 적용됩니다. 큐 집합에서 메시지를 균등하게 분산하는 파티션 키를 선택합니다. 자세한 내용은 [분할][sharding]을 참조하세요.

**Azure 구독 및 서비스 제한을 고려한 분할** 개별 구성 요소 및 서비스에는 제한이 적용되지만, 구독 및 리소스 그룹에 대해서도 제한이 적용됩니다. 매우 큰 애플리케이션의 경우, 이러한 제한을 고려해서 분할해야 할 수 있습니다.

**다양한 수준에서 분할** VM에 배포된 데이터베이스 서버를 살펴보겠습니다. VM에는 Azure Storage가 지원하는 VHD가 있습니다. 저장소 계정은 Azure 구독에 속합니다. 계층의 각 단계에도 제한이 적용됩니다. 데이터베이스 서버에는 연결 풀 제한이 있을 수 있습니다. VM에는 CPU 및 네트워크 제한이 있습니다. 저장소에는 IOPS 제한이 있습니다. 구독에는 VM 코어 수 제한이 있습니다. 일반적으로 계층 구조의 하위에서 분할하는 것이 더 쉽습니다. 대규모 애플리케이션의 경우만 구독 수준에서 분할해야 합니다.

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md
