---
title: Azure로 온-프레미스 AD FS 확장
titleSuffix: Azure Reference Architectures
description: Azure에서 Active Directory 페더레이션 서비스 권한 부여로 보안 하이브리드 네트워크 아키텍처를 구현하는 방법.
author: telmosampaio
ms.date: 11/28/2016
ms.custom: seodec18
ms.openlocfilehash: 95866961cd92f44e0925c5e47eafdc5df71652db
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120223"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a><span data-ttu-id="3faef-103">Azure로 AD FS(Active Directory Federation Services) 확장</span><span class="sxs-lookup"><span data-stu-id="3faef-103">Extend Active Directory Federation Services (AD FS) to Azure</span></span>

<span data-ttu-id="3faef-104">이 참조 아키텍처는 Azure로 온-프레미스 네트워크를 확장하는 보안 하이브리드 네트워크를 구현하고 [AD FS(Active Directory Federation Services)][active-directory-federation-services]를 사용하여 Azure에서 실행되는 구성 요소에 대한 페더레이션 인증 및 권한 부여를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-104">This reference architecture implements a secure hybrid network that extends your on-premises network to Azure and uses [Active Directory Federation Services (AD FS)][active-directory-federation-services] to perform federated authentication and authorization for components running in Azure.</span></span> <span data-ttu-id="3faef-105">[**이 솔루션을 배포합니다**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="3faef-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Active Directory로 하이브리드 네트워크 아키텍처 보안](./images/adfs.png)

<span data-ttu-id="3faef-107">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="3faef-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="3faef-108">AD FS는 온-프레미스에서 호스팅될 수 있지만 애플리케이션이 일부 부분이 Azure에서 구현되는 하이브리드인 경우 클라우드에서 AD FS를 복제하는 것이 더 효율적일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-108">AD FS can be hosted on-premises, but if your application is a hybrid in which some parts are implemented in Azure, it may be more efficient to replicate AD FS in the cloud.</span></span>

<span data-ttu-id="3faef-109">다이어그램은 다음 시나리오를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-109">The diagram shows the following scenarios:</span></span>

- <span data-ttu-id="3faef-110">파트너 조직에서 애플리케이션 코드는 Azure VNet 내에서 호스팅되는 웹 애플리케이션에 액세스합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-110">Application code from a partner organization accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="3faef-111">Active Directory DS(Domain Services) 내부에 저장된 자격 증명으로 등록된 외부 사용자는 Azure VNet 내에서 호스팅되는 웹 애플리케이션에 액세스합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-111">An external, registered user with credentials stored inside Active Directory Domain Services (DS) accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="3faef-112">인증된 장치를 사용하여 VNet에 연결된 사용자는 Azure VNet 내에서 호스팅되는 웹 애플리케이션을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-112">A user connected to your VNet using an authorized device executes a web application hosted inside your Azure VNet.</span></span>

<span data-ttu-id="3faef-113">이 아키텍처의 일반적인 용도는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-113">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="3faef-114">작업이 부분적으로 온-프레미스 및 부분적으로 Azure에서 실행되는 하이브리드 애플리케이션</span><span class="sxs-lookup"><span data-stu-id="3faef-114">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="3faef-115">페더레이션된 권한 부여를 사용하여 파트너 조직에 웹 애플리케이션을 노출하는 솔루션</span><span class="sxs-lookup"><span data-stu-id="3faef-115">Solutions that use federated authorization to expose web applications to partner organizations.</span></span>
- <span data-ttu-id="3faef-116">조직 방화벽 외부에서 실행되는 웹 브라우저에서 액세스를 지원하는 시스템</span><span class="sxs-lookup"><span data-stu-id="3faef-116">Systems that support access from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="3faef-117">원격 컴퓨터, 노트북 및 다른 모바일 장치와 같은 승인된 외부 장치에서 연결하여 사용자가 웹 애플리케이션에 액세스할 수 있도록 하는 시스템</span><span class="sxs-lookup"><span data-stu-id="3faef-117">Systems that enable users to access to web applications by connecting from authorized external devices such as remote computers, notebooks, and other mobile devices.</span></span>

<span data-ttu-id="3faef-118">이 참조 아키텍처는 페더레이션 서버가 사용자를 인증하는 방법 및 시기를 결정하는 *수동 페더레이션*에 중점을 둡니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-118">This reference architecture focuses on *passive federation*, in which the federation servers decide how and when to authenticate a user.</span></span> <span data-ttu-id="3faef-119">사용자는 애플리케이션이 시작될 때 로그인 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-119">The user provides sign in information when the application is started.</span></span> <span data-ttu-id="3faef-120">이 메커니즘은 웹 브라우저에서 가장 일반적으로 사용되며 브라우저를 사용자가 인증하는 사이트로 리디렉션하는 프로토콜을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-120">This mechanism is most commonly used by web browsers and involves a protocol that redirects the browser to a site where the user authenticates.</span></span> <span data-ttu-id="3faef-121">AD FS는 또한 *활성 페더레이션*을 지원합니다. 여기서 애플리케이션은 추가 사용자 상호 작용 없이 자격 증명을 제공해야 하지만 해당 시나리오는 이 아키텍처의 범위를 벗어납니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-121">AD FS also supports *active federation*, where an application takes on responsibility for supplying credentials without further user interaction, but that scenario is outside the scope of this architecture.</span></span>

<span data-ttu-id="3faef-122">추가 고려 사항은 [온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택][considerations]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-122">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].-</span></span>

## <a name="architecture"></a><span data-ttu-id="3faef-123">아키텍처</span><span class="sxs-lookup"><span data-stu-id="3faef-123">Architecture</span></span>

<span data-ttu-id="3faef-124">이 아키텍처는 [Azure로 AD DS 확장][extending-ad-to-azure]에 설명된 구현을 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-124">This architecture extends the implementation described in [Extending AD DS to Azure][extending-ad-to-azure].</span></span> <span data-ttu-id="3faef-125">여기에는 다음 구성 요소가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-125">It contains the followign components.</span></span>

- <span data-ttu-id="3faef-126">**AD DS 서브넷**</span><span class="sxs-lookup"><span data-stu-id="3faef-126">**AD DS subnet**.</span></span> <span data-ttu-id="3faef-127">AD DS 서버는 방화벽 역할을 하는 NSG(네트워크 보안 그룹) 규칙과 함께 자체의 서브넷에 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-127">The AD DS servers are contained in their own subnet with network security group (NSG) rules acting as a firewall.</span></span>

- <span data-ttu-id="3faef-128">**AD DS 서버**</span><span class="sxs-lookup"><span data-stu-id="3faef-128">**AD DS servers**.</span></span> <span data-ttu-id="3faef-129">Azure에서 VM으로 실행되는 도메인 컨트롤러</span><span class="sxs-lookup"><span data-stu-id="3faef-129">Domain controllers running as VMs in Azure.</span></span> <span data-ttu-id="3faef-130">이러한 서버는 도메인 내에서 로컬 ID의 인증을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-130">These servers provide authentication of local identities within the domain.</span></span>

