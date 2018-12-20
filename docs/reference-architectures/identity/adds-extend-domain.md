---
title: AD DS(Active Directory Domain Services)를 Azure로 확장
titleSuffix: Azure Reference Architectures
description: 온-프레미스 Active Directory 도메인을 Azure로 확장
author: telmosampaio
ms.date: 05/02/2018
ms.custom: seodec18
ms.openlocfilehash: 69ce95fcf74579f6446cf99dad9ed53ced31fde7
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120410"
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a>AD DS(Active Directory Domain Services)를 Azure로 확장

이 참조 아키텍처에서는 Active Directory 환경을 Azure로 확장하여 AD DS(Active Directory Domain Services)를 사용하여 분산 인증 서비스를 제공하는 방법을 설명합니다. [**이 솔루션을 배포합니다**](#deploy-the-solution).

![Active Directory로 하이브리드 네트워크 아키텍처 보안](./images/adds-extend-domain.png)

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

AD DS는 보안 도메인에 포함된 사용자, 컴퓨터, 응용 프로그램 및 기타 ID를 인증하는 데 사용됩니다. AD DS는 온-프레미스에서 호스팅할 수 있지만, 응용 프로그램의 일부는 온-프레미스에 호스팅되어 있고 일부는 Azure에 호스팅되어 있는 경우에는 이 기능을 Azure에 복제하는 것이 효율적일 수 있습니다. 이렇게 하면 클라우드에서 온-프레미스에서 실행 중인 AD DS로 인증 및 로컬 인증 요청을 돌려보낼 때 발생하는 대기 시간을 줄일 수 있습니다.

이 아키텍처는 일반적으로 온-프레미스 네트워크와 Azure 가상 네트워크가 VPN 또는 ExpressRoute 연결을 통해 연결된 경우에 사용됩니다. 이 아키텍처는 양방향 복제도 지원합니다. 즉, 온-프레미스나 클라우드 중 한 곳에서 변경을 수행해도 온-프레미스와 클라우드의 데이터가 동일하게 유지됩니다. 일반적으로 이 아키텍처는 온-프레미스와 Azure 사이에 기능이 분산된 하이브리드 응용 프로그램 및 Active Directory를 사용하여 인증을 수행하는 응용 프로그램과 서비스에 사용됩니다.

추가 고려 사항은 [온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택][considerations]을 참조하세요.

## <a name="architecture"></a>아키텍처

이 아키텍처는 [Azure와 인터넷 사이의 DMZ][implementing-a-secure-hybrid-network-architecture-with-internet-access]에 제시된 아키텍처를 확장합니다. 이 아키텍처에는 다음과 같은 구성 요소가 있습니다.

- **온-프레미스 네트워크**. 온-프레미스 네트워크에는 온-프레미스에 위치한 구성 요소에 대해 인증 및 권한 부여를 수행하는 로컬 Active Directory 서버가 포함됩니다.
- **Active Directory 서버**. 클라우드에서 VM으로서 실행되는 디렉터리 서비스(AD DS)를 구현하는 도메인 컨트롤러입니다. Active Directory 서버는 Azure 가상 네트워크에서 실행되는 구성 요소에 인증을 제공할 수 있습니다.
- **Active Directory 서브넷**. AD DS 서버는 별도의 서브넷에 호스팅됩니다. NSG(네트워크 보안 그룹) 규칙은 AD DS 서버를 보호하고 예상치 못한 소스로부터 전달되는 트래픽에 대한 방화벽을 제공합니다.
- **Azure 게이트웨이 및 Active Directory 동기화**. Azure 게이트웨이는 온-프레미스 네트워크와 Azure VNet 사이에 연결을 제공합니다. 이러한 연결은 [VPN 연결][azure-vpn-gateway] 또는 [Azure ExpressRoute][azure-expressroute]일 수 있습니다. 클라우드와 온-프레미스에 존재하는 Active Directory 서버 사이에 이루어지는 모든 동기화 요청은 게이트웨이를 통과합니다. UDR(사용자 정의 경로)이 Azure로 전달되는 온-프레미스 트래픽의 라우팅을 처리합니다. Active Directory 서버로 송수신되는 트래픽은 이 시나리오에서 사용되는 NVA(네트워크 가상 어플라이언스)를 통과하지 않습니다.

UDR 및 NVA 구성에 대한 자세한 내용은 [Azure에 보안 하이브리드 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture]을 참조하세요.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.

### <a name="vm-recommendations"></a>VM 권장 사항

예상되는 인증 요청의 양에 따라 [VM 크기][vm-windows-sizes] 요구 사항을 결정합니다. 온-프레미스에서 AD DS를 호스팅하는 머신의 사양을 출발점으로 삼아 이를 Azure VM 크기와 매칭합니다. 배포가 완료되면 VM에 가해지는 실제 부하를 바탕으로 사용률을 모니터링하고 확대 또는 축소합니다. AD DS 도메인 컨트롤러의 크기 결정에 대한 자세한 내용은 [Active Directory Domain Services의 용량 계획][capacity-planning-for-adds]을 참조하세요.

Active Directory의 데이터베이스, 로그 및 SYSVOL을 저장할 별도의 가상 데이터 디스크를 만듭니다. Active Directory의 데이터베이스, 로그 및 SYSVOL을 운영 체제와 동일한 디스크에 저장하지 않습니다. 기본적으로 VM에 연결된 데이터 디스크는 write-through 캐싱을 사용합니다. 그러나 이러한 유형의 캐싱은 AD DS의 요구 사항과 충돌할 수 있습니다. 따라서 데이터 디스크의 *호스트 캐시 기본 설정*을 *없음*으로 설정해야 합니다. 자세한 내용은 [Azure Virtual Machines에 Windows Server Active Directory를 배포하기 위한 지침][adds-data-disks]을 참조하세요.

도메인 컨트롤러로서 AD DS를 실행하는 VM을 2개 이상 배포하고 이를 [가용성 집합][availability-set]에 추가합니다.

### <a name="networking-recommendations"></a>네트워킹 권장 사항

각 AD DS 서버에 대해 완전한 DNS(도메인 이름 서비스)를 지원할 수 있도록 고정 사설 IP 주소를 사용하여 VM NIC(네트워크 인터페이스)를 구성합니다. 자세한 내용은 [Azure Portal에서 고정 사설 IP 주소를 설정하는 방법][set-a-static-ip-address]을 참조하세요.

> [!NOTE]
> AD DS에 대해 공용 IP 주소를 사용하여 VM NIC를 구성하지 않습니다. 자세한 내용은 [보안 고려 사항][security-considerations]을 참조하세요.
>

Active Directory 서브넷 NSG에는 온-프레미스에서 수신되는 트래픽을 허용하는 규칙이 필요합니다. AD DS에 의해 사용되는 포트에 대한 자세한 내용은 [Active Directory 및 Active Directory Domain Services 포트 요구 사항][ad-ds-ports]을 참조하세요. 또한, UDR 테이블이 이 아키텍처에서 사용되는 NVA를 통과하여 AD DS 트래픽을 라우팅하지 않도록 해야 합니다.

### <a name="active-directory-site"></a>Active Directory 사이트

AD DS에서 사이트는 물리적 위치, 네트워크 또는 장치 모음을 나타냅니다. AD DS 사이트는 서로 가까운 곳에 위치하며 고속 네트워크로 연결된 AD DS 개체를 그룹화하여 AD DS 데이터베이스 복제를 관리하는 데 사용됩니다. AD DS에는 여러 사이트 간에 AD DS 데이터베이스를 복제할 최상의 전략을 선택하는 로직이 포함되어 있습니다.

Azure상의 응용 프로그램에 대해 정의된 서브넷을 포함하여 AD DS 사이트를 만드는 것이 좋습니다. 그런 다음 여러 온-프레미스 AD DS 사이트 사이에 사이트 링크를 구성하면 AD DS가 가장 효율적인 데이터베이스 복제를 수행합니다. 이와 같은 데이터베이스 복제에는 초기 구성 외에 추가로 요구되는 사항이 거의 없습니다.

### <a name="active-directory-operations-masters"></a>Active Directory 작업 마스터

AD DS 도메인 컨트롤러에 작업 마스터 역할을 할당하여 복제된 여러 AD DS 데이터베이스 인스턴스 간의 일관성 확인을 지원할 수 있습니다. 작업 마스터에는 스키마 마스터, 도메인 이름 지정 마스터, 상대 식별자 마스터, 기본 도메인 컨트롤러 마스터 에뮬레이터, 인프라 마스터와 같은 다섯 가지 마스터 역할이 있습니다. 이러한 역할에 대한 자세한 내용은 [작업 마스터란?][ad-ds-operations-masters]을 참조하세요.

Azure에 배포한 도메인 컨트롤러에는 작업 마스터 역할을 할당하지 않는 것이 좋습니다.

### <a name="monitoring"></a>모니터링

도메인 컨트롤러 VM과 AD DS Services의 리소스를 모니터링하고 문제 발생 시 이를 신속하게 해결할 계획을 도출합니다. 자세한 내용은 [Active Directory 모니터링][monitoring_ad]을 참조하세요. 모니터링 서버에 [Microsoft Systems Center][microsoft_systems_center]와 같은 도구를 설치하여(아키텍처 다이어그램 참조) 이러한 작업을 수행할 수도 있습니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

AD DS는 확장성을 지원하도록 설계되었습니다. 요청을 AD DS 도메인 컨트롤러로 전송하기 위해 부하 분산 장치나 트래픽 컨트롤러를 구성할 필요가 없습니다. 염두에 두어야 할 유일한 확장성 고려 사항은 AD DS를 실행하는 VM에 네트워크 부하 요구 사항에 맞는 올바른 크기를 구성하고, VM의 부하를 모니터링하고, 필요에 따라 확장 또는 축소하는 것입니다.

## <a name="availability-considerations"></a>가용성 고려 사항

AD DS를 실행하는 VM을 [가용성 집합][availability-set]에 배포합니다. 또한, 필요에 따라 하나 또는 그 이상의 서버에 [대기 작업 마스터][standby-operations-masters] 역할을 할당하는 방법을 고려할 수도 있습니다. 대기 작업 마스터는 장애 조치(failover) 시에 기본 작업 마스터 서버 대신 사용할 수 있는 작업 마스터의 활성 복사본입니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

정식 AD DS 백업을 수행합니다. VHD상의 AD DS 데이터베이스 파일은 복사 시에 일관성 있는 상태가 아닐 수 있습니다. 이로 인해 데이터베이스를 다시 시작하지 못하게 될 수 있으므로 정식 백업을 수행하는 대신 도메인 컨트롤러의 VHD 파일을 복사하는 것은 지양해야 합니다.

Azure Portal을 사용하여 도메인 컨트롤러 VM을 종료하지 않습니다. 그 대신 게스트 운영 체제에서 VM을 종료하고 다시 시작합니다. 포털을 통해 종료하면 VM의 할당이 취소되고 Active Directory 리포지토리의 `VM-GenerationID` 및 `invocationID`가 재설정됩니다. 이 경우 AD DS RID(상대 식별자) 풀이 폐기되고 SYSVOL이 nonauthoritative로 표시되어 도메인 컨트롤러를 다시 구성해야 할 수 있습니다.

## <a name="security-considerations"></a>보안 고려 사항

AD DS 서버는 인증 서비스를 제공하기 때문에 공격자들의 매력적인 표적이 됩니다. AD DS 서버를 안전하게 보호하기 위해 AD DS 서버를 NSG가 방화벽으로 기능하는 별도의 서브넷에 배치함으로써 인터넷에 직접 연결되지 않도록 합니다. AD DS 서버에서 인증, 권한 부여 및 서버 동기화에 필요한 포트를 제외한 모든 포트를 닫습니다. 자세한 내용은 [Active Directory 및 Active Directory Domain Services 포트 요구 사항][ad-ds-ports]을 참조하세요.

[Azure에서 인터넷 액세스를 이용하여 보안 하이브리드 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture-with-internet-access]에 설명된 바와 같이 한 쌍의 서브넷과 NVA를 사용하여 서버에 추가적인 보안 경계를 구현하는 방법을 고려할 수 있습니다.

AD DS 데이터베이스를 호스팅하는 디스크를 BitLocker 또는 Azure disk encryption을 사용하여 암호화합니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 아키텍처에 대한 배포는 [GitHub][github]에서 사용할 수 있습니다. 전체 배포는 최대 2시간이 걸릴 수 있으며 VPN 게이트웨이 만들기 및 AD DS를 구성하는 스크립트 실행을 포함합니다.

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>시뮬레이션된 온-프레미스 데이터 센터 배포

1. GitHub 리포지토리의 `identity/adds-extend-domain` 폴더로 이동합니다.

2. `onprem.json` 파일을 엽니다. `adminPassword` 및 `Password` 인스턴스를 검색하고 암호 값을 추가합니다.

3. 다음 명령을 실행하고 배포가 끝나기를 기다립니다.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a>Azure VNet에 배포

1. `azure.json` 파일을 엽니다.  `adminPassword` 및 `Password` 인스턴스를 검색하고 암호 값을 추가합니다.

2. 동일한 파일에서 `sharedKey` 인스턴스를 검색하고 VPN 연결에 대한 공유 키를 입력합니다.

    ```json
    "sharedKey": "",
    ```

3. 다음 명령을 실행하고 배포가 끝나기를 기다립니다.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onoprem.json --deploy
    ```

   온-프레미스 VNet과 동일한 리소스 그룹에 배포합니다.

### <a name="test-connectivity-with-the-azure-vnet"></a>Azure VNet을 사용하여 연결 테스트

배포가 완료되면 시뮬레이션된 온-프레미스 환경에서 Azure VNet으로의 연결을 테스트할 수 있습니다.

1. Azure Portal을 사용하여 만든 리소스 그룹으로 이동합니다.

2. `ra-onpremise-mgmt-vm1`이라는 VM을 찾습니다.

3. `Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다. 사용자 이름은 `contoso\testuser`이고, 암호는 `onprem.json` 매개 변수 파일에서 지정한 것입니다.

4. 원격 데스크톱 세션 내에서 `adds-vm1`이라는 VM의 IP 주소인 10.0.4.4에 대한 다른 원격 데스크톱 세션을 엽니다. 사용자 이름은 `contoso\testuser`이고, 암호는 `azure.json` 매개 변수 파일에서 지정한 것입니다.

5. `adds-vm1`에 대한 원격 데스크톱 세션 내에서 **서버 관리자**로 이동하고 **관리할 다른 서버 추가**를 클릭합니다.

6. **Active Directory** 탭에서 **지금 찾기**를 클릭합니다. AD, AD DS 및 웹 VM의 목록이 표시됩니다.

   ![서버 추가 대화 상자의 스크린샷](./images/add-servers-dialog.png)

## <a name="next-steps"></a>다음 단계

- Azure에서 [AD DS 포리스트를 생성][adds-resource-forest]하기 위한 모범 사례를 알아봅니다.
- Azure에서 [AD FS(Active Directory Federation Services) 인프라를 생성][adfs]하기 위한 모범 사례를 알아봅니다.

<!-- links -->

[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[adds-data-disks]: https://msdn.microsoft.com/en-us/library/mt674703.aspx
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: /azure/expressroute/expressroute-introduction
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[capacity-planning-for-adds]: https://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/identity-reference-architectures/tree/master/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/download/details.aspx?id=50013
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: /azure/virtual-network/virtual-networks-static-private-ip-arm-pportal
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "Active Directory로 하이브리드 네트워크 아키텍처 보안"
