---
title: 데이터 웨어하우징 및 데이터 마트
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 6679ff620ca9e64036c02fce38608de38c57df93
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482176"
---
# <a name="data-warehousing-and-data-marts"></a>데이터 웨어하우징 및 데이터 마트

데이터 웨어하우스는 많은 또는 모든 주제 영역에 걸쳐, 하나 이상의 개별 원본에서 가져온 통합된 데이터의 조직적인 중앙 관계형 리포지토리입니다. 데이터 웨어하우스는 현재 및 기록 데이터를 저장하고, 다양한 방식으로 데이터의 보고 및 분석에 사용됩니다.

![Azure의 데이터 웨어하우징](../images/data-warehousing.png)

데이터 웨어하우스로 데이터를 이동하기 위해 중요한 비즈니스 정보를 포함하는 다양한 원본에서 데이터가 주기적으로 추출됩니다. 데이터는 이동될 때, 서식이 지정되고, 정리되고, 유효성이 검사되고, 요약되고, 다시 구성됩니다. 또는 데이터를 가장 낮은 세부 수준으로 저장하고, 보고를 위해 웨어하우스에 집계된 보기를 제공할 수 있습니다. 두 경우 모두에서 데이터 웨어하우스는 BI(비즈니스 인텔리전스) 도구를 사용하여 보고, 분석 및 중요한 비즈니스 의사 결정을 수행하는 데 사용되는 데이터의 영구 저장 공간이 됩니다.

## <a name="data-marts-and-operational-data-stores"></a>데이터 마트 및 운영 데이터 저장소

최대 규모의 데이터를 관리하는 것은 복잡하며, 전체 엔터프라이즈의 모든 데이터를 나타내는 단일 데이터 웨어하우스를 유지하는 것은 별로 일반적이지 않습니다. 대신, 조직에서는 분석을 위해 원하는 데이터를 노출하는 *데이터 마트*라는 좀 더 작고 좀 더 특정 영역을 중심으로 하는 데이터 웨어하우스를 만듭니다. 오케스트레이션 프로세스는 운영 데이터 저장소에 유지 관리되는 데이터로 데이터 마트를 채웁니다. 운영 데이터 저장소는 원본 트랜잭션 시스템 및 데이터 마트 간의 중간 매개 역할을 합니다. 운영 데이터 저장소에서 관리되는 데이터는 원본 트랜잭션 시스템에 있는 데이터의 정리된 버전으로, 일반적으로 데이터 웨어하우스 또는 데이터 마트에서 유지 관리되는 기록 데이터의 하위 집합입니다.

## <a name="when-to-use-this-solution"></a>이 솔루션을 사용해야 하는 경우

운영 체제의 대량의 데이터를 이해하기 쉽고 정확한 최신 형식으로 변환해야 할 경우 데이터 웨어하우스를 선택합니다. 데이터 웨어하우스는 운영/OLTP 데이터베이스에서 사용될 수 있는 동일한 간결한 데이터 구조를 따를 필요가 없습니다. 비즈니스 사용자 및 분석가에게 적절해 보이는 열 이름을 사용하고, 스키므를 다시 구성하여 데이터 관계를 간소화하고, 여러 테이블을 하나로 통합할 수 있습니다. 이러한 단계는 DBA(데이터베이스 관리자) 또는 데이터 개발자의 도움 없이 임시 보고서를 만들거나, 보고서를 만들고 BI 시스템에서 데이터를 분석해야 하는 사용자를 안내하는 데 도움이 됩니다.

성능상의 이유로 기록 데이터를 원본 트랜잭션 시스템과는 따로 보관하려는 경우, 데이터 웨어하우스를 사용하는 것이 좋습니다. 데이터 웨어하우스는 중앙 위치를 제공하여 일반 형식, 공통 키, 공용 데이터 모델 및 일반적인 액세스 방법으로 여러 위치의 기록 데이터에 쉽게 액세스할 수 있도록 합니다.

데이터 웨어하우스는 읽기 액세스에 최적화되어 있으므로, 원본 트랜잭션 시스템에 대해 보고서를 실행하는 방식에 비해 보고서를 더 빠르게 생성할 수 있도록 합니다. 또한 데이터 웨어하우스는 다음과 같은 이점을 제공합니다.