- <span data-ttu-id="3faef-131">**AD FS 서브넷**</span><span class="sxs-lookup"><span data-stu-id="3faef-131">**AD FS subnet**.</span></span> <span data-ttu-id="3faef-132">AD FS 서버는 방화벽 역할을 하는 NSG 규칙과 함께 자체의 서브넷 내에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-132">The AD FS servers are located within their own subnet with NSG rules acting as a firewall.</span></span>

- <span data-ttu-id="3faef-133">**AD FS 서버**</span><span class="sxs-lookup"><span data-stu-id="3faef-133">**AD FS servers**.</span></span> <span data-ttu-id="3faef-134">AD FS 서버는 페더레이션된 인증 및 권한 부여를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-134">The AD FS servers provide federated authorization and authentication.</span></span> <span data-ttu-id="3faef-135">이 아키텍처에서 다음 작업을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-135">In this architecture, they perform the following tasks:</span></span>

  - <span data-ttu-id="3faef-136">파트너 사용자를 대신하여 파트너 페더레이션 서버에서 만들어진 클레임을 포함하는 보안 토큰 받기</span><span class="sxs-lookup"><span data-stu-id="3faef-136">Receiving security tokens containing claims made by a partner federation server on behalf of a partner user.</span></span> <span data-ttu-id="3faef-137">AD FS는 요청에 권한을 부여하기 위해 Azure에서 실행되는 웹 애플리케이션에 클레임을 전달하기 전에 토큰이 유효한지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-137">AD FS verifies that the tokens are valid before passing the claims to the web application running in Azure to authorize requests.</span></span>

    <span data-ttu-id="3faef-138">Azure에서 실행되는 웹 애플리케이션은 *신뢰 당사자*입니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-138">The web application running in Azure is the *relying party*.</span></span> <span data-ttu-id="3faef-139">파트너 페더레이션 서버는 웹 애플리케이션에서 인식할 수 있는 클레임을 발급해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-139">The partner federation server must issue claims that are understood by the web application.</span></span> <span data-ttu-id="3faef-140">파트너 페더레이션 서버는 파트너 조직에서 인증된 계정을 대신하여 액세스 요청을 제출하기 때문에 *계정 파트너*라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-140">The partner federation servers are referred to as *account partners*, because they submit access requests on behalf of authenticated accounts in the partner organization.</span></span> <span data-ttu-id="3faef-141">AD FS 서버는 리소스(웹 애플리케이션)에 대한 액세스를 제공하기 때문에 *리소스 파트너*라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-141">The AD FS servers are called *resource partners* because they provide access to resources (the web application).</span></span>

  - <span data-ttu-id="3faef-142">AD DS 및 [Active Directory Device Registration Service][ADDRS]를 사용하여 웹 브라우저를 실행하는 외부 사용자 또는 웹 애플리케이션에 대한 액세스가 필요한 장치에서 들어오는 요청 인증 및 권한 부여</span><span class="sxs-lookup"><span data-stu-id="3faef-142">Authenticating and authorizing incoming requests from external users running a web browser or device that needs access to web applications, by using AD DS and the [Active Directory Device Registration Service][ADDRS].</span></span>

  <span data-ttu-id="3faef-143">AD FS 서버는 Azure 부하 분산 장치를 통해 액세스되는 팜으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-143">The AD FS servers are configured as a farm accessed through an Azure load balancer.</span></span> <span data-ttu-id="3faef-144">이 구현은 가용성과 확장성을 향상시킵니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-144">This implementation improves availability and scalability.</span></span> <span data-ttu-id="3faef-145">AD FS 서버는 인터넷에 직접 노출되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-145">The AD FS servers are not exposed directly to the Internet.</span></span> <span data-ttu-id="3faef-146">모든 인터넷 트래픽은 AD FS 웹 애플리케이션 프록시 서버 및 DMZ(경계 네트워크라고도 함)를 통해 필터링됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-146">All Internet traffic is filtered through AD FS web application proxy servers and a DMZ (also referred to as a perimeter network).</span></span>

  <span data-ttu-id="3faef-147">AD FS가 작동하는 방법에 대한 자세한 내용은 [Active Directory Federation Services 개요][active-directory-federation-services-overview]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-147">For more information about how AD FS works, see [Active Directory Federation Services Overview][active-directory-federation-services-overview].</span></span> <span data-ttu-id="3faef-148">또한 [Azure에서 AD FS 배포][adfs-intro] 문서는 구현에 대한 자세한 단계별 소개를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-148">Also, the article [AD FS deployment in Azure][adfs-intro] contains a detailed step-by-step introduction to implementation.</span></span>

- <span data-ttu-id="3faef-149">**AD FS 프록시 서브넷**</span><span class="sxs-lookup"><span data-stu-id="3faef-149">**AD FS proxy subnet**.</span></span> <span data-ttu-id="3faef-150">AD FS 프록시 서버는 보호를 제공하는 NSG 규칙과 함께 자체의 서브넷 내에서 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-150">The AD FS proxy servers can be contained within their own subnet, with NSG rules providing protection.</span></span> <span data-ttu-id="3faef-151">이 서브넷의 서버는 Azure 가상 네트워크와 인터넷 간 방화벽을 제공하는 네트워크 가상 어플라이언스의 집합을 통해 인터넷에 노출됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-151">The servers in this subnet are exposed to the Internet through a set of network virtual appliances that provide a firewall between your Azure virtual network and the Internet.</span></span>

- <span data-ttu-id="3faef-152">**AD FS WAP(웹 응용 프로그램 프록시) 서버**</span><span class="sxs-lookup"><span data-stu-id="3faef-152">**AD FS web application proxy (WAP) servers**.</span></span> <span data-ttu-id="3faef-153">이러한 VM은 파트너 조직 및 외부 장치에서 들어오는 요청에 대한 AD FS 서버로 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-153">These VMs act as AD FS servers for incoming requests from partner organizations and external devices.</span></span> <span data-ttu-id="3faef-154">WAP 서버는 AD FS 서버를 인터넷의 직접 액세스에서 보호하는 필터로 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-154">The WAP servers act as a filter, shielding the AD FS servers from direct access from the Internet.</span></span> <span data-ttu-id="3faef-155">AD FS 서버와 마찬가지로 부하 분산으로 팜에서 WAP 서버를 배포하는 것은 독립 실행형 서버 컬렉션을 배포하는 것보다 더 큰 가용성 및 확장성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-155">As with the AD FS servers, deploying the WAP servers in a farm with load balancing gives you greater availability and scalability than deploying a collection of stand-alone servers.</span></span>

  > [!NOTE]
  > <span data-ttu-id="3faef-156">WAP 서버 설치에 대한 자세한 내용은 [웹 애플리케이션 프록시 서버 설치 및 구성][install_and_configure_the_web_application_proxy_server]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-156">For detailed information about installing WAP servers, see [Install and Configure the Web Application Proxy Server][install_and_configure_the_web_application_proxy_server]</span></span>
  >

