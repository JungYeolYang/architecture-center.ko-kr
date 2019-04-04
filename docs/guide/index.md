---
layout: LandingPage
ms.topic: landing-page
ms.service: architecture-center
ms.subservice: reference-architecture
ms.date: 08/30/2018
ms.openlocfilehash: 651f59344e7785a8a23e7b56dd67b4c4a3044741
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343733"
---
# <a name="azure-application-architecture-guide"></a>Azure 애플리케이션 아키텍처 가이드

이 가이드는 확장성과 복원력이 있고 가용성이 뛰어난 Azure에서 애플리케이션을 디자인하기 위한 체계적인 방식을 제시합니다. 고객의 참여를 통해 확보한 검증된 사례를 기반으로 합니다.

## <a name="introduction"></a>소개

클라우드는 애플리케이션을 디자인하는 방식을 변화시키고 있습니다. 애플리케이션은 거대한 단일 프로그램에서 보다 작은 분산형 서비스로 분해되고 있습니다. 이러한 서비스는 API나 비동기 메시징 또는 이벤트를 통해 통신합니다. 애플리케이션은 수평 확장되어 수요가 있을 때 새로운 인스턴스를 추가합니다.

이러한 추세는 새로운 도전을 불러옵니다. 애플리케이션 상태가 배포됩니다. 작업은 병렬 및 비동기식으로 수행됩니다. 장애가 발생할 경우 시스템 전체가 복원력이 있어야 합니다. 배포는 자동화되고 예측이 가능해야 합니다. 모니터링 및 원격 분석은 시스템에 대한 통찰력을 확보하는 데 매우 중요합니다. Azure 애플리케이션 아키텍처 가이드는 이러한 변화를 탐색하는 데 도움이 되도록 작성되었습니다.

<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>기존의 온-프레미스</th><th>최신 클라우드</th></tr>
</thead>
<tbody>
<tr><td>모놀리식, 중앙 집중식<br/>
예측 가능한 확장성을 위한 디자인<br/>
관계형 데이터베이스<br/>
강력한 일관성<br/>
직렬 처리 및 동기화된 처리<br/>
장애 방지를 위한 디자인(MTBF)<br/>
간헐적인 대규모 업데이트<br/>
수동 관리<br/>
Snowflake 서버</td>
<td>
분해, 분산형<br/>
탄력적 확장을 위한 디자인<br/>
다각적인 지속성(저장소 기술의 혼합)<br/>
결과적 일관성<br/>
병렬 및 비동기 처리<br/>
장애 대비 디자인(MTTR)<br/>
빈번한 소규모 업데이트<br/>
자동화된 자체 관리<br/>
변경이 불가능한 인프라<br/>
</td>
</tbody>
</table>

<!-- markdownlint-enable MD033 -->

이 가이드는 애플리케이션 설계자, 개발자 및 운영 팀을 대상으로 합니다. 개별 Azure 서비스 사용 방법을 안내하는 가이드가 아닙니다. 이 가이드를 읽으면 Azure 클라우드 플랫폼을 구축할 때 적용할 아키텍처 패턴과 모범 사례를 이해할 수 있습니다. [가이드의 전자책 버전][ebook]을 다운로드할 수도 있습니다.

## <a name="how-this-guide-is-structured"></a>이 가이드의 구조

Azure 애플리케이션 아키텍처 가이드는 아키텍처와 디자인에서 구현에 이르는 일련의 단계로 구성되어 있습니다. 각 단계마다 애플리케이션 아키텍처 설계에 도움이 되는 부연 지침이 있습니다.

### <a name="architecture-styles"></a>아키텍처 스타일

첫 번째 의사 결정이 가장 근본적인 지점입니다. 어떤 아키텍처를 구축하시겠습니까? 마이크로 서비스 아키텍처, 전통적인 n 계층 애플리케이션 또는 빅 데이터 솔루션 등을 생각할 수 있습니다. 아키텍처 스타일에는 뚜렷이 구분되는 여러 개가 있습니다. 여기에는 각각 이점과 어려운 점이 있습니다.

자세한 정보:

- [아키텍처 스타일](./architecture-styles/index.md)

### <a name="technology-choices"></a>기술 선택

2가지 기술에 대한 선택은 전체 아키텍처에 영향을 미치기 때문에 초기에 결정해야 합니다. 계산 서비스 및 데이터 저장소 선택이 여기에 해당됩니다. *계산*은 애플리케이션이 실행되는 계산 리소스의 호스팅 모델을 말합니다. *데이터 저장소*는 데이터베이스뿐만 아니라 메시지 큐, 캐시, 로그 및 애플리케이션이 저장소에 유지할 수 있는 모든 것을 포함합니다.

자세한 정보:

- [계산 서비스 선택](./technology-choices/compute-overview.md)
- [데이터 저장소 선택](./technology-choices/data-store-overview.md)

### <a name="design-principles"></a>디자인 원칙

애플리케이션의 확장성, 복원력 및 관리성을 높이는 10가지 기본 디자인 원칙을 확인했습니다. 이러한 디자인 원칙은 모든 아키텍처 스타일에 적용됩니다. 디자인 프로세스 전반에서 10가지 기본 설계 원칙을 염두에 둘 필요가 있습니다. 그런 다음, 자동 크기 조정, 캐싱, 데이터 분할, API 디자인 등과 같은 아키텍처의 특정 측면에 대한 모범 사례의 집합을 고려해야 합니다.

자세한 정보:

- [디자인 원칙](./design-principles/index.md)

### <a name="quality-pillars"></a>품질 핵심 요소

성공적인 클라우드 애플리케이션은 다음과 같은 다섯 가지의 소프트웨어 품질 요소에 중점을 둡니다. 확장성, 가용성, 복원력, 관리 및 보안 디자인 검토 검사 목록을 사용하여 이러한 품질 핵심 요소에 따라 아키텍처를 검토하십시오.

- [품질 핵심 요소](./pillars.md)

[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
