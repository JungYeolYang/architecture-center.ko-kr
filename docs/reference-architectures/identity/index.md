---
title: Azure와 온-프레미스 Active Directory 통합
titleSuffix: Azure Reference Architectures
description: 온-프레미스 Active Directory를 Azure와 통합하기 위한 참조 아키텍처를 비교합니다.
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 706cb63a65ce521e72ebc41a997dc900afacaab9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343971"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="88cb8-103">온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택</span><span class="sxs-lookup"><span data-stu-id="88cb8-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="88cb8-104">이 문서에서는 온-프레미스 AD(Active Directory) 환경을 Azure 네트워크와 통합하는 옵션을 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="88cb8-105">각 옵션에서 자세한 참조 아키텍처를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-105">For each option, a more detailed reference architecture is available.</span></span>

<span data-ttu-id="88cb8-106">많은 조직에서는 AD DS(Active Directory Domain Services)를 사용하여 보안 경계에 포함된 사용자, 컴퓨터, 응용 프로그램 또는 기타 리소스와 연결된 ID를 인증합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-106">Many organizations use Active Directory Domain Services (AD DS) to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="88cb8-107">디렉터리 및 ID 서비스는 일반적으로 온-프레미스에서 호스트되지만 응용 프로그램이 일부는 온-프레미스에서, 일부는 Azure에서 호스트되는 경우 Azure의 인증 요청을 온-프레미스로 되돌려 보내는 데 시간이 지연될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="88cb8-108">Azure에 디렉터리와 ID 서비스를 구현하면 이러한 대기 시간을 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="88cb8-109">Azure에서는 Azure에 디렉터리와 ID 서비스를 구현하기 위한 두 가지 솔루션을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span>

- <span data-ttu-id="88cb8-110">[Azure AD][azure-active-directory]를 사용하여 클라우드에서 Active Directory 도메인을 만들고 이를 온-프레미스 Active Directory 도메인에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="88cb8-111">[Azure AD Connect][azure-ad-connect]에서 온-프레미스 디렉터리와 Azure AD를 통합합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

- <span data-ttu-id="88cb8-112">AD DS를 도메인 컨트롤러로 실행하는 Azure에 VM을 배포하여 기존 온-프레미스 Active Directory 인프라를 Azure로 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="88cb8-113">이 아키텍처는 온-프레미스 네트워크와 Azure VNet(Virtual Network)이 VPN 또는 ExpressRoute 연결을 통해 연결되는 경우에 더 일반적입니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="88cb8-114">이 아키텍처는 다음과 같은 몇 가지 변형이 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-114">Several variations of this architecture are possible:</span></span>

  - <span data-ttu-id="88cb8-115">Azure에서 도메인을 만들어 온-프레미스 AD 포리스트에 조인합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
  - <span data-ttu-id="88cb8-116">온-프레미스 포리스트의 도메인이 신뢰하는 별도의 포리스트를 Azure에 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
  - <span data-ttu-id="88cb8-117">AD FS(Active Directory Federation Services) 배포를 Azure에 복제합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span>

<span data-ttu-id="88cb8-118">다음 섹션에서는 이러한 각 옵션에 대해 자세히 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="88cb8-119">온-프레미스 도메인을 Azure AD와 통합</span><span class="sxs-lookup"><span data-stu-id="88cb8-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="88cb8-120">Azure AD(Azure Active Directory)를 사용하여 Azure에서 도메인을 만들어 온-프레미스 AD 도메인에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span>

<span data-ttu-id="88cb8-121">Azure AD 디렉터리는 온-프레미스 디렉터리의 확장이 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="88cb8-122">오히려 동일한 개체와 ID를 포함하는 복사본입니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="88cb8-123">이러한 온-프레미스 항목에 대한 변경 사항은 Azure AD로 복사되지만 Azure AD의 변경 사항은 온-프레미스 도메인으로 다시 복제되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="88cb8-124">또한 온-프레미스 디렉터리를 사용하지 않고 Azure AD를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="88cb8-125">이러한 경우 Azure AD는 온-프레미스 디렉터리에서 복제된 데이터를 포함하지 않고 모든 ID 정보의 기본 출처 역할을 합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>

<span data-ttu-id="88cb8-126">**이점**</span><span class="sxs-lookup"><span data-stu-id="88cb8-126">**Benefits**</span></span>