- <span data-ttu-id="3faef-157">**파트너 조직**</span><span class="sxs-lookup"><span data-stu-id="3faef-157">**Partner organization**.</span></span> <span data-ttu-id="3faef-158">Azure에서 실행되는 웹 애플리케이션에 대한 액세스를 요청하는 웹 애플리케이션을 실행하는 파트너 조직</span><span class="sxs-lookup"><span data-stu-id="3faef-158">A partner organization running a web application that requests access to a web application running in Azure.</span></span> <span data-ttu-id="3faef-159">파트너 조직의 페더레이션 서버는 요청을 로컬로 인증하고 Azure에서 실행되는 AD FS에 대한 클레임을 포함하는 보안 토큰을 제출합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-159">The federation server at the partner organization authenticates requests locally, and submits security tokens containing claims to AD FS running in Azure.</span></span> <span data-ttu-id="3faef-160">Azure에서 AD FS는 보안 토큰의 유효성을 검사하고 유효한 경우 인증을 위해 Azure에서 실행되는 웹 애플리케이션에 클레임을 전달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-160">AD FS in Azure validates the security tokens, and if valid can pass the claims to the web application running in Azure to authorize them.</span></span>

  > [!NOTE]
  > <span data-ttu-id="3faef-161">또한 Azure 게이트웨이를 사용하여 신뢰할 수 있는 파트너에게 AD FS에 대한 직접 액세스를 제공하도록 VPN 터널을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-161">You can also configure a VPN tunnel using Azure gateway to provide direct access to AD FS for trusted partners.</span></span> <span data-ttu-id="3faef-162">이러한 파트너에서 받은 요청은 WAP 서버를 통해 전달하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="3faef-162">Requests received from these partners do not pass through the WAP servers.</span></span>
  >

<span data-ttu-id="3faef-163">AD FS와 관련되지 않은 아키텍처의 부분에 대한 자세한 내용은 다음을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-163">For more information about the parts of the architecture that are not related to AD FS, see the following:</span></span>

- <span data-ttu-id="3faef-164">[Azure에서 보안 하이브리드 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture]</span><span class="sxs-lookup"><span data-stu-id="3faef-164">[Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture]</span></span>
- <span data-ttu-id="3faef-165">[Azure에서 인터넷 액세스로 보안 하이브리드 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture-with-internet-access]</span><span class="sxs-lookup"><span data-stu-id="3faef-165">[Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]</span></span>
- <span data-ttu-id="3faef-166">[Azure에서 Active Directory ID로 보안 하이브리드 네트워크 아키텍처 구현][extending-ad-to-azure]</span><span class="sxs-lookup"><span data-stu-id="3faef-166">[Implementing a secure hybrid network architecture with Active Directory identities in Azure][extending-ad-to-azure].</span></span>

## <a name="recommendations"></a><span data-ttu-id="3faef-167">권장 사항</span><span class="sxs-lookup"><span data-stu-id="3faef-167">Recommendations</span></span>

<span data-ttu-id="3faef-168">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-168">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="3faef-169">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-169">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vm-recommendations"></a><span data-ttu-id="3faef-170">VM 권장 사항</span><span class="sxs-lookup"><span data-stu-id="3faef-170">VM recommendations</span></span>

<span data-ttu-id="3faef-171">예상되는 트래픽 볼륨을 처리할 충분한 리소스가 있는 VM을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-171">Create VMs with sufficient resources to handle the expected volume of traffic.</span></span> <span data-ttu-id="3faef-172">시작 지점으로 온-프레미스에서 AD FS를 호스팅하는 기존 컴퓨터의 크기를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-172">Use the size of the existing machines hosting AD FS on premises as a starting point.</span></span> <span data-ttu-id="3faef-173">리소스 사용률을 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-173">Monitor the resource utilization.</span></span> <span data-ttu-id="3faef-174">VM의 크기를 조정하고 너무 큰 경우 축소할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-174">You can resize the VMs and scale down if they are too large.</span></span>

<span data-ttu-id="3faef-175">[Azure에서 Windows VM 실행][vm-recommendations]에 나열된 권장 사항을 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-175">Follow the recommendations listed in [Running a Windows VM on Azure][vm-recommendations].</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="3faef-176">네트워킹 권장 사항</span><span class="sxs-lookup"><span data-stu-id="3faef-176">Networking recommendations</span></span>

<span data-ttu-id="3faef-177">정적 개인 IP 주소로 AD FS 및 WAP 서버를 호스팅하는 각 VM에 대한 네트워크 인터페이스를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-177">Configure the network interface for each of the VMs hosting AD FS and WAP servers with static private IP addresses.</span></span>

<span data-ttu-id="3faef-178">AD FS VM에 공용 IP 주소를 제공하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="3faef-178">Do not give the AD FS VMs public IP addresses.</span></span> <span data-ttu-id="3faef-179">자세한 내용은 보안 고려 사항 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-179">For more information, see the Security considerations section.</span></span>

<span data-ttu-id="3faef-180">Active Directory DS VM을 참조하기 위해 각 AD FS 및 WAP VM의 네트워크 인터페이스에 대한 기본 및 보조 DNS(도메인 이름 서비스) 서버의 IP 주소를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-180">Set the IP address of the preferred and secondary domain name service (DNS) servers for the network interfaces for each AD FS and WAP VM to reference the Active Directory DS VMs.</span></span> <span data-ttu-id="3faef-181">Active Directory DS VM은 DNS를 실행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-181">The Active Directory DS VMS should be running DNS.</span></span> <span data-ttu-id="3faef-182">이 단계는 각 VM을 도메인에 조인하기 위해 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-182">This step is necessary to enable each VM to join the domain.</span></span>

### <a name="ad-fs-availability"></a><span data-ttu-id="3faef-183">AD FS 가용성</span><span class="sxs-lookup"><span data-stu-id="3faef-183">AD FS availability</span></span>