- 여러 원본의 모든 기록 데이터가 저장되며, 신뢰할 수 있는 단일 원본으로서 데이터 웨어하우스에서 액세스될 수 있습니다.
- 데이터 웨어하우스로 데이터를 가져올 때 정리하고, 일관된 코드 및 설명을 제공할 뿐만 아니라 보다 정확한 데이터를 제공하여 데이터 품질을 개선할 수 있습니다.
- 보고 도구는 쿼리 처리 주기 동안 트랜잭션 원본 시스템과 경쟁하지 않습니다. 데이터 웨어하우스는 대부분의 읽기 요청을 충족하면서, 트랜잭션 시스템이 쓰기 작업을 처리하는 데 주로 초점을 맞출 수 있도록 합니다.
- 데이터 웨어하우스는 서로 다른 소프트웨어의 데이터를 통합하는 데 도움이 될 수 있습니다.
- 데이터 마이닝 도구는 웨어하우스에 저장된 데이터에 대해 자동 방법을 사용하여 숨겨진 패턴을 찾는 데 도움이 될 수 있습니다.
- 데이터 웨어하우스를 사용하면 허가되지 않은 사용자의 액세스는 제한하면서 허가된 사용자에게 쉽게 보안 액세스를 제공할 수 있습니다. 비즈니스 사용자에게 원본 데이터에 대한 액세스 권한을 부여할 필요가 없으므로, 하나 이상의 프로덕션 트랜잭션 시스템에 대한 잠재적인 공격 벡터가 제거됩니다.
- 데이터 웨어하우스를 데이터 위에 [OLAP 큐브](online-analytical-processing.md)와 같은 비즈니스 인텔리전스 솔루션을 보다 쉽게 만들 수 있습니다.

## <a name="challenges"></a>과제

비즈니스의 요구에 맞게 데이터 웨어하우스를 적절히 구성하려는 경우 다음과 같은 해결 과제에 직면할 수 있습니다.

- 비즈니스 개념을 적절히 모델링하는 데 필요한 시간 확정. 데이터 웨어하우스는 정보를 중심으로 하므로, 개념 매핑이 나머지 프로젝트를 구동하게 됩니다. 따라서 이 단계는 중요합니다. 이 단계에서는 비즈니스 관련 용어 및 일반 형식(예: 통화 및 날짜)이 표준화되고, 비즈니스 사용자에게 보다 적절한 방식으로 데이터가 다시 구성되지만, 데이터 집계 및 관계의 정확도도 계속 보장됩니다.

- 데이터 오케스트레이션 계획 및 설정. 고려 사항으로는 원본 트랜잭션 시스템의 데이터를 데이터 웨어하우스로 복사하는 방법과, 운영 데이터 저장소의 기록 데이터를 웨어하우스로 이동하는 시기가 포함됩니다.

- 데이터를 웨어하우스로 가져올 때 데이터를 정리하여 품질 유지 또는 개선.

## <a name="data-warehousing-in-azure"></a>Azure의 데이터 웨어하우징

Azure에서는 고객의 트랜잭션이든, 다양한 부서에서 사용되는 다양한 비즈니스 애플리케이션이든, 하나 이상의 데이터 원본이 있을 수 있습니다. 이 데이터는 일반적으로 하나 이상의 [OLTP](online-transaction-processing.md) 데이터베이스에 저장됩니다. 데이터는 네트워크 공유, Azure Storage Blob 또는 Data Lake 같은 기타 스토리지 미디어에 유지될 수 있습니다. 데이터가 데이터 웨어하우스 자체 또는 Azure SQL Database 같은 관계형 데이터베이스에 저장할 수도 있습니다. 분석 데이터 저장소 계층의 목적은 데이터 웨어하우스 또는 데이터 마트에 대해 분석 및 보고 도구에서 실행한 쿼리를 충족하는 것입니다. Azure에서 이 분석 저장소 기능은 Azure SQL Data Warehouse를 사용하거나 Hive 또는 대화형 쿼리를 통해 Azure HDInsight를 사용하여 충족될 수 있습니다. 또한 데이터 저장소의 데이터를 데이터 웨어하우스에 주기적으로 이동하거나 복사하기 위해 일정 수준의 오케스트레이션이 필요합니다. 이 작업은 Azure Data Factory 또는 Azure HDInsight의 Oozie를 사용하여 수행할 수 있습니다.

