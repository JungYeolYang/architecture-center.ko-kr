---
title: Azure로 온-프레미스 AD FS 확장
titleSuffix: Azure Reference Architectures
description: Azure에서 Active Directory 페더레이션 서비스 권한 부여로 보안 하이브리드 네트워크 아키텍처를 구현합니다.
author: telmosampaio
ms.date: 12/18.2018
ms.custom: seodec18
ms.openlocfilehash: 0cdb8ce61c48189a7b708dfa022a68080040ec3d
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111846"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>Azure로 AD FS(Active Directory Federation Services) 확장

이 참조 아키텍처는 Azure로 온-프레미스 네트워크를 확장하는 보안 하이브리드 네트워크를 구현하고 [AD FS(Active Directory Federation Services)][active-directory-federation-services]를 사용하여 Azure에서 실행되는 구성 요소에 대한 페더레이션 인증 및 권한 부여를 수행합니다. [**이 솔루션을 배포합니다**](#deploy-the-solution).

![Active Directory로 하이브리드 네트워크 아키텍처 보안](./images/adfs.png)

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

AD FS는 온-프레미스에서 호스팅될 수 있지만 애플리케이션이 일부 부분이 Azure에서 구현되는 하이브리드인 경우 클라우드에서 AD FS를 복제하는 것이 더 효율적일 수 있습니다.

다이어그램은 다음 시나리오를 보여 줍니다.

- 파트너 조직에서 애플리케이션 코드는 Azure VNet 내에서 호스팅되는 웹 애플리케이션에 액세스합니다.
- Active Directory DS(Domain Services) 내부에 저장된 자격 증명으로 등록된 외부 사용자는 Azure VNet 내에서 호스팅되는 웹 애플리케이션에 액세스합니다.
- 인증된 장치를 사용하여 VNet에 연결된 사용자는 Azure VNet 내에서 호스팅되는 웹 애플리케이션을 실행합니다.

이 아키텍처의 일반적인 용도는 다음과 같습니다.

- 작업이 부분적으로 온-프레미스 및 부분적으로 Azure에서 실행되는 하이브리드 애플리케이션
- 페더레이션된 권한 부여를 사용하여 파트너 조직에 웹 애플리케이션을 노출하는 솔루션
- 조직 방화벽 외부에서 실행되는 웹 브라우저에서 액세스를 지원하는 시스템
- 원격 컴퓨터, 노트북 및 다른 모바일 장치와 같은 승인된 외부 장치에서 연결하여 사용자가 웹 애플리케이션에 액세스할 수 있도록 하는 시스템

이 참조 아키텍처는 페더레이션 서버가 사용자를 인증하는 방법 및 시기를 결정하는 *수동 페더레이션*에 중점을 둡니다. 사용자는 애플리케이션이 시작될 때 로그인 정보를 제공합니다. 이 메커니즘은 웹 브라우저에서 가장 일반적으로 사용되며 브라우저를 사용자가 인증하는 사이트로 리디렉션하는 프로토콜을 포함합니다. AD FS는 또한 *활성 페더레이션*을 지원합니다. 여기서 애플리케이션은 추가 사용자 상호 작용 없이 자격 증명을 제공해야 하지만 해당 시나리오는 이 아키텍처의 범위를 벗어납니다.

추가 고려 사항은 [온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택][considerations]을 참조하세요.

## <a name="architecture"></a>아키텍처

이 아키텍처는 [Azure로 AD DS 확장][extending-ad-to-azure]에 설명된 구현을 확장합니다. 여기에는 다음 구성 요소가 포함됩니다.

- **AD DS 서브넷** AD DS 서버는 방화벽 역할을 하는 NSG(네트워크 보안 그룹) 규칙과 함께 자체의 서브넷에 포함됩니다.

- **AD DS 서버** Azure에서 VM으로 실행되는 도메인 컨트롤러 이러한 서버는 도메인 내에서 로컬 ID의 인증을 제공합니다.

- **AD FS 서브넷** AD FS 서버는 방화벽 역할을 하는 NSG 규칙과 함께 자체의 서브넷 내에 있습니다.

- **AD FS 서버** AD FS 서버는 페더레이션된 인증 및 권한 부여를 제공합니다. 이 아키텍처에서 다음 작업을 수행합니다.

  - 파트너 사용자를 대신하여 파트너 페더레이션 서버에서 만들어진 클레임을 포함하는 보안 토큰 받기 AD FS는 요청에 권한을 부여하기 위해 Azure에서 실행되는 웹 애플리케이션에 클레임을 전달하기 전에 토큰이 유효한지 확인합니다.

    Azure에서 실행되는 애플리케이션은 *신뢰 당사자*입니다. 파트너 페더레이션 서버는 웹 애플리케이션에서 인식할 수 있는 클레임을 발급해야 합니다. 파트너 페더레이션 서버는 파트너 조직에서 인증된 계정을 대신하여 액세스 요청을 제출하기 때문에 *계정 파트너*라고 합니다. AD FS 서버는 리소스(웹 애플리케이션)에 대한 액세스를 제공하기 때문에 *리소스 파트너*라고 합니다.

  - AD DS 및 [Active Directory Device Registration Service][ADDRS]를 사용하여 웹 브라우저를 실행하는 외부 사용자 또는 웹 애플리케이션에 대한 액세스가 필요한 장치에서 들어오는 요청 인증 및 권한 부여

  AD FS 서버는 Azure 부하 분산 장치를 통해 액세스되는 팜으로 구성됩니다. 이 구현은 가용성과 확장성을 향상시킵니다. AD FS 서버는 인터넷에 직접 노출되지 않습니다. 모든 인터넷 트래픽은 AD FS 웹 애플리케이션 프록시 서버 및 DMZ(경계 네트워크라고도 함)를 통해 필터링됩니다.

  AD FS가 작동하는 방법에 대한 자세한 내용은 [Active Directory Federation Services 개요][active-directory-federation-services-overview]를 참조하세요. 또한 [Azure에서 AD FS 배포][adfs-intro] 문서는 구현에 대한 자세한 단계별 소개를 포함합니다.

- **AD FS 프록시 서브넷** AD FS 프록시 서버는 보호를 제공하는 NSG 규칙과 함께 자체의 서브넷 내에서 포함될 수 있습니다. 이 서브넷의 서버는 Azure 가상 네트워크와 인터넷 간 방화벽을 제공하는 네트워크 가상 어플라이언스의 집합을 통해 인터넷에 노출됩니다.

- **AD FS WAP(웹 애플리케이션 프록시) 서버** 이러한 VM은 파트너 조직 및 외부 디바이스에서 들어오는 요청에 대한 AD FS 서버로 작동합니다. WAP 서버는 AD FS 서버를 인터넷의 직접 액세스에서 보호하는 필터로 작동합니다. AD FS 서버와 마찬가지로 부하 분산으로 팜에서 WAP 서버를 배포하는 것은 독립 실행형 서버 컬렉션을 배포하는 것보다 더 큰 가용성 및 확장성을 제공합니다.

  > [!NOTE]
  > WAP 서버 설치에 대한 자세한 내용은 [웹 애플리케이션 프록시 서버 설치 및 구성][install_and_configure_the_web_application_proxy_server]을 참조하세요.
  >

- **파트너 조직** Azure에서 실행되는 웹 애플리케이션에 대한 액세스를 요청하는 웹 애플리케이션을 실행하는 파트너 조직 파트너 조직의 페더레이션 서버는 요청을 로컬로 인증하고 Azure에서 실행되는 AD FS에 대한 클레임을 포함하는 보안 토큰을 제출합니다. Azure에서 AD FS는 보안 토큰의 유효성을 검사하고 유효한 경우 인증을 위해 Azure에서 실행되는 웹 애플리케이션에 클레임을 전달할 수 있습니다.

  > [!NOTE]
  > 또한 Azure 게이트웨이를 사용하여 신뢰할 수 있는 파트너에게 AD FS에 대한 직접 액세스를 제공하도록 VPN 터널을 구성할 수 있습니다. 이러한 파트너에서 받은 요청은 WAP 서버를 통해 전달하지 마십시오.
  >

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.

### <a name="networking-recommendations"></a>네트워킹 권장 사항

정적 개인 IP 주소로 AD FS 및 WAP 서버를 호스팅하는 각 VM에 대한 네트워크 인터페이스를 구성합니다.

AD FS VM에 공용 IP 주소를 제공하지 마십시오. 자세한 내용은 [보안 고려 사항](#security-considerations) 섹션을 참조하세요.

Active Directory DS VM을 참조하기 위해 각 AD FS 및 WAP VM의 네트워크 인터페이스에 대한 기본 및 보조 DNS(도메인 이름 서비스) 서버의 IP 주소를 설정합니다. Active Directory DS VM은 DNS를 실행해야 합니다. 이 단계는 각 VM을 도메인에 조인하기 위해 필요합니다.

### <a name="ad-fs-installation"></a>AD FS 설치

[페더레이션 서버 팜 배포][Deploying_a_federation_server_farm] 문서는 AD FS 설치 및 구성에 대한 자세한 지침을 제공합니다. 팜의 첫 번째 AD FS 서버를 구성하기 전에 다음 작업을 수행합니다.

1. 서버 인증을 수행하기 위해 공개적으로 신뢰할 수 있는 인증서를 가져옵니다. *주체 이름*은 페더레이션 서비스에 액세스하는 데 사용하는 이름 클라이언트를 포함해야 합니다. 부하 분산에 대해 등록된 DNS 이름일 수 있습니다(예: *adfs.contoso.com*). (보안상의 이유로 **.contoso.com*과 같은 와일드카드 이름 사용을 피합니다.) 모든 AD FS 서버 VM에 동일한 인증서를 사용합니다. 신뢰할 수 있는 인증 기관에서 인증서를 구입할 수 있지만 조직에서 Active Directory Certificate Services를 사용하는 경우 직접 만들 수 있습니다.

    *주체 대체 이름*은 외부 디바이스에서 액세스할 수 있도록 DRS(디바이스 등록 서비스)에서 사용됩니다. *enterpriseregistration.contoso.com* 형식이어야 합니다.

    자세한 내용은 [AD FS에 대한 SSL(Secure Sockets Layer) 인증서 가져오기 및 구성][adfs_certificates]을 참조하세요.

2. 도메인 컨트롤러에서 키 배포 서비스에 대한 새 루트 키를 생성합니다. 10시간을 뺀 현재 시간으로 유효한 시간을 설정합니다.(이 구성은 도메인에서 키 배포 및 동기화에서 발생할 수 있는 지연을 줄입니다.) 이 단계는 AD FS 서비스를 실행하는 데 사용되는 그룹 서비스 계정 만들기를 지원하는 데 필요합니다. 다음 PowerShell 명령은 이 작업을 수행하는 방법의 예를 보여 줍니다.

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. 각 AD FS 서버 VM을 도메인에 추가합니다.

> [!NOTE]
> AD FS를 설치하려면 도메인에 대한 PDC(주 도메인 컨트롤러) 에뮬레이터 FSMO(유연한 단일 마스터 작업) 역할을 실행하는 도메인 컨트롤러는 AD FS VM에서 실행되고 액세스할 수 있어야 합니다. <<RBC: 덜 반복적으로 만드는 방법이 있나요?>>
>

### <a name="ad-fs-trust"></a>AD FS 신뢰

AD FS 설치 및 모든 파트너 조직의 페더레이션 서버 간에 페더레이션 트러스트를 설정합니다. 필요한 클레임 필터링 및 매핑을 구성합니다.

- 각 파트너 조직의 DevOps 직원은 AD FS 서버를 통해 액세스할 수 있는 웹 애플리케이션에 대한 신뢰 당사자 트러스트를 추가해야 합니다.
- 조직의 DevOps 직원은 AD FS 서버가 파트너 조직에서 제공하는 클레임을 신뢰할 수 있도록 클레임 공급자 트러스트를 구성해야 합니다.
- 또한 조직의 DevOps 직원은 조직의 웹 애플리케이션에 클레임을 전달하도록 AD FS를 구성해야 합니다.

자세한 내용은 [페더레이션 트러스트 설정][establishing-federation-trust]을 참조하세요.

조직의 웹 애플리케이션을 게시하고 WAP 서버를 통해 사전 인증을 사용하여 외부 파트너에 사용할 수 있도록 합니다. 자세한 내용은 [AD FS 사전 인증을 사용하여 애플리케이션 게시][publish_applications_using_AD_FS_preauthentication]를 참조하세요.

AD FS는 토큰 변환 및 확대를 지원합니다. Azure Active Directory는 이 기능을 제공하지 않습니다. AD FS를 사용하여 트러스트 관계를 설정할 때 다음을 수행할 수 있습니다.

- 권한 부여 규칙에 대한 클레임 변환을 구성합니다. 예를 들어 비 Microsoft 파트너 조직에서 사용되는 표현에서 조직에서 해당 Active Directory DS가 조직에서 권한을 부여할 수 있는 것으로 그룹 보안을 매핑할 수 있습니다.
- 클레임을 한 형식에서 다른 형식으로 변환합니다. 예를 들어 애플리케이션이 SAML 1.1 클레임만을 지원하는 경우 SAML 2.0에서 SAML 1.1로 매핑할 수 있습니다.

### <a name="ad-fs-monitoring"></a>AD FS 모니터링

[Active Directory Federation Services 2012 R2용 Microsoft System Center 관리 팩][oms-adfs-pack]은 페더레이션 서버에 대한 AD FS 배포의 사전 예방적이며 반응적인 모니터링을 제공합니다. 이 관리 팩은 다음을 모니터링합니다.

- AD FS 서비스가 해당 이벤트 로그에 기록하는 이벤트
- AD FS 성능 카운터가 수집하는 성능 데이터
- AD FS 시스템 및 웹 애플리케이션(신뢰 당사자)의 전반적인 상태, 중요한 문제 및 경고에 대한 경고를 제공합니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

[AD FS 배포 계획][plan-your-adfs-deployment] 문서에서 요약된 다음 고려 사항은 AD FS 팜 크기 조정에 대한 시작 지점을 제공합니다.

- 1000명의 사용자보다 적은 경우 전용 서버를 만들지 마십시오. 대신 클라우드의 각 Active Directory DS 서버에 AD FS를 설치합니다. 가용성을 유지하기 위해 두 개 이상의 Active Directory DS 서버가 있는지 확인합니다. 단일 WAP 서버를 만듭니다.
- 1000명에서 15000명 사이의 사용자가 있는 경우 두 개의 전용 AD FS 서버 및 두 개의 전용 WAP 서버를 만듭니다.
- 15000명에서 60000명 사이의 사용자가 있는 경우 세 개에서 다섯 개 사이의 전용 AD FS 서버 및 두 개 이상의 전용 WAP 서버를 만듭니다.

이러한 고려 사항은 Azure에서 듀얼 쿼드 코어 VM(표준 D4_v2 또는 그 이상) 크기를 사용하고 있다고 가정합니다.

AD FS 구성 데이터를 저장하는 데 Windows 내부 데이터베이스를 사용하는 경우 팜에서 8개의 AD FS 서버로 제한됩니다. 나중에 더 필요할 것이라고 예상하는 경우 SQL Server를 사용합니다. 자세한 내용은 [AD FS 구성 데이터베이스의 역할][adfs-configuration-database]을 참조하세요.

## <a name="availability-considerations"></a>가용성 고려 사항

서비스의 가용성 향상을 위해 두 개 이상의 서버와 함께 AD FS 팜을 만듭니다. 팜의 각 AD FS VM에 대해 다른 저장소 계정을 사용합니다. 이 방법을 사용하면 단일 저장소 계정의 실패가 전체 팜을 액세스할 수 없도록 하지 않습니다.

AD FS 및 WAP VM에 대한 별도 Azure 가용성 집합을 만듭니다. 각 집합에 두 개 이상의 VM이 있는지 확인합니다. 각 가용성 집합은 두 개 이상의 업데이트 도메인 및 두 개의 장애 도메인이 있어야 합니다.

다음과 같이 AD FS VM 및 WAP VM에 대한 부하 분산 장치를 구성합니다.

- WAP VM에 대한 외부 액세스를 제공하는 Azure 부하 분산 장치 및 팜의 AD FS 서버 간에 부하를 분산하는 내부 부하 분산 장치를 사용합니다.
- 포트 443(HTTPS)에 표시되는 트래픽을 AD FS/WAP 서버에 전달합니다.
- 부하 분산 장치에 고정 IP 주소를 지정합니다.
- `/adfs/probe`에 대해 HTTP를 사용하여 상태 프로브를 만듭니다. 자세한 내용은 [하드웨어 Load Balancer 상태 검사 및 웹 애플리케이션 프록시 / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/)를 참조합니다.

  > [!NOTE]
  > AD FS 서버는 SNI(서버 이름 표시) 프로토콜을 사용하므로 부하 분산 장치에서 HTTPS 엔드포인트를 사용하는 프로브에 대한 시도는 실패합니다.
  >

- AD FS 부하 분산 장치에 대한 도메인에 DNS *A* 레코드를 추가합니다. 부하 분산 장치의 IP 주소를 지정하고 도메인의 이름을 지정합니다(예: adfs.contoso.com). 이는 AD FS 서버 팜에 액세스하는 데 사용하는 이름 클라이언트 및 WAP 서버입니다.

SQL Server 또는 Windows 내부 데이터베이스를 사용하여 AD FS 구성 정보를 보관할 수 있습니다. Windows 내부 데이터베이스는 기본 중복성을 제공합니다. 변경 내용은 AD FS 클러스터의 AD FS 데이터베이스 중 하나에 직접 기록되는 반면 다른 서버는 끌어오기 복제를 사용하여 해당 데이터베이스를 최신 상태로 유지합니다. SQL Server를 사용하면 장애 조치(failover) 클러스터링 또는 미러링을 사용하여 전체 데이터베이스 중복성 및 고가용성을 제공할 수 있습니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

DevOps 직원은 다음 작업을 수행할 준비가 되어 있어야 합니다.

- AD FS 팜 관리, 페더레이션 서버의 트러스트 정책 관리 및 페더레이션 서비스에서 사용되는 인증서 관리를 포함하는 페더레이션 서버 관리
- WAP 팜 및 인증서 관리를 포함하는 WAP 서버 관리
- 신뢰 당사자, 인증 방법 및 클레임 매핑 구성을 포함하는 웹 애플리케이션 관리
- AD FS 구성 요소 백업

## <a name="security-considerations"></a>보안 고려 사항

AD FS는 HTTPS를 사용하므로 웹 계층 VM을 포함하는 서브넷에 대한 NSG 규칙이 HTTPS 요청을 허용하는지 확인합니다. 이러한 요청은 온-프레미스 네트워크, 웹 계층, 비즈니스 계층, 데이터 계층, 개인 DMZ, 공용 DMZ를 포함하는 서브넷 및 AD FS 서버를 포함하는 서브넷에서 발생할 수 있습니다.

인터넷에 대한 AD FS 서버의 직접 노출을 방지합니다. AD FS 서버는 보안 토큰을 부여하는 완전한 권한이 있는 도메인에 가입된 컴퓨터입니다. 서버가 손상되면 악의적인 사용자가 모든 웹 애플리케이션 및 AD FS로 보호되는 모든 페더레이션 서버에 전체 액세스 토큰을 발급할 수 있습니다. 시스템이 신뢰할 수 있는 파트너 사이트에서 연결하지 않은 외부 사용자의 요청을 처리해야 하는 경우 WAP 서버를 사용하여 이러한 요청을 처리합니다. 자세한 내용은 [페더레이션 서버 프록시를 배치할 위치][where-to-place-an-fs-proxy]를 참조하세요.

AD FS 서버와 WAP 서버를 자체 방화벽이 있는 별도 서브넷에 배치합니다. NSG 규칙을 사용하여 방화벽 규칙을 정의할 수 있습니다. 모든 방화벽은 포트 443(HTTPS)의 트래픽을 허용해야 합니다.

AD FS 및 WAP 서버에 대한 직접 로그인 액세스를 제한합니다. DevOps 직원만 연결할 수 있어야 합니다. WAP 서버를 도메인에 조인하지 마십시오.

감사를 목적으로 가상 네트워크의 에지를 탐색하는 트래픽의 자세한 정보를 기록하는 네트워크 가상 어플라이언스의 집합을 사용하는 것이 좋습니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 아키텍처에 대한 배포는 [GitHub][github]에서 사용할 수 있습니다. 전체 배포는 최대 2시간이 걸릴 수 있으며 VPN 게이트웨이 만들기 및 Active Directory 및 AD FS를 구성하는 스크립트 실행을 포함합니다.

### <a name="prerequisites"></a>필수 조건

1. [GitHub 리포지토리](https://github.com/mspnp/identity-reference-architectures)의 zip 파일을 복제, 포크 또는 다운로드합니다.

1. [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest)을 설치합니다.

1. [Azure 빌딩 블록](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm 패키지를 설치합니다.

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

1. 명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음과 같은 Azure 계정에 로그인합니다.

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>시뮬레이션된 온-프레미스 데이터 센터 배포

1. GitHub 리포지토리의 `adfs` 폴더로 이동합니다.

1. `onprem.json` 파일을 엽니다. `adminPassword`, `Password` 및 `SafeModeAdminPassword` 인스턴스를 검색하고 암호를 업데이트합니다.

1. 다음 명령을 실행하고 배포가 끝나기를 기다립니다.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-infrastructure"></a>Azure 인프라 배포

1. `azure.json` 파일을 엽니다.  `adminPassword` 및 `Password` 인스턴스를 검색하고 암호 값을 추가합니다.

1. 다음 명령을 실행하고 배포가 끝나기를 기다립니다.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

### <a name="set-up-the-ad-fs-farm"></a>AD FS 팜 설정

1. `adfs-farm-first.json` 파일을 엽니다.  `AdminPassword`를 검색하고 기본 암호를 바꿉니다.

1. 다음 명령 실행:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-first.json --deploy
    ```

1. `adfs-farm-rest.json` 파일을 엽니다.  `AdminPassword`를 검색하고 기본 암호를 바꿉니다.

1. 다음 명령을 실행하고 배포가 끝나기를 기다립니다.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-rest.json --deploy
    ```

### <a name="configure-ad-fs-part-1"></a>AD FS 구성(1부)

1. `ra-adfs-jb-vm1`이라는 VM(jumpbox VM)에 원격 데스크톱 세션을 엽니다. 사용자 이름은 `testuser`입니다.

1. jumpbox에서 `ra-adfs-proxy-vm1`이라는 VM에 대한 원격 데스크톱 세션을 엽니다. 사설 IP 주소는 10.0.6.4입니다.

1. 이 원격 데스크톱 세션에서 [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-)를 실행합니다.

1. PowerShell에서 다음 디렉터리로 이동합니다.

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. 다음 코드를 복사하여 스크립트 창에 붙여넣고 실행합니다.

    ```powershell
    . .\adfs-webproxy.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxyApp -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxyApp
    ```

    `Get-Credential` 프롬프트에 배포 매개 변수 파일에 지정한 암호를 입력합니다.

1. [DSC](/powershell/dsc/overview/overview) 구성의 진행률을 모니터링하려면 다음 명령을 실행합니다.

    ```powershell
    Get-DscConfigurationStatus
    ```

    일관성을 구현하는 데 몇 분 정도 걸릴 수 있습니다. 이때 명령에서 오류가 표시될 수 있습니다. 구성에 성공하면 다음과 비슷한 출력이 표시됩니다.

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

### <a name="configure-ad-fs-part-2"></a>AD FS 구성(2부)

1. jumpbox에서 `ra-adfs-proxy-vm2`이라는 VM에 대한 원격 데스크톱 세션을 엽니다. 사설 IP 주소는 10.0.6.5입니다.

1. 이 원격 데스크톱 세션에서 [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-)를 실행합니다.

1. 다음 디렉터리로 이동합니다.

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. 스크립트 창에 다음을 전달하고 스크립트를 실행합니다.

    ```powershell
    . .\adfs-webproxy-rest.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxy -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxy
    ```

    `Get-Credential` 프롬프트에 배포 매개 변수 파일에 지정한 암호를 입력합니다.

1. DSC 구성의 진행률을 모니터링하려면 다음 명령을 실행합니다.

    ```powershell
    Get-DscConfigurationStatus
    ```

    일관성을 구현하는 데 몇 분 정도 걸릴 수 있습니다. 이때 명령에서 오류가 표시될 수 있습니다. 구성에 성공하면 다음과 비슷한 출력이 표시됩니다.

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

    이 DSC가 실패하는 경우도 있습니다. 상태 확인 결과에 `Status=Failure` 및 `Type=Consistency`가 표시될 경우 4단계를 다시 실행해 보세요.

### <a name="sign-into-ad-fs"></a>AD FS에 로그인

1. jumpbox에서 `ra-adfs-adfs-vm1`이라는 VM에 대한 원격 데스크톱 세션을 엽니다. 사설 IP 주소는 10.0.5.4입니다.

1. 로그온 페이지를 사용하려면 [Idp 시작 로그온 페이지 사용](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page)의 단계를 따르세요.

1. jumpbox에서 `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`으로 이동합니다. 이 테스트에 대해 무시할 수 있는 인증서 경고가 표시될 수 있습니다.

1. Contoso Corporation 로그인 페이지가 표시되는지 확인합니다. **contoso\testuser**로 로그인합니다.

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: /windows-server/identity/active-directory-federation-services
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: /powershell/azure/overview
[aad]: /azure/active-directory/
[aadb2c]: /azure/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
