---
title: 가용성 패턴
titleSuffix: Cloud Design Patterns
description: 가용성은 시스템이 기능하고 작동하는 시간의 비율을 정의합니다. 시스템 오류, 인프라 문제, 악의적인 공격, 시스템 부하 등의 영향을 받습니다. 일반적으로 가동 시간의 백분율로 측정됩니다. 클라우드 애플리케이션은 일반적으로 사용자에게 SLA(서비스 수준 계약)를 제공합니다. 이것은 애플리케이션이 가용성을 최대화하는 방식으로 디자인 및 구현되어야 함을 의미합니다.
keywords: 디자인 패턴
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 897bc3c87beb23770220ef0fc3c5c4394d192f2a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243044"
---
# <a name="availability-patterns"></a>가용성 패턴

[!INCLUDE [header](../../_includes/header.md)]

가용성은 시스템이 기능하고 작동하는 시간의 비율을 정의합니다. 시스템 오류, 인프라 문제, 악의적인 공격, 시스템 부하 등의 영향을 받습니다. 일반적으로 가동 시간의 백분율로 측정됩니다. 클라우드 애플리케이션은 일반적으로 사용자에게 SLA(서비스 수준 계약)를 제공합니다. 이것은 애플리케이션이 가용성을 최대화하는 방식으로 디자인 및 구현되어야 함을 의미합니다.

|                            패턴                             |                                                           요약                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [상태 엔드포인트 모니터링](../health-endpoint-monitoring.md) | 외부 도구가 노출된 엔드포인트를 통해 주기적으로 액세스할 수 있는 기능 검사를 애플리케이션 내부에 구현합니다. |
|  [큐 기반 부하 평준화](../queue-based-load-leveling.md)  | 작업 그리고 그 작업이 일시적인 높은 부하를 부드럽게 처리하기 위해 호출하는 서비스 사이에서 버퍼 역할을 하는 큐를 사용합니다.  |
|                 [제한](../throttling.md)                 |   애플리케이션 인스턴스, 개별 테넌트 또는 서비스 전체의 리소스 사용량을 제어합니다.    |