<span data-ttu-id="3faef-184">서비스의 가용성 향상을 위해 두 개 이상의 서버와 함께 AD FS 팜을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-184">Create an AD FS farm with at least two servers to increase availability of the service.</span></span> <span data-ttu-id="3faef-185">팜의 각 AD FS VM에 대해 다른 저장소 계정을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-185">Use different storage accounts for each AD FS VM in the farm.</span></span> <span data-ttu-id="3faef-186">이 방법을 사용하면 단일 저장소 계정의 실패가 전체 팜을 액세스할 수 없도록 하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-186">This approach helps to ensure that a failure in a single storage account does not make the entire farm inaccessible.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3faef-187">[관리 디스크](/azure/storage/storage-managed-disks-overview)를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-187">We recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview).</span></span> <span data-ttu-id="3faef-188">관리 디스크는 저장소 계정이 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-188">Managed disks do not require a storage account.</span></span> <span data-ttu-id="3faef-189">디스크의 크기와 유형을 지정하기만 하면 고가용성 방식으로 배포됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-189">You simply specify the size and type of disk and it is deployed in a highly available way.</span></span> <span data-ttu-id="3faef-190">[참조 아키텍처](/azure/architecture/reference-architectures/)는 현재 관리 디스크를 배포하지 않지만 [템플릿 빌딩 블록](https://github.com/mspnp/template-building-blocks/wiki)은 버전 2에서 관리 디스크를 배포하도록 업데이트됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-190">Our [reference architectures](/azure/architecture/reference-architectures/) do not currently deploy managed disks but the [template building blocks](https://github.com/mspnp/template-building-blocks/wiki) will be updated to deploy managed disks in version 2.</span></span>

<span data-ttu-id="3faef-191">AD FS 및 WAP VM에 대한 별도 Azure 가용성 집합을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-191">Create separate Azure availability sets for the AD FS and WAP VMs.</span></span> <span data-ttu-id="3faef-192">각 집합에 두 개 이상의 VM이 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-192">Ensure that there are at least two VMs in each set.</span></span> <span data-ttu-id="3faef-193">각 가용성 집합은 두 개 이상의 업데이트 도메인 및 두 개의 장애 도메인이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-193">Each availability set must have at least two update domains and two fault domains.</span></span>

<span data-ttu-id="3faef-194">다음과 같이 AD FS VM 및 WAP VM에 대한 부하 분산 장치를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-194">Configure the load balancers for the AD FS VMs and WAP VMs as follows:</span></span>

- <span data-ttu-id="3faef-195">WAP VM에 대한 외부 액세스를 제공하는 Azure 부하 분산 장치 및 팜의 AD FS 서버 간에 부하를 분산하는 내부 부하 분산 장치를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-195">Use an Azure load balancer to provide external access to the WAP VMs, and an internal load balancer to distribute the load across the AD FS servers in the farm.</span></span>
- <span data-ttu-id="3faef-196">포트 443(HTTPS)에 표시되는 트래픽을 AD FS/WAP 서버에 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-196">Only pass traffic appearing on port 443 (HTTPS) to the AD FS/WAP servers.</span></span>
- <span data-ttu-id="3faef-197">부하 분산 장치에 고정 IP 주소를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-197">Give the load balancer a static IP address.</span></span>
- <span data-ttu-id="3faef-198">`/adfs/probe`에 대해 HTTP를 사용하여 상태 프로브를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-198">Create a health probe using HTTP against `/adfs/probe`.</span></span> <span data-ttu-id="3faef-199">자세한 내용은 [하드웨어 Load Balancer 상태 검사 및 웹 애플리케이션 프록시 / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/)를 참조합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-199">For more information, see [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).</span></span>

  > [!NOTE]
  > <span data-ttu-id="3faef-200">AD FS 서버는 SNI(서버 이름 표시) 프로토콜을 사용하므로 부하 분산 장치에서 HTTPS 엔드포인트를 사용하는 프로브에 대한 시도는 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-200">AD FS servers use the Server Name Indication (SNI) protocol, so attempting to probe using an HTTPS endpoint from the load balancer fails.</span></span>
  >

- <span data-ttu-id="3faef-201">AD FS 부하 분산 장치에 대한 도메인에 DNS *A* 레코드를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-201">Add a DNS *A* record to the domain for the AD FS load balancer.</span></span> <span data-ttu-id="3faef-202">부하 분산 장치의 IP 주소를 지정하고 도메인의 이름을 지정합니다(예: adfs.contoso.com).</span><span class="sxs-lookup"><span data-stu-id="3faef-202">Specify the IP address of the load balancer, and give it a name in the domain (such as adfs.contoso.com).</span></span> <span data-ttu-id="3faef-203">이는 AD FS 서버 팜에 액세스하는 데 사용하는 이름 클라이언트 및 WAP 서버입니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-203">This is the name clients and the WAP servers use to access the AD FS server farm.</span></span>

### <a name="ad-fs-security"></a><span data-ttu-id="3faef-204">AD FS 보안</span><span class="sxs-lookup"><span data-stu-id="3faef-204">AD FS security</span></span>

<span data-ttu-id="3faef-205">인터넷에 대한 AD FS 서버의 직접 노출을 방지합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-205">Prevent direct exposure of the AD FS servers to the Internet.</span></span> <span data-ttu-id="3faef-206">AD FS 서버는 보안 토큰을 부여하는 완전한 권한이 있는 도메인에 가입된 컴퓨터입니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-206">AD FS servers are domain-joined computers that have full authorization to grant security tokens.</span></span> <span data-ttu-id="3faef-207">서버가 손상되면 악의적인 사용자가 모든 웹 애플리케이션 및 AD FS로 보호되는 모든 페더레이션 서버에 전체 액세스 토큰을 발급할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-207">If a server is compromised, a malicious user can issue full access tokens to all web applications and to all federation servers that are protected by AD FS.</span></span> <span data-ttu-id="3faef-208">시스템이 신뢰할 수 있는 파트너 사이트에서 연결하지 않은 외부 사용자의 요청을 처리해야 하는 경우 WAP 서버를 사용하여 이러한 요청을 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-208">If your system must handle requests from external users not connecting from trusted partner sites, use WAP servers to handle these requests.</span></span> <span data-ttu-id="3faef-209">자세한 내용은 [페더레이션 서버 프록시를 배치할 위치][where-to-place-an-fs-proxy]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-209">For more information, see [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy].</span></span>

<span data-ttu-id="3faef-210">AD FS 서버와 WAP 서버를 자체 방화벽이 있는 별도 서브넷에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-210">Place AD FS servers and WAP servers in separate subnets with their own firewalls.</span></span> <span data-ttu-id="3faef-211">NSG 규칙을 사용하여 방화벽 규칙을 정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-211">You can use NSG rules to define firewall rules.</span></span> <span data-ttu-id="3faef-212">보다 포괄적인 보호가 필요한 경우 [Azure에서 인터넷 액세스로 보안 하이브리드 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture-with-internet-access] 문서에서 설명된 대로 한 쌍의 서브넷 및 NVA(네트워크 가상 어플라이언스)를 사용하여 서버 주변에 추가 보안 경계를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-212">If you require more comprehensive protection you can implement an additional security perimeter around servers by using a pair of subnets and network virtual appliances (NVAs), as described in the document [Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span> <span data-ttu-id="3faef-213">모든 방화벽은 포트 443(HTTPS)의 트래픽을 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-213">All firewalls should allow traffic on port 443 (HTTPS).</span></span>

<span data-ttu-id="3faef-214">AD FS 및 WAP 서버에 대한 직접 로그인 액세스를 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-214">Restrict direct sign in access to the AD FS and WAP servers.</span></span> <span data-ttu-id="3faef-215">DevOps 직원만 연결할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-215">Only DevOps staff should be able to connect.</span></span>

<span data-ttu-id="3faef-216">WAP 서버를 도메인에 조인하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="3faef-216">Do not join the WAP servers to the domain.</span></span>

### <a name="ad-fs-installation"></a><span data-ttu-id="3faef-217">AD FS 설치</span><span class="sxs-lookup"><span data-stu-id="3faef-217">AD FS installation</span></span>

<span data-ttu-id="3faef-218">[페더레이션 서버 팜 배포][Deploying_a_federation_server_farm] 문서는 AD FS 설치 및 구성에 대한 자세한 지침을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-218">The article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] provides detailed instructions for installing and configuring AD FS.</span></span> <span data-ttu-id="3faef-219">팜의 첫 번째 AD FS 서버를 구성하기 전에 다음 작업을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-219">Perform the following tasks before configuring the first AD FS server in the farm:</span></span>

1. <span data-ttu-id="3faef-220">서버 인증을 수행하기 위해 공개적으로 신뢰할 수 있는 인증서를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-220">Obtain a publicly trusted certificate for performing server authentication.</span></span> <span data-ttu-id="3faef-221">*주체 이름*은 페더레이션 서비스에 액세스하는 데 사용하는 이름 클라이언트를 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-221">The *subject name* must contain the name clients use to access the federation service.</span></span> <span data-ttu-id="3faef-222">부하 분산에 대해 등록된 DNS 이름일 수 있습니다(예: *adfs.contoso.com*). (보안상의 이유로 \**.contoso.com*과 같은 와일드카드 이름 사용을 피합니다.)</span><span class="sxs-lookup"><span data-stu-id="3faef-222">This can be the DNS name registered for the load balancer, for example, *adfs.contoso.com* (avoid using wildcard names such as \**.contoso.com*, for security reasons).</span></span> <span data-ttu-id="3faef-223">모든 AD FS 서버 VM에 동일한 인증서를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-223">Use the same certificate on all AD FS server VMs.</span></span> <span data-ttu-id="3faef-224">신뢰할 수 있는 인증 기관에서 인증서를 구입할 수 있지만 조직에서 Active Directory Certificate Services를 사용하는 경우 직접 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-224">You can purchase a certificate from a trusted certification authority, but if your organization uses Active Directory Certificate Services you can create your own.</span></span>

    <span data-ttu-id="3faef-225">*주체 대체 이름*은 외부 장치에서 액세스할 수 있도록 DRS(장치 등록 서비스)에서 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-225">The *subject alternative name* is used by the device registration service (DRS) to enable access from external devices.</span></span> <span data-ttu-id="3faef-226">*enterpriseregistration.contoso.com* 형식이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-226">This should be of the form *enterpriseregistration.contoso.com*.</span></span>

    <span data-ttu-id="3faef-227">자세한 내용은 [AD FS에 대한 SSL(Secure Sockets Layer) 인증서 가져오기 및 구성][adfs_certificates]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-227">For more information, see [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates].</span></span>

2. <span data-ttu-id="3faef-228">도메인 컨트롤러에서 키 배포 서비스에 대한 새 루트 키를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-228">On the domain controller, generate a new root key for the Key Distribution Service.</span></span> <span data-ttu-id="3faef-229">10시간을 뺀 현재 시간으로 유효한 시간을 설정합니다.(이 구성은 도메인에서 키 배포 및 동기화에서 발생할 수 있는 지연을 줄입니다.)</span><span class="sxs-lookup"><span data-stu-id="3faef-229">Set the effective time to the current time minus 10 hours (this configuration reduces the delay that can occur in distributing and synchronizing keys across the domain).</span></span> <span data-ttu-id="3faef-230">이 단계는 AD FS 서비스를 실행하는 데 사용되는 그룹 서비스 계정 만들기를 지원하는 데 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-230">This step is necessary to support creating the group service account that is used to run the AD FS service.</span></span> <span data-ttu-id="3faef-231">다음 PowerShell 명령은 이 작업을 수행하는 방법의 예를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-231">The following PowerShell command shows an example of how to do this:</span></span>

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. <span data-ttu-id="3faef-232">각 AD FS 서버 VM을 도메인에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-232">Add each AD FS server VM to the domain.</span></span>

> [!NOTE]
> <span data-ttu-id="3faef-233">AD FS를 설치하려면 도메인에 대한 PDC(주 도메인 컨트롤러) 에뮬레이터 FSMO(유연한 단일 마스터 작업) 역할을 실행하는 도메인 컨트롤러는 AD FS VM에서 실행되고 액세스할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-233">To install AD FS, the domain controller running the primary domain controller (PDC) emulator flexible single master operation (FSMO) role for the domain must be running and accessible from the AD FS VMs.</span></span> <span data-ttu-id="3faef-234"><<RBC: 덜 반복적으로 만드는 방법이 있나요?>></span><span class="sxs-lookup"><span data-stu-id="3faef-234"><<RBC: Is there a way to make this less repetitive?>></span></span>
>

### <a name="ad-fs-trust"></a><span data-ttu-id="3faef-235">AD FS 신뢰</span><span class="sxs-lookup"><span data-stu-id="3faef-235">AD FS trust</span></span>

<span data-ttu-id="3faef-236">AD FS 설치 및 모든 파트너 조직의 페더레이션 서버 간에 페더레이션 트러스트를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-236">Establish federation trust between your AD FS installation, and the federation servers of any partner organizations.</span></span> <span data-ttu-id="3faef-237">필요한 클레임 필터링 및 매핑을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-237">Configure any claims filtering and mapping required.</span></span>

- <span data-ttu-id="3faef-238">각 파트너 조직의 DevOps 직원은 AD FS 서버를 통해 액세스할 수 있는 웹 애플리케이션에 대한 신뢰 당사자 트러스트를 추가해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-238">DevOps staff at each partner organization must add a relying party trust for the web applications accessible through your AD FS servers.</span></span>
- <span data-ttu-id="3faef-239">조직의 DevOps 직원은 AD FS 서버가 파트너 조직에서 제공하는 클레임을 신뢰할 수 있도록 클레임 공급자 트러스트를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-239">DevOps staff in your organization must configure claims-provider trust to enable your AD FS servers to trust the claims that partner organizations provide.</span></span>
- <span data-ttu-id="3faef-240">또한 조직의 DevOps 직원은 조직의 웹 애플리케이션에 클레임을 전달하도록 AD FS를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-240">DevOps staff in your organization must also configure AD FS to pass claims on to your organization's web applications.</span></span>

<span data-ttu-id="3faef-241">자세한 내용은 [페더레이션 트러스트 설정][establishing-federation-trust]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-241">For more information, see [Establishing Federation Trust][establishing-federation-trust].</span></span>

<span data-ttu-id="3faef-242">조직의 웹 애플리케이션을 게시하고 WAP 서버를 통해 사전 인증을 사용하여 외부 파트너에 사용할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-242">Publish your organization's web applications and make them available to external partners by using preauthentication through the WAP servers.</span></span> <span data-ttu-id="3faef-243">자세한 내용은 [AD FS 사전 인증을 사용하여 애플리케이션 게시][publish_applications_using_AD_FS_preauthentication]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-243">For more information, see [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication]</span></span>

<span data-ttu-id="3faef-244">AD FS는 토큰 변환 및 확대를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-244">AD FS supports token transformation and augmentation.</span></span> <span data-ttu-id="3faef-245">Azure Active Directory는 이 기능을 제공하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-245">Azure Active Directory does not provide this feature.</span></span> <span data-ttu-id="3faef-246">AD FS를 사용하여 트러스트 관계를 설정할 때 다음을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-246">With AD FS, when you set up the trust relationships, you can:</span></span>

- <span data-ttu-id="3faef-247">권한 부여 규칙에 대한 클레임 변환을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-247">Configure claim transformations for authorization rules.</span></span> <span data-ttu-id="3faef-248">예를 들어 비 Microsoft 파트너 조직에서 사용되는 표현에서 조직에서 해당 Active Directory DS가 조직에서 권한을 부여할 수 있는 것으로 그룹 보안을 매핑할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-248">For example, you can map group security from a representation used by a non-Microsoft partner organization to something that that Active Directory DS can authorize in your organization.</span></span>
- <span data-ttu-id="3faef-249">클레임을 한 형식에서 다른 형식으로 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-249">Transform claims from one format to another.</span></span> <span data-ttu-id="3faef-250">예를 들어 애플리케이션이 SAML 1.1 클레임만을 지원하는 경우 SAML 2.0에서 SAML 1.1로 매핑할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-250">For example, you can map from SAML 2.0 to SAML 1.1 if your application only supports SAML 1.1 claims.</span></span>

### <a name="ad-fs-monitoring"></a><span data-ttu-id="3faef-251">AD FS 모니터링</span><span class="sxs-lookup"><span data-stu-id="3faef-251">AD FS monitoring</span></span>

<span data-ttu-id="3faef-252">[Active Directory Federation Services 2012 R2용 Microsoft System Center 관리 팩][oms-adfs-pack]은 페더레이션 서버에 대한 AD FS 배포의 사전 예방적이며 반응적인 모니터링을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-252">The [Microsoft System Center Management Pack for Active Directory Federation Services 2012 R2][oms-adfs-pack] provides both proactive and reactive monitoring of your AD FS deployment for the federation server.</span></span> <span data-ttu-id="3faef-253">이 관리 팩은 다음을 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-253">This management pack monitors:</span></span>

- <span data-ttu-id="3faef-254">AD FS 서비스가 해당 이벤트 로그에 기록하는 이벤트</span><span class="sxs-lookup"><span data-stu-id="3faef-254">Events that the AD FS service records in its event logs.</span></span>
- <span data-ttu-id="3faef-255">AD FS 성능 카운터가 수집하는 성능 데이터</span><span class="sxs-lookup"><span data-stu-id="3faef-255">The performance data that the AD FS performance counters collect.</span></span>
- <span data-ttu-id="3faef-256">AD FS 시스템 및 웹 애플리케이션(신뢰 당사자)의 전반적인 상태, 중요한 문제 및 경고에 대한 경고를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-256">The overall health of the AD FS system and web applications (relying parties), and provides alerts for critical issues and warnings.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="3faef-257">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="3faef-257">Scalability considerations</span></span>

<span data-ttu-id="3faef-258">[AD FS 배포 계획][plan-your-adfs-deployment] 문서에서 요약된 다음 고려 사항은 AD FS 팜 크기 조정에 대한 시작 지점을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-258">The following considerations, summarized from the article [Plan your AD FS deployment][plan-your-adfs-deployment], give a starting point for sizing AD FS farms:</span></span>

- <span data-ttu-id="3faef-259">1000명의 사용자보다 적은 경우 전용 서버를 만들지 마십시오. 대신 클라우드의 각 Active Directory DS 서버에 AD FS를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-259">If you have fewer than 1000 users, do not create dedicated servers, but instead install AD FS on each of the Active Directory DS servers in the cloud.</span></span> <span data-ttu-id="3faef-260">가용성을 유지하기 위해 두 개 이상의 Active Directory DS 서버가 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-260">Make sure that you have at least two Active Directory DS servers to maintain availability.</span></span> <span data-ttu-id="3faef-261">단일 WAP 서버를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-261">Create a single WAP server.</span></span>
- <span data-ttu-id="3faef-262">1000명에서 15000명 사이의 사용자가 있는 경우 두 개의 전용 AD FS 서버 및 두 개의 전용 WAP 서버를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-262">If you have between 1000 and 15000 users, create two dedicated AD FS servers and two dedicated WAP servers.</span></span>
- <span data-ttu-id="3faef-263">15000명에서 60000명 사이의 사용자가 있는 경우 세 개에서 다섯 개 사이의 전용 AD FS 서버 및 두 개 이상의 전용 WAP 서버를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-263">If you have between 15000 and 60000 users, create between three and five dedicated AD FS servers and at least two dedicated WAP servers.</span></span>

<span data-ttu-id="3faef-264">이러한 고려 사항은 Azure에서 듀얼 쿼드 코어 VM(표준 D4_v2 또는 그 이상) 크기를 사용하고 있다고 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-264">These considerations assume that you are using dual quad-core VM (Standard D4_v2, or better) sizes in Azure.</span></span>

<span data-ttu-id="3faef-265">AD FS 구성 데이터를 저장하는 데 Windows 내부 데이터베이스를 사용하는 경우 팜에서 8개의 AD FS 서버로 제한됩니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-265">If you are using the Windows Internal Database to store AD FS configuration data, you are limited to eight AD FS servers in the farm.</span></span> <span data-ttu-id="3faef-266">나중에 더 필요할 것이라고 예상하는 경우 SQL Server를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-266">If you anticipate that you will need more in the future, use SQL Server.</span></span> <span data-ttu-id="3faef-267">자세한 내용은 [AD FS 구성 데이터베이스의 역할][adfs-configuration-database]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="3faef-267">For more information, see [The Role of the AD FS Configuration Database][adfs-configuration-database].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="3faef-268">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="3faef-268">Availability considerations</span></span>

<span data-ttu-id="3faef-269">SQL Server 또는 Windows 내부 데이터베이스를 사용하여 AD FS 구성 정보를 보관할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-269">You can use either SQL Server or the Windows Internal Database to hold AD FS configuration information.</span></span> <span data-ttu-id="3faef-270">Windows 내부 데이터베이스는 기본 중복성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-270">The Windows Internal Database provides basic redundancy.</span></span> <span data-ttu-id="3faef-271">변경 내용은 AD FS 클러스터의 AD FS 데이터베이스 중 하나에 직접 기록되는 반면 다른 서버는 끌어오기 복제를 사용하여 해당 데이터베이스를 최신 상태로 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-271">Changes are written directly to only one of the AD FS databases in the AD FS cluster, while the other servers use pull replication to keep their databases up to date.</span></span> <span data-ttu-id="3faef-272">SQL Server를 사용하면 장애 조치(failover) 클러스터링 또는 미러링을 사용하여 전체 데이터베이스 중복성 및 고가용성을 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-272">Using SQL Server can provide full database redundancy and high availability using failover clustering or mirroring.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="3faef-273">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="3faef-273">Manageability considerations</span></span>

<span data-ttu-id="3faef-274">DevOps 직원은 다음 작업을 수행할 준비가 되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-274">DevOps staff should be prepared to perform the following tasks:</span></span>

- <span data-ttu-id="3faef-275">AD FS 팜 관리, 페더레이션 서버의 트러스트 정책 관리 및 페더레이션 서비스에서 사용되는 인증서 관리를 포함하는 페더레이션 서버 관리</span><span class="sxs-lookup"><span data-stu-id="3faef-275">Managing the federation servers, including managing the AD FS farm, managing trust policy on the federation servers, and managing the certificates used by the federation services.</span></span>
- <span data-ttu-id="3faef-276">WAP 팜 및 인증서 관리를 포함하는 WAP 서버 관리</span><span class="sxs-lookup"><span data-stu-id="3faef-276">Managing the WAP servers including managing the WAP farm and certificates.</span></span>
- <span data-ttu-id="3faef-277">신뢰 당사자, 인증 방법 및 클레임 매핑 구성을 포함하는 웹 애플리케이션 관리</span><span class="sxs-lookup"><span data-stu-id="3faef-277">Managing web applications including configuring relying parties, authentication methods, and claims mappings.</span></span>
- <span data-ttu-id="3faef-278">AD FS 구성 요소 백업</span><span class="sxs-lookup"><span data-stu-id="3faef-278">Backing up AD FS components.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="3faef-279">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="3faef-279">Security considerations</span></span>

<span data-ttu-id="3faef-280">AD FS는 HTTPS 프로토콜을 사용하므로 웹 계층 VM을 포함하는 서브넷에 대한 NSG 규칙이 HTTPS 요청을 허용하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-280">AD FS utilizes the HTTPS protocol, so make sure that the NSG rules for the subnet containing the web tier VMs permit HTTPS requests.</span></span> <span data-ttu-id="3faef-281">이러한 요청은 온-프레미스 네트워크, 웹 계층, 비즈니스 계층, 데이터 계층, 개인 DMZ, 공용 DMZ를 포함하는 서브넷 및 AD FS 서버를 포함하는 서브넷에서 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-281">These requests can originate from the on-premises network, the subnets containing the web tier, business tier, data tier, private DMZ, public DMZ, and the subnet containing the AD FS servers.</span></span>

<span data-ttu-id="3faef-282">감사를 목적으로 가상 네트워크의 에지를 탐색하는 트래픽의 자세한 정보를 기록하는 네트워크 가상 어플라이언스의 집합을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-282">Consider using a set of network virtual appliances that logs detailed information on traffic traversing the edge of your virtual network for auditing purposes.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="3faef-283">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="3faef-283">Deploy the solution</span></span>

<span data-ttu-id="3faef-284">[GitHub][github]에서 이 참조 아키텍처를 배포할 수 있는 솔루션을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-284">A solution is available on [GitHub][github] to deploy this reference architecture.</span></span> <span data-ttu-id="3faef-285">솔루션을 배포하는 PowerShell 스크립트를 실행하려면 최신 버전의 [Azure CLI][azure-cli]가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-285">You will need the latest version of the [Azure CLI][azure-cli] to run the Powershell script that deploys the solution.</span></span> <span data-ttu-id="3faef-286">이 참조 아키텍처를 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-286">To deploy the reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="3faef-287">[GitHub][github]의 해당 솔루션 폴더를 로컬 컴퓨터로 다운로드하거나 복제합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-287">Download or clone the solution folder from [GitHub][github] to your local machine.</span></span>

2. <span data-ttu-id="3faef-288">Azure CLI를 열고 로컬 솔루션 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-288">Open the Azure CLI and navigate to the local solution folder.</span></span>

3. <span data-ttu-id="3faef-289">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="3faef-289">Run the following command:</span></span>

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```

    <span data-ttu-id="3faef-290">`<subscription id>` 를 Azure 구독 ID로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-290">Replace `<subscription id>` with your Azure subscription ID.</span></span>

    <span data-ttu-id="3faef-291">`<location>`에서 Azure 지역(예: `eastus` 또는 `westus`)을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-291">For `<location>`, specify an Azure region, such as `eastus` or `westus`.</span></span>

    <span data-ttu-id="3faef-292">`<mode>` 매개 변수는 배포의 세분성을 제어합니다. 매개 변수는 다음 값 중 하나일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-292">The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:</span></span>

   - <span data-ttu-id="3faef-293">`Onpremise`: 시뮬레이트된 온-프레미스 환경을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-293">`Onpremise`: Deploys a simulated on-premises environment.</span></span> <span data-ttu-id="3faef-294">기존 온-프레미스 네트워크가 없거나 기존 온-프레미스 네트워크의 구성을 변경하지 않고 이 참조 아키텍처를 테스트하려는 경우 이 배포를 사용하여 테스트 및 실험할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-294">You can use this deployment to test and experiment if you do not have an existing on-premises network, or if you want to test this reference architecture without changing the configuration of your existing on-premises network.</span></span>
   - <span data-ttu-id="3faef-295">`Infrastructure`: VNet 인프라 및 점프 상자를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-295">`Infrastructure`: deploys the VNet infrastructure and jump box.</span></span>
   - <span data-ttu-id="3faef-296">`CreateVpn`: Azure 가상 네트워크 게이트웨이를 배포하고 시뮬레이트된 온-프레미스 네트워크에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-296">`CreateVpn`: deploys an Azure virtual network gateway and connects it to the simulated on-premises network.</span></span>
   - <span data-ttu-id="3faef-297">`AzureADDS`: Active Directory DS 서버 역할을 하는 VM을 배포하고, 이러한 VM에 Active Directory를 배포하고, Azure에서 도메인을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-297">`AzureADDS`: deploys the VMs acting as Active Directory DS servers, deploys Active Directory to these VMs, and creates the domain in Azure.</span></span>
   - <span data-ttu-id="3faef-298">`AdfsVm`: AD FS VM을 배포하고 Azure에서 도메인에 조인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-298">`AdfsVm`: deploys the AD FS VMs and joins them to the domain in Azure.</span></span>
   - <span data-ttu-id="3faef-299">`PublicDMZ`: Azure에서 공용 DMZ를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-299">`PublicDMZ`: deploys the public DMZ in Azure.</span></span>
   - <span data-ttu-id="3faef-300">`ProxyVm`: AD FS 프록시 VM을 배포하고 Azure에서 도메인에 조인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-300">`ProxyVm`: deploys the AD FS proxy VMs and joins them to the domain in Azure.</span></span>
   - <span data-ttu-id="3faef-301">`Prepare`: 모든 이전 배포를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-301">`Prepare`: deploys all of the preceding deployments.</span></span> <span data-ttu-id="3faef-302">**완전히 새로운 배포를 작성하고 기존 온-프레미스 인프라가 없는 경우 권장되는 옵션입니다.**</span><span class="sxs-lookup"><span data-stu-id="3faef-302">**This is the recommended option if you are building an entirely new deployment and you don't have an existing on-premises infrastructure.**</span></span>
   - <span data-ttu-id="3faef-303">`Workload`: 필요에 따라 웹, 비즈니스 및 데이터 계층 VM 및 지원하는 네트워크를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-303">`Workload`: optionally deploys web, business, and data tier VMs and supporting network.</span></span> <span data-ttu-id="3faef-304">`Prepare` 배포 모드에 포함되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-304">Not included in the `Prepare` deployment mode.</span></span>
   - <span data-ttu-id="3faef-305">`PrivateDMZ`: 필요에 따라 위에 배포된 `Workload` VM의 앞에 Azure의 개인 DMZ를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-305">`PrivateDMZ`: optionally deploys the private DMZ in Azure in front of the `Workload` VMs deployed above.</span></span> <span data-ttu-id="3faef-306">`Prepare` 배포 모드에 포함되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-306">Not included in the `Prepare` deployment mode.</span></span>

4. <span data-ttu-id="3faef-307">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-307">Wait for the deployment to complete.</span></span> <span data-ttu-id="3faef-308">`Prepare` 옵션을 사용한 경우 배포를 완료하는 데 여러 시간이 소요되며 `Preparation is completed. Please install certificate to all AD FS and proxy VMs.` 메시지와 함께 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-308">If you used the `Prepare` option, the deployment takes several hours to complete, and finishes with the message `Preparation is completed. Please install certificate to all AD FS and proxy VMs.`</span></span>

5. <span data-ttu-id="3faef-309">점프 상자를 다시 시작하여(*ra-adfs-security-rg* 그룹에서 *ra-adfs-mgmt-vm1*) 해당 DNS 설정을 적용하도록 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-309">Restart the jump box (*ra-adfs-mgmt-vm1* in the *ra-adfs-security-rg* group) to allow its DNS settings to take effect.</span></span>

6. <span data-ttu-id="3faef-310">[AD FS용 SSL 인증서를 가져오고][adfs_certificates] AD FS VM에 이 인증서를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-310">[Obtain an SSL Certificate for AD FS][adfs_certificates] and install this certificate on the AD FS VMs.</span></span> <span data-ttu-id="3faef-311">점프 상자를 통해 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-311">Note that you can connect to them through the jump box.</span></span> <span data-ttu-id="3faef-312">IP 주소는 **10.0.5.4** 및 **10.0.5.5**입니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-312">The IP addresses are **10.0.5.4** and **10.0.5.5**.</span></span> <span data-ttu-id="3faef-313">기본 사용자 이름은 **AweSome@PW** 암호가 있는 **contoso\testuser**입니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-313">The default username is **contoso\testuser** with password **AweSome@PW**.</span></span>

   > [!NOTE]
   > <span data-ttu-id="3faef-314">이 때 Deploy-ReferenceArchitecture.ps1 스크립트에 있는 설명은 `makecert` 명령을 사용하여 자체 서명된 테스트 인증서 및 권한을 만들기 위한 자세한 지침을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-314">The comments in the Deploy-ReferenceArchitecture.ps1 script at this point provides detailed instructions for creating a self-signed test certificate and authority using the `makecert` command.</span></span> <span data-ttu-id="3faef-315">그러나 이러한 단계를 **테스트**로만 수행하고 프로덕션 환경에서 makecert에 의해 생성된 인증서를 사용하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="3faef-315">However, perform these steps as a **test** only and do not use the certificates generated by makecert in a production environment.</span></span>

7. <span data-ttu-id="3faef-316">다음 PowerShell 명령을 실행하여 AD FS 서버 팜을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-316">Run the following PowerShell command to deploy the AD FS server farm:</span></span>

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ```

8. <span data-ttu-id="3faef-317">점프 상자에서 `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`으로 이동하여 AD FS 설치를 테스트합니다(이 테스트를 무시할 수 있다고 경고하는 인증서를 받을 수 있음).</span><span class="sxs-lookup"><span data-stu-id="3faef-317">On the jump box, browse to `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` to test the AD FS installation (you may receive a certificate warning that you can ignore for this test).</span></span> <span data-ttu-id="3faef-318">Contoso Corporation 로그인 페이지가 표시되는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-318">Verify that the Contoso Corporation sign-in page appears.</span></span> <span data-ttu-id="3faef-319">암호 **AweS0me@PW**를 사용하여 **contoso\testuser**로 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-319">Sign in as **contoso\testuser** with password **AweS0me@PW**.</span></span>

9. <span data-ttu-id="3faef-320">AD FS 프록시 VM에 SSL 인증서를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-320">Install the SSL certificate on the AD FS proxy VMs.</span></span> <span data-ttu-id="3faef-321">IP 주소는 *10.0.6.4* 및 *10.0.6.5*입니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-321">The IP addresses are *10.0.6.4* and *10.0.6.5*.</span></span>

10. <span data-ttu-id="3faef-322">다음 PowerShell 명령을 실행하여 첫 번째 AD FS 프록시 서버를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-322">Run the following PowerShell command to deploy the first AD FS proxy server:</span></span>

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. <span data-ttu-id="3faef-323">스크립트에 표시된 지침을 따라 첫 번째 프록시 서버의 설치를 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-323">Follow the instructions displayed by the script to test the installation of the first proxy server.</span></span>

12. <span data-ttu-id="3faef-324">다음 PowerShell 명령을 실행하여 두 번째 AD FS 프록시 서버를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-324">Run the following PowerShell command to deploy the second proxy server:</span></span>

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. <span data-ttu-id="3faef-325">스크립트에 표시된 지침을 따라 완전한 프록시 구성을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-325">Follow the instructions displayed by the script to test the complete proxy configuration.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3faef-326">다음 단계</span><span class="sxs-lookup"><span data-stu-id="3faef-326">Next steps</span></span>

- <span data-ttu-id="3faef-327">[Azure Active Directory][aad]에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-327">Learn about [Azure Active Directory][aad].</span></span>
- <span data-ttu-id="3faef-328">[Azure Active Directory B2C][aadb2c]에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="3faef-328">Learn about [Azure Active Directory B2C][aadb2c].</span></span>

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
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