- <span data-ttu-id="88cb8-127">클라우드에서 AD 인프라를 유지할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="88cb8-128">Azure AD가 Microsoft에서 완벽하게 관리 및 유지 관리될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
- <span data-ttu-id="88cb8-129">Azure AD는 온-프레미스에서 사용할 수 있는 것과 동일한 ID 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-129">Azure AD provides the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="88cb8-130">Azure에서 인증이 발생할 수 있으므로 외부 애플리케이션과 사용자가 온-프레미스 도메인에 연결할 필요성이 줄어듭니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="88cb8-131">**과제**</span><span class="sxs-lookup"><span data-stu-id="88cb8-131">**Challenges**</span></span>

- <span data-ttu-id="88cb8-132">ID 서비스가 사용자 및 그룹으로 제한됩니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="88cb8-133">서비스 및 컴퓨터 계정을 인증할 수 있는 기능이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-133">There is no ability to authenticate service and computer accounts.</span></span>
- <span data-ttu-id="88cb8-134">Azure AD 디렉터리를 동기화 상태로 유지하려면 온-프레미스 도메인과의 연결을 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span>
- <span data-ttu-id="88cb8-135">Azure AD를 통해 인증하려면 응용 프로그램을 다시 작성해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="88cb8-136">**참조 아키텍처**</span><span class="sxs-lookup"><span data-stu-id="88cb8-136">**Reference architecture**</span></span>

- <span data-ttu-id="88cb8-137">[Azure Active Directory와 온-프레미스 Active Directory 도메인 통합][aad]</span><span class="sxs-lookup"><span data-stu-id="88cb8-137">[Integrate on-premises Active Directory domains with Azure Active Directory][aad]</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="88cb8-138">온-프레미스 포리스트에 조인된 Azure의 AD DS</span><span class="sxs-lookup"><span data-stu-id="88cb8-138">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="88cb8-139">AD DS(AD Domain Services) 서버를 Azure에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-139">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="88cb8-140">Azure에서 도메인을 만들어 온-프레미스 AD 포리스트에 조인합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-140">Create a domain in Azure and join it to your on-premises AD forest.</span></span>

<span data-ttu-id="88cb8-141">현재 Azure AD에서 구현되지 않은 AD DS 기능을 사용해야 하는 경우 이 옵션을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-141">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span>

<span data-ttu-id="88cb8-142">**이점**</span><span class="sxs-lookup"><span data-stu-id="88cb8-142">**Benefits**</span></span>

- <span data-ttu-id="88cb8-143">온-프레미스에서 사용할 수 있는 동일한 ID 정보에 대한 액세스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-143">Provides access to the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="88cb8-144">온-프레미스 및 Azure에서 사용자, 서비스 및 컴퓨터 계정을 인증할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-144">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
- <span data-ttu-id="88cb8-145">별도의 AD 포리스트를 관리할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-145">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="88cb8-146">Azure의 도메인은 온-프레미스 포리스트에 속할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-146">The domain in Azure can belong to the on-premises forest.</span></span>
- <span data-ttu-id="88cb8-147">온-프레미스 그룹 정책 개체에 의해 정의된 그룹 정책을 Azure의 도메인에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-147">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="88cb8-148">**과제**</span><span class="sxs-lookup"><span data-stu-id="88cb8-148">**Challenges**</span></span>

- <span data-ttu-id="88cb8-149">클라우드에서 고유한 AD DS 서버 및 도메인을 배포하고 관리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-149">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
- <span data-ttu-id="88cb8-150">클라우드의 도메인 서버와 온-프레미스에서 실행되는 서버 간에 동기화 대기 시간이 어느 정도 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-150">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="88cb8-151">**참조 아키텍처**</span><span class="sxs-lookup"><span data-stu-id="88cb8-151">**Reference architecture**</span></span>

- <span data-ttu-id="88cb8-152">[AD DS(Active Directory Domain Services)를 Azure로 확장][ad-ds]</span><span class="sxs-lookup"><span data-stu-id="88cb8-152">[Extend Active Directory Domain Services (AD DS) to Azure][ad-ds]</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="88cb8-153">별도의 포리스트가 있는 Azure의 AD DS</span><span class="sxs-lookup"><span data-stu-id="88cb8-153">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="88cb8-154">AD DS(AD Domain Services) 서버를 Azure에 배포하지만 온-프레미스 포리스트와 분리된 별도의 Active Directory [포리스트][ad-forest-defn]를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-154">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="88cb8-155">이 포리스트는 온-프레미스 포리스트의 도메인에서 신뢰할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-155">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="88cb8-156">이 아키텍처의 일반적인 용도는 클라우드에 저장된 개체 및 ID에 대한 보안 분리를 유지하는 것과 개별 도메인을 온-프레미스에서 클라우드로 마이그레이션하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-156">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="88cb8-157">**이점**</span><span class="sxs-lookup"><span data-stu-id="88cb8-157">**Benefits**</span></span>

