---
title: 클라우드 애플리케이션에 대한 성능 안티패턴
titleSuffix: Azure Architecture Center
description: 확장성 문제를 일으킬 가능성이 있는 일반적인 사례입니다.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 9ce1177bac2c93139faf6bc757f2866d6b7eac2d
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009732"
---
# <a name="performance-antipatterns-for-cloud-applications"></a>클라우드 애플리케이션에 대한 성능 안티패턴

*성능 안티패턴*은 애플리케이션이 압력을 받고 있을 때 확장성 문제를 일으킬 가능성이 있는 일반적인 사례입니다.

일반적인 시나리오는 다음과 같습니다. 애플리케이션이 성능 테스트 중에 제대로 작동합니다. 그래서 프로덕션에 릴리스되고 실제 워크로드를 처리하기 시작합니다. 이 시점에 성능이 저하되기 시작하여 &mdash; 사용자 요청 거부, 지연 또는 예외가 발생합니다. 이때 개발 팀은 다음 두 가지 질문에 직면합니다.

- 테스트하는 동안 이 동작이 나타나지 않은 이유는 무엇입니까?
- 이 문제를 어떻게 해결합니까?

첫 번째 질문에 대한 답은 간단합니다. 테스트 환경에서 실제 사용자, 해당 사용자의 행동 패턴 및 사용자가 수행할 수 있는 작업의 양을 시뮬레이션하기는 매우 어렵습니다. 시스템이 부하를 받을 때 동작하는 방법을 완전하게 확실히 이해하는 유일한 방법은 프로덕션 중에 관찰하는 것입니다. 확실히 말할 수 있는 것은 성능 테스트를 건너뛰지 말라는 것입니다. 성능 테스트는 기준선 성능 메트릭을 얻기 위해 중요합니다. 그러나 성능 문제가 작동 중인 시스템에서 발생하는 경우 관찰하고 해결할 수 있도록 준비해야 합니다.

두 번째 질문, 문제를 해결하는 방법에 대한 답은 그리 간단치 않습니다. 수많은 요인이 기여할 수 있으며 경우에 따라 문제가 특정 상황에서만 명확히 나타납니다. 계측 및 로깅은 근본 원인을 찾는 열쇠이지만 찾고 있는 내용이 무엇인지 알고 있어야 합니다.

Microsoft Azure 고객의 참여를 바탕으로 고객이 프로덕션 중에 보게 되는 가장 일반적인 성능 문제를 식별했습니다. 각 안티패턴에 대해 안티패턴이 일반적으로 발생하는 이유, 안티패턴의 증상 및 문제를 해결하는 기술을 설명합니다. 또한 안티패턴과 권장 솔루션을 보여 주는 샘플 코드도 제공합니다.

이러한 안티패턴 중 일부는 설명을 읽을 때에는 명백해 보일 수 있지만 생각보다 더 자주 발생합니다. 경우에 따라 애플리케이션이 온-프레미스로 작동하는 디자인을 상속하지만 클라우드에서 크기 조정이 되지 않습니다. 또는 애플리케이션이 아주 간결한 디자인으로 시작할 수 있지만 새 기능이 추가되는 경우 이러한 안티패턴 중 하나 이상이 도사리고 있습니다. 그런데도 이 가이드는 이러한 안티패턴을 식별하고 해결하는 데 도움이 될 것입니다.

다음은 식별된 안티패턴의 목록입니다.

| 안티패턴 | 설명 |
|-------------|-------------|
| [사용량이 많은 데이터베이스][BusyDatabase] | 너무 많은 처리가 데이터 저장소에서 오프로드로 수행됩니다. |
| [사용량이 많은 프런트 엔드][BusyFrontEnd] | 리소스를 많이 사용하는 작업을 백그라운드로 이동합니다. |
| [번잡한 I/O][ChattyIO] | 계속해서 많은 작은 네트워크 요청을 보냅니다. |
| [불필요한 가져오기][ExtraneousFetching] | 필요한 것보다 더 많은 데이터를 검색하면 불필요한 I/O 처리가 발생합니다. |
| [부적절한 인스턴스화][ImproperInstantiation] | 공유 및 다시 사용하도록 설계된 개체를 반복적으로 만들고 삭제합니다. |
| [모놀리식 지속성][MonolithicPersistence] | 매우 다른 사용량 패턴을 가진 데이터에 동일한 데이터 저장소를 사용합니다. |
| [캐싱 없음][NoCaching] | 데이터를 캐시하지 못합니다. |
| [동기 I/O][SynchronousIO] | I/O가 완료하는 동안 호출 스레드를 차단합니다. |

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md