Azure에서는 사용자의 요구에 따라 다음과 같은 몇 가지 데이터 웨어하우스 구현 옵션을 사용할 수 있습니다. 다음 목록은 SMP([Symmetric Multiprocessing](https://en.wikipedia.org/wiki/Symmetric_multiprocessing)) 및 MPP([Massively Parallel Processing](https://en.wikipedia.org/wiki/Massively_parallel))의 두 범주로 구분됩니다.

SMP:

- [Azure SQL Database](/azure/sql-database/)
- [가상 컴퓨터의 SQL Server](/sql/sql-server/sql-server-technical-documentation)

MPP:

- [Azure Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight의 Apache Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight의 대화형 쿼리(Hive LLAP)](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

일반적으로 MPP는 종종 빅 데이터에 사용되지만, SMP 기반 웨어하우스는 소형에서 중형 데이터 집합(최대 4-100TB)에 가장 적합합니다. 소형/중형과 빅 데이터 간의 경계는 조직의 정의 및 지원 인프라와 어느 정도 관련이 있습니다. ([OLTP 데이터 저장소 선택](online-transaction-processing.md#scalability-capabilities)을 참조하세요.)

데이터 크기보다는, 워크로드 패턴 유형이 더 큰 고려 요인일 수 있습니다. 예를 들어, 복잡한 쿼리는 SMP 솔루션에 비해 너무 느리므로 대신 MPP 솔루션이 필요할 수 있습니다. MPP 기반 시스템은 노드에서 작업이 배포되고 통합되는 방식 때문에, 데이터 크기가 작은 경우 성능이 저하될 수 있습니다. 데이터 크기가 이미 1TB를 초과하며, 지속적으로 증가할 수 있는 경우 MPP 솔루션을 선택하는 것이 좋습니다. 그러나 데이터 크기가 이 범위보다 작지만 워크로드가 SMP 솔루션의 가용 리소스를 초과할 경우에는 MPP가 가장 적합할 수 있습니다.

데이터 웨어하우스에서 액세스하거나 저장하는 데이터는 [Azure Data Lake Store](/azure/data-lake-store/)와 같은 데이터 레이크를 포함하는 다양한 데이터 원본에서 가져올 수 있습니다. Azure Data Lake를 사용할 수 있는 MPP 서비스의 서로 다른 장점을 비교하는 비디오 세션은 [Azure Data Lake 및 Azure Data Warehouse: 앱에 최신 사례 적용](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/)을 참조하세요.

SMP 시스템은 모든 리소스(CPU/메모리/디스크)를 공유하는 관계형 데이터베이스 관리 시스템의 단일 인스턴스로 나타낼 수 있습니다. SMP 시스템은 확장할 수 있습니다. VM에서 실행 중인 SQL Server의 경우 VM 크기를 확장할 수 있습니다. Azure SQL Database의 경우 다른 서비스 계층을 선택하여 확장할 수 있습니다.

MPP 시스템은 계산 노드(자체 CUP, 메모리 및 I/O 하위 시스템 포함)를 더 추가하여 스케일 아웃할 수 있습니다. 서버를 강화하는 데는 물리적 제한이 있습니다. 이 경우 워크로드에 따라 스케일 아웃하는 것이 좀 더 바람직합니다. 그러나 MPP 솔루션에는 데이터의 쿼리, 모델링, 분할 방식의 차이와 병렬 처리와 관련된 기타 요인 때문에 다른 기술 집합이 필요합니다.

사용할 SMP 솔루션을 결정할 때는 [Azure SQL 데이터베이스 및 Azure VM의 SQL Server에서 자세히 보기](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms)를 참조하세요.

워크로드가 계산 및 메모리 집약적인 소형 및 중형 데이터 세트에도 Azure SQL Data Warehouse를 사용할 수 있습니다. SQL Data Warehouse 패턴 및 일반적인 시나리오에 대해 읽어보세요.

- [SQL Data Warehouse 패턴 및 안티패턴](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)

- [SQL Data Warehouse 로딩 패턴 및 전략](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)

- [Azure SQL Data Warehouse로 데이터 마이그레이션](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/)(영문)

- [Azure SQL Data Warehouse를 사용하는 일반적인 ISV 애플리케이션 패턴](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/)

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 사용자 고유의 서버를 관리하지 않고 관리되는 서비스를 원하시나요?

- 매우 큰 데이터 집합 또는 매우 복잡하고 오래 실행되는 쿼리로 작업하고 있나요? 그렇다면 MPP 옵션을 고려합니다.

- 큰 데이터 집합의 경우 데이터 원본이 구조적인가요 아니면 반구조적인가요? 구조화되지 않은 데이터는 HDInsight의 Spark, Azure Databricks, HDInsight의 Hive LLAP 또는 Azure Data Lake Analytics와 같은 빅 데이터 환경에서 처리해야 할 수 있습니다. 이러한 모든 환경은 ELT(추출, 로드, 변환) 및 ETL(추출, 변환, 로드) 엔진으로 사용할 수 있습니다. 처리된 데이터를 구조화된 데이터로 출력하여 SQL Data Warehouse 또는 다른 옵션 중 하나로 보다 쉽게 로드할 수 있습니다. 구조화된 데이터의 경우, SQL Data Warehouse는 아주 높은 성능을 요구하는 계산 집약적 워크로드를 위해 [컴퓨팅 최적화]라고 지칭하는 성능 계층을 갖습니다.

- 현재, 운영 데이터에서 기록 데이터를 분리하려고 하나요? 그렇다면 [오케스트레이션](../technology-choices/pipeline-orchestration-data-movement.md)이 필요한 옵션 중 하나를 선택합니다. 이것은 과도한 읽기 액세스에 최적화된 독립 실행형 웨어하우스로, 별도의 기록 데이터 저장소로 가장 적합합니다.

- OLTP 데이터 저장소 이외의 여러 원본에 있는 데이터를 통합해야 하나요? 그렇다면 여러 데이터 원본을 쉽게 통합하는 옵션을 고려합니다.

- 다중 테넌트 요구 사항이 있나요? 그렇다면 이 요구 사항에는 SQL Data Warehouse가 바람직하지 않습니다. 자세한 내용은 [SQL Data Warehouse 패턴 및 안티패턴](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)을 참조하세요.

- 관계형 데이터 저장소를 선호하나요? 그렇다면, 관계형 데이터 저장소가 있는 옵션으로 범위를 좁히되, 필요한 경우 PolyBase와 같은 도구를 사용하여 비관계형 데이터 저장소를 쿼리할 수 있다는 점에 유의하세요. 그러나 PolyBase를 사용하기로 결정한 경우 워크로드의 구조화되지 않은 데이터 집합에 대해 성능 테스트를 실행합니다.

- 실시간 보고 요구 사항이 있나요? 고용량의 단일 삽입에 대해 신속한 쿼리 응답 시간이 필요한 경우 실시간 보고를 지원할 수 있는 옵션으로 범위를 좁힙니다.

- 많은 수의 동시 사용자 및 연결을 지원해야 하나요? 동시 사용자/연결 수를 지원하는 기능은 여러 가지 요인에 따라 달라집니다.

  - Azure SQL Database에 대해서는 서비스 계층에 따라 [문서화된 리소스 제한](/azure/sql-database/sql-database-resource-limits)을 참조하세요.
  
  - SQL Server에서는 최대 32,767개의 사용자 연결을 허용합니다. VM에서 실행할 경우 성능은 VM 크기 및 기타 요인에 따라 달라집니다.

  - SQL Data Warehouse는 동시 쿼리 및 동시 연결에 대한 제한이 있습니다. 자세한 내용은 [SQL Data Warehouse의 동시성 및 워크로드 관리](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency)를 참조하세요. [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)와 같은 보완 서비스를 사용하여 SQL Data Warehouse의 제한을 극복하는 것이 바람직합니다.

- 어떤 종류의 워크로드가 발생하나요? 일반적으로 MPP 기반 웨어하우스 솔루션은 분석, 일괄 처리 워크로드에 가장 적합합니다. 워크로드가 기본적으로 많은 작은 읽기/쓰기 작업 또는 여러 행 단위 작업을 포함하는 트랜잭션 작업인 경우 SMP 옵션 중 하나를 사용하는 것이 좋습니다. 이 지침의 한 가지 예외는 HDInsight 클러스터에서 Spark Streaming 같은 스트림 처리를 사용하고 데이터를 Hive 테이블 내에 저장하는 경우입니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능

<!-- markdownlint-disable MD033 -->

| | Azure SQL Database | SQL Server(VM) | SQL Data Warehouse | HDInsight의 Apache Hive | HDInsight의 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 관리되는 서비스인지 여부 | 예 | no | 예 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 데이터 오케스트레이션 필요(데이터 사본/기록 데이터 보유) | 아니요 | 아니요 | 예 | 예 | 예 |
| 여러 데이터 원본을 쉽게 통합 | 아니요 | 아니요 | 예 | 예 | 예 |
| 계산 일시 중지 지원 여부 | 아니요 | 아니요 | 예 | 아니요 <sup>2</sup> | 아니요 <sup>2</sup> |
| 관계형 데이터 저장소 | 예 | 예 |  예 | 아니요 | 아니요 |
| 실시간 보고 | 예 | 예 | 아니요 | 아니요 | 예 |
| 유연한 백업/복원 지점 | 예 | 예 | 아니요 <sup>3</sup> | 예 <sup>4</sup> | 예 <sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

<!-- markdownlint-enable MD033 -->

[1] 수동 구성 및 크기 조정

[2] 필요할 때 HDInsight 클러스터를 삭제한 다음, 다시 만들 수 있습니다. 클러스터를 삭제할 떼 데이터가 유지되도록 외부 데이터 저장소를 클러스터에 연결합니다. Azure Data Factory로 워크로드를 처리하는 요청 시 HDInsight 클러스터를 만들어 클러스터의 수명 주기를 자동화한 다음, 처리가 완료된 후 삭제할 수 있습니다.

[3] SQL Data Warehouse를 사용할 경우 지난 7일 이내의 사용 가능한 복원 지점으로 데이터베이스를 복원할 수 있습니다. 스냅숏은 4~8시간마다 시작되며 7일 동안 사용할 수 있습니다. 7일보다 오래된 스냅숏은 만료되고 해당 복원 지점을 더 이상 사용할 수 없게 됩니다.

[4] 필요할 때 백업 및 복원할 수 있는 [외부 Hive metastore](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore)를 사용하는 것이 좋습니다. Blob Storage 또는 Data Lake Store에 적용되는 표준 백업 및 복원 옵션을 데이터에 사용할 수 있으며, 더 큰 유연성이나 사용 편의성을 원할 경우 [Imanis Data](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/)와 같은 타사 HDInsight 백업 및 복원 솔루션을 사용할 수 있습니다.

### <a name="scalability-capabilities"></a>확장성 기능

<!-- markdownlint-disable MD033 -->

| | Azure SQL Database | SQL Server(VM) |  SQL Data Warehouse | HDInsight의 Apache Hive | HDInsight의 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 고가용성을 위한 중복 지역 서버  | 예 | 예 | 예 | 아니요 | 아니요 |
| 쿼리 스케일 아웃 지원 여부(분산 쿼리)  | 아니요 | 아니요 | 예 | 예 | 예 |
| 동적 확장성 | 예 | 아니요 | 예 <sup>1</sup> | 아니요 | 아니요 |
| 데이터의 메모리 내 캐싱 지원 여부 | 예 |  예 | no | 예 | 예 |

[1] SQL Data Warehouse에서는 DWU(데이터 웨어하우스 단위) 수를 조정하여 강화 및 축소할 수 있습니다. [Azure SQL Data Warehouse의 계산 능력 관리](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview)를 참조하세요.

<!-- markdownlint-enable MD033 -->

### <a name="security-capabilities"></a>보안 기능

<!-- markdownlint-disable MD033 -->

|                         |           Azure SQL Database            |  가상 컴퓨터의 SQL Server  | SQL Data Warehouse |   HDInsight의 Apache Hive    |    HDInsight의 Hive LLAP     |
|-------------------------|-----------------------------------------|-----------------------------------|--------------------|-------------------------------|-------------------------------|
|     인증      | SQL/Azure AD(Azure Active Directory) | SQL / Azure AD / Active Directory |   SQL / Azure AD   | 로컬/Azure AD <sup>1</sup> | 로컬/Azure AD <sup>1</sup> |
|      권한 부여      |                   예                   |                예                |        예         |              예              |       예 <sup>1</sup>        |
|        감사         |                   예                   |                예                |        예         |              예              |       예 <sup>1</sup>        |
| 휴지 상태의 암호화 |            예 <sup>2</sup>             |         예 <sup>2</sup>          |  예 <sup>2</sup>  |       예 <sup>2</sup>        |       예 <sup>1</sup>        |
|   행 수준 보안    |                   예                   |                예                |        예         |              아니요               |       예 <sup>1</sup>        |
|   방화벽 지원 여부    |                   예                   |                예                |        예         |              예              |       예 <sup>3</sup>        |
|  동적 데이터 마스킹   |                   예                   |                예                |        예         |              아니요               |       예 <sup>1</sup>        |

<!-- markdownlint-enable MD033 -->

[1] [도메인 가입 HDInsight 클러스터](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)를 사용해야 합니다.

[2] 미사용 데이터의 암호화 및 암호 해독을 위해 TDE(투명한 데이터 암호화)를 사용해야 합니다.

[3] [Azure Virtual Network 내에서 사용](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)할 때 지원됩니다.

데이터 웨어하우스 보안에 대해 읽어보세요.

- [SQL Database 보안 설정](/azure/sql-database/sql-database-security-overview#connection-security)

- [SQL Data Warehouse에서 데이터베이스 보호](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)

- [Azure Virtual Network를 사용하여 Azure HDInsight 확장](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)

- [도메인 조인 HDInsight 클러스터의 엔터프라이즈 수준 Hadoop 보안](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)