- <span data-ttu-id="88cb8-158">온-프레미스 ID와 별도의 Azure 전용 ID를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-158">You can implement on-premises identities and separate Azure-only identities.</span></span>
- <span data-ttu-id="88cb8-159">온-프레미스 AD 포리스트에서 Azure로 복제할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-159">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="88cb8-160">**과제**</span><span class="sxs-lookup"><span data-stu-id="88cb8-160">**Challenges**</span></span>

- <span data-ttu-id="88cb8-161">온-프레미스 ID를 Azure 내에서 인증받으려면 온-프레미스 AD 서버에 추가 네트워크 홉이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-161">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
- <span data-ttu-id="88cb8-162">클라우드에서 고유한 AD DS 서버 및 포리스트를 배포하고 포리스트 간에 적절한 신뢰 관계를 설정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-162">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="88cb8-163">**참조 아키텍처**</span><span class="sxs-lookup"><span data-stu-id="88cb8-163">**Reference architecture**</span></span>

- <span data-ttu-id="88cb8-164">[Azure에서 AD DS(Active Directory Domain Services) 리소스 포리스트 만들기][ad-ds-forest]</span><span class="sxs-lookup"><span data-stu-id="88cb8-164">[Create an Active Directory Domain Services (AD DS) resource forest in Azure][ad-ds-forest]</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="88cb8-165">AD FS를 Azure로 확장</span><span class="sxs-lookup"><span data-stu-id="88cb8-165">Extend AD FS to Azure</span></span>

<span data-ttu-id="88cb8-166">AD FS(Active Directory Federation Services) 배포를 Azure에 복제하여 Azure에서 실행되는 구성 요소에 대해 페더레이션 인증 및 권한 부여를 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-166">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span>

<span data-ttu-id="88cb8-167">이 아키텍처의 일반적인 용도는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-167">Typical uses for this architecture:</span></span>

- <span data-ttu-id="88cb8-168">파트너 조직에서 사용자 인증 및 권한 부여</span><span class="sxs-lookup"><span data-stu-id="88cb8-168">Authenticate and authorize users from partner organizations.</span></span>
- <span data-ttu-id="88cb8-169">사용자가 조직 방화벽 외부에서 실행되는 웹 브라우저에서 인증할 수 있도록 허용</span><span class="sxs-lookup"><span data-stu-id="88cb8-169">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="88cb8-170">사용자가 모바일 디바이스와 같은 승인된 외부 디바이스에서 연결할 수 있도록 허용</span><span class="sxs-lookup"><span data-stu-id="88cb8-170">Allow users to connect from authorized external devices such as mobile devices.</span></span>

<span data-ttu-id="88cb8-171">**이점**</span><span class="sxs-lookup"><span data-stu-id="88cb8-171">**Benefits**</span></span>

- <span data-ttu-id="88cb8-172">클레임 인식 애플리케이션을 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-172">You can leverage claims-aware applications.</span></span>
- <span data-ttu-id="88cb8-173">인증 시 외부 파트너를 신뢰할 수 있는 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-173">Provides the ability to trust external partners for authentication.</span></span>
- <span data-ttu-id="88cb8-174">대규모 인증 프로토콜 집합과 호환됩니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-174">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="88cb8-175">**과제**</span><span class="sxs-lookup"><span data-stu-id="88cb8-175">**Challenges**</span></span>

- <span data-ttu-id="88cb8-176">Azure에서 고유한 AD DS, AD FS 및 AD FS 웹 애플리케이션 프록시 서버를 배포해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-176">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
- <span data-ttu-id="88cb8-177">이 아키텍처를 구성하는 것은 복잡할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="88cb8-177">This architecture can be complex to configure.</span></span>

<span data-ttu-id="88cb8-178">**참조 아키텍처**</span><span class="sxs-lookup"><span data-stu-id="88cb8-178">**Reference architecture**</span></span>

- <span data-ttu-id="88cb8-179">[Azure로 AD FS(Active Directory Federation Services) 확장][adfs]</span><span class="sxs-lookup"><span data-stu-id="88cb8-179">[Extend Active Directory Federation Services (AD FS) to Azure][adfs]</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
