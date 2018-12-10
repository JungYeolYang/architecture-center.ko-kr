---
title: Azure에서 고가용성 SharePoint Server 2016 팜 실행
titleSuffix: Azure Reference Architectures
description: Azure에서 고가용성 SharePoint Server 2016 팜을 배포하기 위해 권장되는 아키텍처입니다.
author: njray
ms.date: 07/26/2018
ms.custom: seodec18
ms.openlocfilehash: 6cc8255f95cb4944ff3ef138ad5edf2e5bbea4b4
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120104"
---
# <a name="run-a-highly-available-sharepoint-server-2016-farm-in-azure"></a>Azure에서 고가용성 SharePoint Server 2016 팜 실행

이 참조 아키텍처에서는 MinRole 토폴로지 및 SQL Server Always On 가용성 그룹을 사용하여 Azure에 고가용성 SharePoint Server 2016 팜을 배포하는 검증된 사례를 보여줍니다. SharePoint 팜은 인터넷 연결 엔드포인트 또는 실체가 없는 안전한 가상 네트워크에 배포됩니다. [**이 솔루션을 배포합니다**](#deploy-the-solution).

![Azure의 고가용성 SharePoint Server 2016 팜에 대한 참조 아키텍처](./images/sharepoint-ha.png)

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>아키텍처

이 아키텍처는 [N 계층 응용 프로그램에 대한 Windows VM 실행][windows-n-tier]에 나와 있는 아키텍처를 사용합니다. Azure 가상 네트워크(VNet) 내부에 고가용성 SharePoint Server 2016 팜을 배포합니다. 이 아키텍처는 테스트 또는 프로덕션 환경, Office 365를 사용하는 SharePoint 하이브리드 인프라 또는 재해 복구 시나리오의 기준으로 사용하기에 적합합니다.

이 아키텍처는 다음과 같은 구성 요소로 구성됩니다.

- **리소스 그룹**. [리소스 그룹][resource-group]은 관련 Azure 리소스를 보유하는 컨테이너입니다. 한 리소스 그룹은 SharePoint 서버에 사용되고, 다른 리소스 그룹은 가상 네트워크나 부하 분산 장치처럼 VM에 독립적인 인프라 구성 요소에 사용됩니다.

- **가상 네트워크(VNet)**. VM은 고유한 인트라넷 주소 공간을 사용하는 VNet에 배포됩니다. VNet은 서브넷으로 더욱 세분화됩니다.

- **VM(가상 머신)**. VM은 VNet에 배포되며, 모든 VM은 개인 고정 IP 주소가 할당됩니다. SQL Server 및 SharePoint Server 2016을 실행하는 VM에는 재시작 후 IP 주소 캐싱 및 주소 변경 문제를 방지할 수 있도록 고정 IP 주소를 사용하는 것이 좋습니다.

- **가용성 집합**. 각 SharePoint 역할의 VM을 별도의 [가용성 집합][availability-set]에 배치하고, 역할마다 VM(가상 머신)을 두 대 이상 프로비전합니다. 이렇게 하면 VM이 더 높은 Service Level Agreement(서비스 수준 약정)를 충족할 수 있습니다.

- **내부 부하 분산 장치**. [부하 분산 장치][load-balancer]는 온-프레미스 네트워크의 SharePoint 요청 트래픽을 SharePoint 팜의 프런트 엔드 웹 서버로 분산합니다.

- **NSG(네트워크 보안 그룹)**. 가상 머신을 포함하고 있는 서브넷마다 [네트워크 보안 그룹][nsg]이 생성됩니다. 서브넷을 격리하려면 NSG를 사용하여 VNet 내부의 네트워크 트래픽을 제한합니다.

- **게이트웨이**. 게이트웨이는 온-프레미스 네트워크와 Azure 가상 네트워크를 연결합니다. 연결에 ExpressRoute 또는 사이트 간 VPN을 사용할 수 있습니다. 자세한 내용은 [온-프레미스 네트워크를 Azure에 연결][hybrid-ra]을 참조하세요.

- **Windows Server AD(Active Directory) 도메인 컨트롤러**. 이 참조 아키텍처는 Windows Server AD 도메인 컨트롤러를 배포합니다. 이러한 도메인 컨트롤러는 Azure VNet에서 실행되며 온-프레미스 Windows Server AD 포리스트와 트러스트 관계에 있습니다. 게이트웨이 연결의 인증 트래픽을 온-프레미스 네트워크로 전송하는 대신 VNet에서 SharePoint 팜 리소스에 대한 클라이언트 웹 요청이 인증됩니다. DNS에서 인트라넷 A 또는 CNAME 레코드가 생성되므로 인트라넷 사용자는 내부 부하 분산 장치의 개인 IP 주소에 해당하는 SharePoint 팜 이름을 확인할 수 있습니다.

  SharePoint Server 2016은 [Azure Active Directory Domain Services](/azure/active-directory-domain-services/)를 사용하도록 지원합니다. Azure AD Domain Services는 관리 도메인 서비스를 제공하므로 Azure에서 도메인 컨트롤러를 배포하고 관리할 필요가 없습니다.

- **SQL Server Always On 가용성 그룹**. SQL Server 데이터 고가용성의 경우 [SQL Server Always On 가용성 그룹][sql-always-on]을 추천합니다. 두 대의 가상 머신이 SQL Server에 사용됩니다. 하나는 주 데이터베이스 복제본을 포함하고 다른 하나는 보조 복제본을 포함합니다.

- **주 노드 VM**. 이 VM은 장애 조치(failover) 클러스터의 쿼럼 설정을 허용합니다. 자세한 내용은 [장애 조치(Failover) 클러스터의 쿼럼 구성 이해][sql-quorum]를 참조하세요.

- **SharePoint 서버**. SharePoint 서버는 웹 프런트 엔드, 캐싱, 응용 프로그램 및 검색 역할을 수행합니다.

- **Jumpbox**. [요새 호스트][bastion-host]라고도 합니다. 관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다. jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다. NSG는 RDP(원격 데스크톱) 트래픽을 허용해야 합니다.

## <a name="recommendations"></a>권장 사항

개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요.

### <a name="resource-group-recommendations"></a>리소스 그룹 권장 사항

서버 역할에 따라 리소스 그룹을 나누고, 전역 리소스인 인프라 구성 요소에 사용할 별도의 리소스 그룹을 두는 것이 좋습니다. 이 아키텍처에서는 SharePoint 리소스가 한 그룹을 형성하고, SQL Server와 기타 유틸리티 자산이 또 다른 그룹을 형성합니다.

### <a name="virtual-network-and-subnet-recommendations"></a>가상 네트워크 및 서브넷 권장 사항

SharePoint 역할마다 서브넷을 하나씩 사용하고, 게이트웨이와 jumpbox에도 서브넷을 하나씩 사용합니다.

게이트웨이 서브넷의 이름을 *GatewaySubnet*으로 지정해야 합니다. 가상 네트워크 주소 공간의 마지막 부분에서 게이트웨이 서브넷 주소 공간을 할당합니다. 자세한 내용은 [VPN 게이트웨이를 사용하여 온-프레미스 네트워크를 Azure에 연결][hybrid-vpn-ra]을 참조하세요.

### <a name="vm-recommendations"></a>VM 권장 사항

이 아키텍처에는 최소 44코어가 필요합니다.

- Standard_DS3_v2(각각 4코어)에 SharePoint 서버 8개 = 32코어
- Standard_DS1_v2(각각 1코어)에 Active Directory 도메인 컨트롤러 2개 = 2코어
- Standard_DS3_v2에 SQL Server VM 2대 = 8코어
- Standard_DS1_v2에 주 노드 1개 = 1코어
- Standard_DS1_v2에 관리 서버 1개 = 1코어

배포에 사용할 Azure 구독의 VM 코어 할당량이 충분해야 하며, 그렇지 않으면 배포가 실패합니다. [Azure 구독 및 서비스 제한, 할당량 및 제약 조건][quotas]을 참조하세요.

Search Indexer를 제외한 모든 SharePoint 역할에는 [Standard_DS3_v2][vm-sizes-general] VM 크기를 사용하는 것이 좋습니다. Search Indexer에는 최소한 [Standard_DS13_v2][vm-sizes-memory] 이상을 사용해야 합니다. 테스트하기 위해 이 참조 아키텍처의 매개 변수 파일은 검색 인덱서 역할에 대해 더 작은 DS3_v2 크기를 지정합니다. 프로덕션 배포의 경우 DS13 이상 크기를 사용하도록 매개 변수 파일을 업데이트합니다. 자세한 정보는 [SharePoint Server 2016의 하드웨어 및 소프트웨어 요구 사항][sharepoint-reqs]을 참조하세요.

SQL Server VM의 경우 최소 4코어 8GB RAM을 사용하는 것이 좋습니다. 이 참조 아키텍처의 매개 변수 파일은 DS3_v2 크기를 지정합니다. 프로덕션 배포의 경우 더 큰 VM 크기를 지정해야 합니다. 자세한 내용은 [저장소 및 SQL Server 용량 계획 및 구성(SharePoint Server)](/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#estimate-memory-requirements)을 참조하세요.

### <a name="nsg-recommendations"></a>NSG 권장 사항

VM을 포함하는 서브넷마다 NSG를 하나씩 사용하여 서브넷을 격리하는 것이 좋습니다. 서브넷 격리를 구성하려면 각 서브넷에 허용 또는 거부되는 인바운드 또는 아웃 바운드 트래픽을 정의하는 NSG 규칙을 추가하세요. 자세한 내용은 [네트워크 보안 그룹을 사용하여 네트워크 트래픽 필터링][virtual-networks-nsg]을 참조하세요.

게이트웨이 서브넷에는 NSG를 할당하지 마세요. 할당하면 게이트웨이의 작동이 중지됩니다.

### <a name="storage-recommendations"></a>저장소 권장 사항

팜의 VM 저장소 구성은 온-프레미스 배포에 사용되는 적절한 모범 사례와 일치해야 합니다. SharePoint 서버는 로그에 사용할 별도의 디스크가 필요합니다. 검색 인덱스 역할을 호스팅하는 SharePoint 서버는 검색 인덱스를 저장할 추가 디스크 공간이 필요합니다. SQL Server의 경우 데이터와 로그를 구분하는 것이 표준 사례입니다. 데이터베이스 백업 저장소에 디스크를 추가하고 [tempdb][tempdb]에 별도의 디스크를 사용합니다.

안정성을 최대로 높이려면 [Azure Managed Disks][managed-disks]를 사용하는 것이 좋습니다. 관리 디스크는 가용성 집합 내의 VM용 디스크를 격리하여 단일 실패 지점을 방지합니다.

> [!NOTE]
> 현재 이 참조 아키텍처에 대한 Resource Manager 템플릿은 관리 디스크를 사용하지 않습니다. 관리 디스크를 사용하도록 템플릿을 업데이트할 예정입니다.

모든 SharePoint 및 SQL Server VM에 프리미엄 관리 디스크를 사용합니다. 주 노드 서버, 도메인 컨트롤러 및 관리 서버에 표준 관리 디스크를 사용해도 됩니다.

### <a name="sharepoint-server-recommendations"></a>SharePoint Server 권장 사항

SharePoint 팜을 구성하기 전에 서비스마다 Windows Server Active Directory 서비스 계정이 하나씩 있는지 확인합니다. 이 아키텍처는 역할당 권한을 격리할 다음과 같은 최소 도메인 수준 계정이 필요합니다.

- SQL Server 서비스 계정
- 설치 사용자 계정
- 서버 팜 계정
- 서버 서비스 계정
- 콘텐츠 액세스 계정
- 웹앱 풀 계정
- 서비스 앱 풀 계정
- 캐시 슈퍼 사용자 계정
- 캐시 슈퍼 읽기 권한자 계정

초당 최소 200MB라는 디스크 처리량 지원 요구를 충족하도록 검색 아키텍처를 계획해야 합니다. [SharePoint Server 2013에서 엔터프라이즈 검색 아키텍처 계획][sharepoint-search]을 참조하세요. 그리고 [SharePoint Server 2016에서 크롤링하는 방법에 대한 모범 사례][sharepoint-crawling]의 지침을 따르세요.

또한 검색 구성 요소 데이터를 별도의 고성능 저장소 볼륨 또는 파티션에 저장합니다. 부하를 줄이고 처리량을 높일 수 있도록 이 아키텍처에 필요한 개체 캐시 사용자 계정을 구성합니다. Windows Server 운영 체제 파일, SharePoint Server 2016 프로그램 파일 및 진단 로그를 별도의 중간 성능 저장소 볼륨 또는 파티션 3개에 분할합니다.

이러한 권장 사항에 대한 자세한 내용은 [SharePoint Server 2016의 초기 배포 관리 및 서비스 계정][sharepoint-accounts]을 참조하세요.

### <a name="hybrid-workloads"></a>하이브리드 워크로드

이 참조 아키텍처는 [SharePoint 하이브리드 환경] [sharepoint-hybrid]&mdash;으로 사용할 수 있는, 다시 말해서 SharePoint Server 2016을 Office 365 SharePoint Online으로 확장하는 SharePoint Server 2016 팜을 배포합니다. Office Online Server가 있는 분은 [Azure의 Office Web Apps 및 Office Online Server 지원 여부][office-web-apps]를 참조하세요.

이 배포의 기본 서비스 응용 프로그램은 하이브리드 워크로드를 지원하도록 설계되었습니다. SharePoint 인프라를 변경하지 않고도 모든 SharePoint Server 2016 및 Office 365 하이브리드 워크로드를 이 팜에 배포할 수 있지만 한 가지 예외가 있습니다. 클라우드 하이브리드 검색 서비스 응용 프로그램은 기존 검색 토폴로지를 호스팅하는 서버에 배포하면 안 됩니다. 따라서 이 하이브리드 시나리오를 지원하려면 하나 이상의 검색-역할 기반 VM을 팜에 추가해야 합니다.

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On 가용성 그룹

SharePoint Server 2016에서 Azure SQL Database를 사용할 수 없으므로 이 아키텍처는 SQL Server 가상 머신을 사용합니다. SQL Server에서 고가용성을 지원하려면 함께 장애 조치(failover)되는 데이터베이스 집합을 지정하여 가용성과 복원력을 높이는 Always On 가용성 그룹을 사용하는 것이 좋습니다. 이 참조 아키텍처에서는 배포하는 동안 데이터베이스가 생성되지만, 수동으로 Always On 가용성 그룹을 사용하도록 설정하고 가용성 그룹에 SharePoint 데이터베이스를 추가해야 합니다. 자세한 내용은 [가용성 그룹을 만들고 SharePoint 데이터베이스 추가][create-availability-group]를 참조하세요.

또한 SQL Server 가상 머신의 내부 부하 분산 장치에 대한 개인 IP 주소인 수신기 IP 주소를 클러스터에 추가하는 것이 좋습니다.

Azure에서 실행되는 SQL Server에 대한 권장 VM 크기 및 기타 권장 성능은 [Azure Virtual Machines의 SQL Server에 대한 성능 모범 사례][sql-performance]를 참조하세요. 또한 [SharePoint Server 2016 팜의 SQL Server에 대한 모범 사례][sql-sharepoint-best-practices]의 권장 사항을 따르세요.

주 노드 서버는 복제 파트너와 분리된 별도의 컴퓨터에 상주하는 것이 좋습니다. 서버는 보호 우선 모드 세션의 보조 복제 파트너 서버가 자동 장애 조치(failover)의 시작 여부를 인식하도록 지원합니다. 두 파트너와는 달리, 주 노드 서버는 데이터베이스를 제공하지 않는 대신 자동 장애 조치(failover)를 지원합니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

기존 서버를 강화하려면 VM 크기를 변경하기만 하면 됩니다.

SharePoint Server 2016의 [MinRoles][minroles] 기능을 사용하면 서버의 역할을 기반으로 서버를 규모 확장하고 역할에서 서버를 제거할 수 있습니다. 서버를 역할에 추가할 때 원하는 단일 역할 또는 복합 역할을 지정할 수 있습니다. 하지만 검색 역할에 서버를 추가하는 경우 PowerShell을 사용하여 검색 토폴로지도 다시 구성해야 합니다. MinRoles을 사용하여 역할을 변환할 수도 있습니다. 자세한 내용은 [SharePoint Server 2016에서 MinRole 서버 팜 관리][sharepoint-minrole]를 참조하세요.

SharePoint Server 2016은 자동 크기 조정에 가상 머신 확장 집합을 사용하는 것을 지원하지 않습니다.

## <a name="availability-considerations"></a>가용성 고려 사항

이 참조 아키텍처는 역할마다 2개 이상의 VM이 가용성 집합에 배포되기 때문에 Azure 지역 내에서 고가용성을 지원합니다.

지역 장애를 방지하려면 다른 Azure 지역에 별도의 재해 복구 팜을 만듭니다. RTO(복구 시간 목표) 및 RPO(복구 지점 목표)에 따라 설치 요구 사항이 결정됩니다. 자세한 내용은 [SharePoint 2016에 대한 재해 복구 전략 선택][sharepoint-dr]을 참조하세요. 보조 지역은 주 지역과 *쌍을 이루는 지역*이어야 합니다. 광범위한 중단 발생 시 한 지역의 복구가 모든 쌍의 복구보다 높은 우선 순위를 갖습니다. 자세한 내용은 [BCDR(비즈니스 연속성 및 재해 복구): Azure 쌍을 이루는 지역][paired-regions]을 참조하세요.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

서버, 서버 팜 및 사이트를 운영하고 유지 관리하려면 SharePoint 운영에 대한 다음 권장 사례를 따르세요. 자세한 내용은 [SharePoint Server 2016 운영][sharepoint-ops]을 참조하세요.

SharePoint 환경에서 SQL Server를 관리할 때 고려할 작업은 데이터베이스 응용 프로그램의 일반적인 고려 사항과 다를 수 있습니다. 야간 증분 백업으로 매주 모든 SQL 데이터베이스 전체를 백업하는 것이 모범 사례입니다. 15분마다 백업 트랜잭션이 로깅됩니다. 또 다른 방법으로 기본 제공 SharePoint 작업을 비활성화하고 데이터베이스에 SQL Server 유지 관리 작업을 구현할 수 있습니다. 자세한 내용은 [저장소 및 SQL Server 용량 계획 및 구성][sql-server-capacity-planning]을 참조하세요.

## <a name="security-considerations"></a>보안 고려 사항

SharePoint Server 2016을 실행하는 데 사용되는 도메인 수준 서비스 계정은 도메인 가입 및 인증 프로세스를 처리하기 위한 Windows Server AD 도메인 컨트롤러가 필요합니다. Azure Active Directory Domain Services는 이 용도로 사용할 수 없습니다. 이미 인트라넷에서 가동 중인 Windows Server AD ID 인프라를 확장하기 위해 이 아키텍처는 기존 온-프레미스 Windows Server AD 포리스트의 두 Windows Server AD 복제 도메인 컨트롤러를 사용합니다.

또한 항상 보안 강화를 계획하는 것이 좋습니다. 기타 권장 사항으로 다음과 같은 것들이 있습니다.

- 서브넷 및 역할을 격리하는 규칙을 NSG에 추가합니다.
- VM에 공용 IP 주소를 할당하지 않습니다.
- 침입 감지 및 페이로드 분석의 경우 내부 Azure 부하 분산 장치 대신 프런트 엔드 웹 서버 앞에 네트워크 가상 어플라이언스를 사용하는 방안을 고려합니다.
- 필요에 따라 서버 간 일반 텍스트 트래픽을 암호화하는 IPsec 정책을 사용합니다. 서브넷도 격리하려는 경우 IPsec 트래픽을 허용하도록 네트워크 보안 그룹 규칙을 업데이트합니다.
- VM을 위한 맬웨어 방지 에이전트를 설치합니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 참조 아키텍처에 대한 배포는 [GitHub][github]에서 사용할 수 있습니다. 전체 배포를 완료하는 데 몇 시간이 걸릴 수 있습니다.

배포는 구독에서 다음과 같은 리소스 그룹을 만듭니다.

- ra-onprem-sp2016-rg
- ra-sp2016-network-rg

템플릿 매개 변수 파일은 이러한 이름을 참조하므로 이름을 변경하는 경우 매개 변수 파일을 일치하도록 업데이트합니다.

매개 변수 파일은 다양한 위치에서 하드 코딩된 암호를 포함하고 있습니다. 배포하기 전에 이러한 값을 변경하세요.

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deployment-steps"></a>배포 단계

1. 시뮬레이션된 온-프레미스 네트워크를 배포하려면 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p onprem.json --deploy
    ```

2. Azure VNet 및 VPN Gateway를 배포하려면 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p connections.json --deploy
    ```

3. Jumpbox, AD 도메인 컨트롤러 및 SQL Server VM을 배포하려면 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure1.json --deploy
    ```

4. 장애 조치 클러스터 및 가용성 그룹을 만들려면 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure2-cluster.json --deploy
    ```

5. 다음 명령을 실행하여 나머지 VM을 배포합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure3.json --deploy
    ```

이 경우에 SQL Server Always On 가용성 그룹의 경우 웹 프런트 엔드에서 부하 분산 장치로 TCP를 연결할 수 있는지 확인합니다. 이렇게 하려면 다음 단계를 수행합니다.

1. Azure Portal을 사용하여 `ra-sp2016-network-rg` 리소스 그룹에서 `ra-sp-jb-vm1`이라는 VM을 찾습니다. 이것이 Jumpbox VM입니다.

2. `Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다. `azure1.json` 매개 변수 파일에서 지정한 암호를 사용합니다.

3. 원격 데스크톱 세션에서 10.0.5.4로 로그인합니다. `ra-sp-app-vm1`이라는 VM의 IP 주소입니다.

4. VM에서 PowerShell 콘솔을 열고, `Test-NetConnection` cmdlet을 사용하여 부하 분산 장치에 연결할 수 있는지 확인합니다.

    ```powershell
    Test-NetConnection 10.0.3.100 -Port 1433
    ```

출력은 다음과 비슷해야 합니다.

```console
ComputerName     : 10.0.3.100
RemoteAddress    : 10.0.3.100
RemotePort       : 1433
InterfaceAlias   : Ethernet 3
SourceAddress    : 10.0.0.132
TcpTestSucceeded : True
```

실패한 경우 Azure Portal을 사용하여 `ra-sp-sql-vm2`라는 VM을 다시 시작합니다. VM을 다시 시작한 후에 `Test-NetConnection` 명령을 다시 실행합니다. 연결이 성공하려면 VM이 다시 시작된 후에 1분 정도 대기해야 합니다.

이제 다음과 같은 배포를 완료합니다.

1. SharePoint 팜 주 노드를 배포하려면 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure4-sharepoint-server.json --deploy
    ```

2. SharePoint 캐시, 검색 및 웹을 배포하려면 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure5-sharepoint-farm.json --deploy
    ```

3. NSG 규칙을 만들려면 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure6-security.json --deploy
    ```

### <a name="validate-the-deployment"></a>배포 유효성 검사

1. [Azure Portal][azure-portal]에서 `ra-onprem-sp2016-rg` 리소스 그룹으로 이동합니다.

2. 리소스 목록에서 `ra-onpr-u-vm1`이라는 VM 리소스를 선택합니다.

3. [가상 머신에 연결][connect-to-vm]의 설명에 따라 VM을 연결합니다. 사용자 이름은 `\onpremuser`입니다.

4. VM에 대한 원격 연결이 설정되면 VM에서 브라우저를 열고 `http://portal.contoso.local`로 이동합니다.

5. **Windows 보안** 상자에서 사용자 이름으로 `contoso.local\testuser`를 사용하여 SharePoint 포털에 로그온합니다.

이 로그온은 온-프레미스 네트워크에서 사용하는 Fabrikam.com 도메인에서 SharePoint 포털이 사용하는 contoso.local 도메인으로 터널링합니다. SharePoint 사이트가 열리면 루트 데모 사이트가 보일 것입니다.

**_이 참조 아키텍처에 대한 기여자_** &mdash; Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
