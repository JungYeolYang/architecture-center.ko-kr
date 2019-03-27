---
title: SQL Server를 사용하는 Windows N 계층 애플리케이션
titleSuffix: Azure Reference Architectures
description: 가용성, 보안, 확장성 및 관리 효율성을 위해 Azure에서 다중 계층 아키텍처를 구현합니다.
author: MikeWasson
ms.date: 11/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: 200d2e18890582a7c1551ef5053e388c4a5f29a3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242954"
---
# <a name="windows-n-tier-application-on-azure-with-sql-server"></a><span data-ttu-id="290e6-103">SQL Server를 사용한 Azure의 Windows N계층 애플리케이션</span><span class="sxs-lookup"><span data-stu-id="290e6-103">Windows N-tier application on Azure with SQL Server</span></span>

<span data-ttu-id="290e6-104">이 참조 아키텍처에서는 데이터 계층에 대해 Windows에서 SQL Server를 사용하여 [N 계층](../../guide/architecture-styles/n-tier.md) 애플리케이션을 위해 구성되는 VM 및 가상 네트워크를 배포하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-104">This reference architecture shows how to deploy VMs and a virtual network configured for an [N-tier](../../guide/architecture-styles/n-tier.md) application, using SQL Server on Windows for the data tier.</span></span> <span data-ttu-id="290e6-105">[**이 솔루션을 배포합니다**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="290e6-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Microsoft Azure를 사용하는 N계층 아키텍처](./images/n-tier-sql-server.png)

<span data-ttu-id="290e6-107">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="290e6-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="290e6-108">아키텍처</span><span class="sxs-lookup"><span data-stu-id="290e6-108">Architecture</span></span>

<span data-ttu-id="290e6-109">이 아키텍처의 구성 요소는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-109">The architecture has the following components:</span></span>

- <span data-ttu-id="290e6-110">**리소스 그룹**.</span><span class="sxs-lookup"><span data-stu-id="290e6-110">**Resource group**.</span></span> <span data-ttu-id="290e6-111">[리소스 그룹][resource-manager-overview]은 리소스를 수명, 소유자를 비롯한 기준으로 관리할 수 있도록 리소스를 그룹화하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-111">[Resource groups][resource-manager-overview] are used to group resources so they can be managed by lifetime, owner, or other criteria.</span></span>

- <span data-ttu-id="290e6-112">**VNet(가상 네트워크) 및 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="290e6-112">**Virtual network (VNet) and subnets**.</span></span> <span data-ttu-id="290e6-113">모든 Azure VM은 VNet에 배포되어 서브넷으로 분할될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-113">Every Azure VM is deployed into a VNet that can be segmented into subnets.</span></span> <span data-ttu-id="290e6-114">각 계층에 대해 별도의 서브넷을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-114">Create a separate subnet for each tier.</span></span>

- <span data-ttu-id="290e6-115">**Application Gateway**</span><span class="sxs-lookup"><span data-stu-id="290e6-115">**Application gateway**.</span></span> <span data-ttu-id="290e6-116">[Azure Application Gateway](/azure/application-gateway/)는 계층 7 부하 분산 장치입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-116">[Azure Application Gateway](/azure/application-gateway/) is a layer 7 load balancer.</span></span> <span data-ttu-id="290e6-117">이 아키텍처에서 HTTP 요청을 웹 프런트 엔드로 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-117">In this architecture, it routes HTTP requests to the web front end.</span></span> <span data-ttu-id="290e6-118">또한 application Gateway는 일반적인 악용 및 취약점으로부터 애플리케이션을 보호하는 WAF([웹 애플리케이션 방화벽](/azure/application-gateway/waf-overview))을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-118">Application Gateway also provides a [web application firewall](/azure/application-gateway/waf-overview) (WAF) that protects the application from common exploits and vulnerabilities.</span></span>

- <span data-ttu-id="290e6-119">**NSG**.</span><span class="sxs-lookup"><span data-stu-id="290e6-119">**NSGs**.</span></span> <span data-ttu-id="290e6-120">[NSG(네트워크 보안 그룹)][nsg]을 사용하여 VNet 내 네트워크 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-120">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="290e6-121">예를 들어 여기에 표시된 3계층 아키텍처에서 데이터베이스 계층은 비즈니스 계층 및 관리 서브넷뿐 아니라 웹 프론트 엔드의 트래픽을 허용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-121">For example, in the three-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>

- <span data-ttu-id="290e6-122">**DDoS Protection**</span><span class="sxs-lookup"><span data-stu-id="290e6-122">**DDoS Protection**.</span></span> <span data-ttu-id="290e6-123">Azure 플랫폼이 DDoS(분산 서비스 거부) 공격에 대한 기본 보호를 제공하지만 [DDoS Protection 표준][ddos]을 사용하는 것이 좋습니다. 그러면 DDoS 완화를 강화하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-123">Although the Azure platform provides basic protection against distributed denial of service (DDoS) attacks, we recommend using [DDoS Protection Standard][ddos], which has enhanced DDoS mitigation features.</span></span> <span data-ttu-id="290e6-124">[보안 고려사항](#security-considerations)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-124">See [Security considerations](#security-considerations).</span></span>

- <span data-ttu-id="290e6-125">**가상 머신**.</span><span class="sxs-lookup"><span data-stu-id="290e6-125">**Virtual machines**.</span></span> <span data-ttu-id="290e6-126">VM 구성 권장 사항은 [Azure에서 Windows VM 실행](./windows-vm.md) 및 [Azure에서 Linux VM 실행](./linux-vm.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-126">For recommendations on configuring VMs, see [Run a Windows VM on Azure](./windows-vm.md) and [Run a Linux VM on Azure](./linux-vm.md).</span></span>

- <span data-ttu-id="290e6-127">**가용성 집합**.</span><span class="sxs-lookup"><span data-stu-id="290e6-127">**Availability sets**.</span></span> <span data-ttu-id="290e6-128">각 계층에 [가용성 집합][azure-availability-sets]을 만들고, 각 계층에서 적어도 두 개의 VM을 프로비전하면 VM은 높은 [SLA(서비스 수준 계약)][vm-sla]에도 적합해집니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-128">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier, which makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla].</span></span>

- <span data-ttu-id="290e6-129">**부하 분산 장치**.</span><span class="sxs-lookup"><span data-stu-id="290e6-129">**Load balancers**.</span></span> <span data-ttu-id="290e6-130">[Azure Load Balancer][load-balancer]를 사용하여 웹 계층에서 비즈니스 계층으로, 비즈니스 계층에서 SQL Server로 네트워크 트래픽을 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-130">Use [Azure Load Balancer][load-balancer] to distribute network traffic from the web tier to the business tier, and from the business tier to SQL Server.</span></span>

- <span data-ttu-id="290e6-131">**공용 IP 주소**.</span><span class="sxs-lookup"><span data-stu-id="290e6-131">**Public IP address**.</span></span> <span data-ttu-id="290e6-132">애플리케이션이 인터넷 트래픽을 수신하려면 공용 IP 주소가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-132">A public IP address is needed for the application to receive Internet traffic.</span></span>

- <span data-ttu-id="290e6-133">**Jumpbox**.</span><span class="sxs-lookup"><span data-stu-id="290e6-133">**Jumpbox**.</span></span> <span data-ttu-id="290e6-134">[요새 호스트]라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-134">Also called a [bastion host].</span></span> <span data-ttu-id="290e6-135">관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-135">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="290e6-136">jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-136">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="290e6-137">NSG는 RDP(원격 데스크톱) 트래픽을 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-137">The NSG should permit remote desktop (RDP) traffic.</span></span>

- <span data-ttu-id="290e6-138">**SQL Server Always On 가용성 그룹**.</span><span class="sxs-lookup"><span data-stu-id="290e6-138">**SQL Server Always On Availability Group**.</span></span> <span data-ttu-id="290e6-139">복제 및 장애 조치(failover)를 사용하여 데이터 계층에서 높은 가용성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-139">Provides high availability at the data tier, by enabling replication and failover.</span></span> <span data-ttu-id="290e6-140">장애 조치(failover)에 대해 WSFC(Windows Server 장애 조치 클러스터) 기술을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-140">It uses Windows Server Failover Cluster (WSFC) technology for failover.</span></span>

- <span data-ttu-id="290e6-141">**AD DS(Active Directory Domain Services) 서버**.</span><span class="sxs-lookup"><span data-stu-id="290e6-141">**Active Directory Domain Services (AD DS) Servers**.</span></span> <span data-ttu-id="290e6-142">장애 조치(failover) 클러스터 및 관련 클러스터형 역할에 대한 컴퓨터 개체는 AD DS(Active Directory Domain Services)에서 만들어집니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-142">The computer objects for the failover cluster and its associated clustered roles are created in Active Directory Domain Services (AD DS).</span></span>

- <span data-ttu-id="290e6-143">**클라우드 감시**.</span><span class="sxs-lookup"><span data-stu-id="290e6-143">**Cloud Witness**.</span></span> <span data-ttu-id="290e6-144">장애 조치(failover) 클러스터는 쿼럼이 있는 것으로 알려진 해당 노드의 절반을 초과하여 실행되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-144">A failover cluster requires more than half of its nodes to be running, which is known as having quorum.</span></span> <span data-ttu-id="290e6-145">클러스터에 두 개의 노드만 있는 경우 네트워크 파티션은 각 노드가 마스터 노드라고 인지하게 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-145">If the cluster has just two nodes, a network partition could cause each node to think it's the master node.</span></span> <span data-ttu-id="290e6-146">그 경우에 연결을 차단하고 쿼럼을 설정할 *감시*가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-146">In that case, you need a *witness* to break ties and establish quorum.</span></span> <span data-ttu-id="290e6-147">감시는 쿼럼을 설정하려면 연결 차단기로 작동할 수 있는 공유 디스크 같은 리소스입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-147">A witness is a resource such as a shared disk that can act as a tie breaker to establish quorum.</span></span> <span data-ttu-id="290e6-148">클라우드 감시는 Azure Blob Storage를 사용하는 감시의 한 유형입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-148">Cloud Witness is a type of witness that uses Azure Blob Storage.</span></span> <span data-ttu-id="290e6-149">쿼럼의 개념에 대한 자세한 알려면 [클러스터 및 풀 쿼럼 이해](/windows-server/storage/storage-spaces/understand-quorum)를 참조합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-149">To learn more about the concept of quorum, see [Understanding cluster and pool quorum](/windows-server/storage/storage-spaces/understand-quorum).</span></span> <span data-ttu-id="290e6-150">클라우드 감시에 대한 자세한 내용은 [장애 조치(Failover) 클러스터에 대한 클라우드 감시 배포](/windows-server/failover-clustering/deploy-cloud-witness)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-150">For more information about Cloud Witness, see [Deploy a Cloud Witness for a Failover Cluster](/windows-server/failover-clustering/deploy-cloud-witness).</span></span>

- <span data-ttu-id="290e6-151">**Azure DNS**.</span><span class="sxs-lookup"><span data-stu-id="290e6-151">**Azure DNS**.</span></span> <span data-ttu-id="290e6-152">[Azure DNS][azure-dns]는 DNS 도메인에 대한 호스팅 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-152">[Azure DNS][azure-dns] is a hosting service for DNS domains.</span></span> <span data-ttu-id="290e6-153">이 서비스는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-153">It provides name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="290e6-154">Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-154">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="290e6-155">권장 사항</span><span class="sxs-lookup"><span data-stu-id="290e6-155">Recommendations</span></span>

<span data-ttu-id="290e6-156">개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-156">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="290e6-157">여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-157">Use these recommendations as a starting point.</span></span>

### <a name="vnet--subnets"></a><span data-ttu-id="290e6-158">VNet/서브넷</span><span class="sxs-lookup"><span data-stu-id="290e6-158">VNet / Subnets</span></span>

<span data-ttu-id="290e6-159">VNet을 만들 때는 각 서브넷에 포함된 리소스에 몇 개의 IP 주소가 필요한지 결정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-159">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="290e6-160">[CIDR] 표기법을 사용하여 필요한 IP 주소를 충족하는 서브넷 마스크와 VNet 주소 범위를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-160">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="290e6-161">표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12 및 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-161">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="290e6-162">추후 VNet과 온-프레미스 네트워크 사이에 게이트웨이를 설정해야 할 경우에 대비하여 온-프레미스 네트워크와 중복되지 않는 주소 범위를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-162">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premises network later.</span></span> <span data-ttu-id="290e6-163">VNet을 만든 뒤에는 주소 범위를 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-163">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="290e6-164">기능 및 보안 요구 사항을 염두에 두고 서브넷을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-164">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="290e6-165">동일한 계층이나 역할에 속한 모든 VM은 동일한 서브넷에 속해야 합니다. 이때 서브넷은 보안 경계가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-165">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="290e6-166">VNet 및 서브넷 디자인에 대한 자세한 내용은 [Azure Virtual Networks 계획 및 디자인][plan-network]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-166">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

### <a name="load-balancers"></a><span data-ttu-id="290e6-167">부하 분산 장치</span><span class="sxs-lookup"><span data-stu-id="290e6-167">Load balancers</span></span>

<span data-ttu-id="290e6-168">VM을 인터넷에 직접 노출시키는 대신 각 VM에 사설 IP 주소를 부여합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-168">Don't expose the VMs directly to the Internet, but instead give each VM a private IP address.</span></span> <span data-ttu-id="290e6-169">클라이언트는 Application Gateway와 연결된 공용 IP 주소를 사용하여 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-169">Clients connect using the public IP address associated with the Application Gateway.</span></span>

<span data-ttu-id="290e6-170">네트워크 트래픽이 VM으로 전달되도록 부하 분산 장치 규칙을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-170">Define load balancer rules to direct network traffic to the VMs.</span></span> <span data-ttu-id="290e6-171">예를 들어 HTTP 트래픽을 허용하려면 프론트 엔드 구성의 포트 80을 백엔드 주소 풀의 포트 80으로 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-171">For example, to enable HTTP traffic, map port 80 from the front-end configuration to port 80 on the back-end address pool.</span></span> <span data-ttu-id="290e6-172">클라이언트가 포트 80으로 HTTP 요청을 전송하면 부하 분산 장치가 소스 IP 주소를 포함하는 [해싱 알고리즘][load-balancer-hashing]을 사용하여 백엔드 IP 주소를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-172">When a client sends an HTTP request to port 80, the load balancer selects a back-end IP address by using a [hashing algorithm][load-balancer-hashing] that includes the source IP address.</span></span> <span data-ttu-id="290e6-173">클라이언트 요청은 백 엔드 주소 풀의 모든 VM에 걸쳐 분산됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-173">Client requests are distributed across all the VMs in the back-end address pool.</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="290e6-174">네트워크 보안 그룹</span><span class="sxs-lookup"><span data-stu-id="290e6-174">Network security groups</span></span>

<span data-ttu-id="290e6-175">NSG 규칙을 사용하여 계층 사이의 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-175">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="290e6-176">위에 표시된 3계층 아키텍처에서 웹 계층은 데이터베이스 계층과 직접 통신하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-176">In the three-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="290e6-177">이를 위해서는 데이터베이스 계층에서 웹 계층 서브넷으로부터 수신되는 트래픽을 차단해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-177">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>

1. <span data-ttu-id="290e6-178">VNet의 모든 인바운드 트래픽을 거부합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-178">Deny all inbound traffic from the VNet.</span></span> <span data-ttu-id="290e6-179">(규칙에 `VIRTUAL_NETWORK` 태그를 사용합니다.)</span><span class="sxs-lookup"><span data-stu-id="290e6-179">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span>
2. <span data-ttu-id="290e6-180">비즈니스 계층 서브넷의 인바운드 트래픽을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-180">Allow inbound traffic from the business tier subnet.</span></span>
3. <span data-ttu-id="290e6-181">데이터베이스 계층 서브넷 자체의 인바운드 트래픽을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-181">Allow inbound traffic from the database tier subnet itself.</span></span> <span data-ttu-id="290e6-182">이 규칙은 데이터베이스 복제와 장애 조치에 필요한 데이터베이스 VM 간 통신을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-182">This rule allows communication between the database VMs, which is needed for database replication and failover.</span></span>
4. <span data-ttu-id="290e6-183">jumpbox 서브넷에서 RDP 트래픽(3389 포트)을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-183">Allow RDP traffic (port 3389) from the jumpbox subnet.</span></span> <span data-ttu-id="290e6-184">관리자는 이 규칙을 사용하여 jumpbox에서 데이터베이스 계층에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-184">This rule lets administrators connect to the database tier from the jumpbox.</span></span>

<span data-ttu-id="290e6-185">첫 번째 규칙보다 우선 순위가 높은 2&ndash;4 규칙을 만들어 재정의합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-185">Create rules 2 &ndash; 4 with higher priority than the first rule, so they override it.</span></span>

### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="290e6-186">SQL Server Always On 가용성 그룹</span><span class="sxs-lookup"><span data-stu-id="290e6-186">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="290e6-187">SQL Server의 고가용성을 위해 [Always On 가용성 그룹][sql-alwayson]을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-187">We recommend [Always On Availability Groups][sql-alwayson] for SQL Server high availability.</span></span> <span data-ttu-id="290e6-188">Windows Server 2016 전 버전에서는 Always On 가용성 그룹에 도메인 컨트롤러가 필요하며, 가용성 그룹의 모든 노드가 동일한 AD 도메인에 속해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-188">Prior to Windows Server 2016, Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same AD domain.</span></span>

<span data-ttu-id="290e6-189">다른 계층은 [가용성 그룹 수신기][sql-alwayson-listeners]를 통해 데이터베이스에 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-189">Other tiers connect to the database through an [availability group listener][sql-alwayson-listeners].</span></span> <span data-ttu-id="290e6-190">수신기는 SQL 클라이언트가 SQL Server의 물리적 인스턴스의 이름을 알지 못해도 연결할 수 있도록 해 줍니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-190">The listener enables a SQL client to connect without knowing the name of the physical instance of SQL Server.</span></span> <span data-ttu-id="290e6-191">데이터베이스에 액세스하는 VM은 도메인에 연결되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-191">VMs that access the database must be joined to the domain.</span></span> <span data-ttu-id="290e6-192">클라이언트(여기서는 다른 계층)는 DNS를 사용하여 수신기의 가상 네트워크 이름을 IP 주소로 해석합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-192">The client (in this case, another tier) uses DNS to resolve the listener's virtual network name into IP addresses.</span></span>

<span data-ttu-id="290e6-193">다음과 같이 SQL Server Always On 가용성 그룹을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-193">Configure the SQL Server Always On Availability Group as follows:</span></span>

1. <span data-ttu-id="290e6-194">WSFC(Windows Server 장애 조치 클러스터링) 클러스터, SQL Server Always On 가용성 그룹과 주 복제본을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-194">Create a Windows Server Failover Clustering (WSFC) cluster, a SQL Server Always On Availability Group, and a primary replica.</span></span> <span data-ttu-id="290e6-195">자세한 내용은 [Always On 가용성 그룹 시작][sql-alwayson-getting-started]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-195">For more information, see [Getting Started with Always On Availability Groups][sql-alwayson-getting-started].</span></span>
2. <span data-ttu-id="290e6-196">고정 사설 IP 주소를 사용하여 내부 부하 분산 장치를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-196">Create an internal load balancer with a static private IP address.</span></span>
3. <span data-ttu-id="290e6-197">가용성 그룹 수신기를 만든 다음 수신기의 DNS 이름을 내부 부하 분산 장치의 IP 주소로 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-197">Create an availability group listener, and map the listener's DNS name to the IP address of an internal load balancer.</span></span>
4. <span data-ttu-id="290e6-198">SQL Server 수신 포트(기본값: TCP 포트 1433)에 대한 부하 분산 장치 규칙을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-198">Create a load balancer rule for the SQL Server listening port (TCP port 1433 by default).</span></span> <span data-ttu-id="290e6-199">부하 분산 장치 규칙은 Direct Server Return이라고도 불리는 *부동 IP*를 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-199">The load balancer rule must enable *floating IP*, also called Direct Server Return.</span></span> <span data-ttu-id="290e6-200">이로 인해 VM은 클라이언트에 직접 응답하여 주 복제본에 대한 직접 연결을 지원하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-200">This causes the VM to reply directly to the client, which enables a direct connection to the primary replica.</span></span>

   > [!NOTE]
   > <span data-ttu-id="290e6-201">부동 IP가 지원된 경우에는 부하 분산 장치 규칙의 프론트 엔드 포트 번호가 백엔드 포트 번호와 같아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-201">When floating IP is enabled, the front-end port number must be the same as the back-end port number in the load balancer rule.</span></span>
   >

<span data-ttu-id="290e6-202">SQL 클라이언트가 연결을 시도하면 부하 분산 장치가 연결 요청을 주 복제본으로 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-202">When a SQL client tries to connect, the load balancer routes the connection request to the primary replica.</span></span> <span data-ttu-id="290e6-203">다른 복제본으로의 장애 조치(failover)가 이루어지면 부하 분산 장치는 자동으로 새로운 요청을 새로운 주 복제본에 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-203">If there is a failover to another replica, the load balancer automatically routes new requests to a new primary replica.</span></span> <span data-ttu-id="290e6-204">자세한 내용은 [SQL Server Always On 가용성 그룹에 대한 ILB 수신기 구성][sql-alwayson-ilb]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-204">For more information, see [Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb].</span></span>

<span data-ttu-id="290e6-205">장애 조치(failover)가 진행되는 동안에는 기존 클라이언트 연결이 닫힙니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-205">During a failover, existing client connections are closed.</span></span> <span data-ttu-id="290e6-206">장애 조치(failover)가 완료되면 새로운 연결이 새로운 주 복제본으로 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-206">After the failover completes, new connections will be routed to the new primary replica.</span></span>

<span data-ttu-id="290e6-207">애플리케이션에서 쓰기보다 읽기가 훨씬 많이 발생한다면 읽기 전용 쿼리 중 일부를 보조 복제본으로 부하 분산할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-207">If your application makes significantly more reads than writes, you can offload some of the read-only queries to a secondary replica.</span></span> <span data-ttu-id="290e6-208">[수신기를 사용하여 읽기 전용 보조 복제본에 연결(읽기 전용 라우팅)][sql-alwayson-read-only-routing]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-208">See [Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing].</span></span>

<span data-ttu-id="290e6-209">가용성 그룹의 [수동 장애 조치(failover)를 강제로 수행][sql-alwayson-force-failover]하여 배포 환경을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-209">Test your deployment by [forcing a manual failover][sql-alwayson-force-failover] of the availability group.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="290e6-210">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="290e6-210">Jumpbox</span></span>

<span data-ttu-id="290e6-211">공용 인터넷으로부터 애플리케이션 워크로드를 실행하는 VM에 대한 RDP 액세스를 허용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-211">Don't allow RDP access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="290e6-212">대신 이러한 VM에 대한 모든 RDP 액세스는 jumpbox를 통해 이루어져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-212">Instead, all RDP access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="290e6-213">관리자는 jumpbox에 로그인한 뒤에 jumpbox에서 다른 VM에 로그인하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-213">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="290e6-214">jumpbox는 인터넷에서 수신되는 RDP 트래픽 중 알려진 안전한 IP 주소만을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-214">The jumpbox allows RDP traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="290e6-215">jumpbox에는 최소 성능 요구 사항이 있으므로 작은 VM 크기를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-215">The jumpbox has minimal performance requirements, so select a small VM size.</span></span> <span data-ttu-id="290e6-216">jumpbox에 대한 [공용 IP 주소]를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-216">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="290e6-217">jumpbox를 다른 VM과 동일한 VNet 안에 배치하되 별도의 관리 서브넷에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-217">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="290e6-218">jumpbox를 보호하려면 안전한 공용 IP 주소 집합의 RDP 연결만 허용하는 NSG 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-218">To secure the jumpbox, add an NSG rule that allows RDP connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="290e6-219">관리 서브넷으로부터 수신되는 RDP 트래픽을 허용하도록 다른 서브넷에 대한 NSG를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-219">Configure the NSGs for the other subnets to allow RDP traffic from the management subnet.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="290e6-220">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="290e6-220">Scalability considerations</span></span>

<span data-ttu-id="290e6-221">웹 및 비즈니스 계층의 경우 가용성 집합에 별도 VM을 배포하는 대신 [가상 머신 확장 집합][vmss]을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-221">For the web and business tiers, consider using [virtual machine scale sets][vmss], instead of deploying separate VMs into an availability set.</span></span> <span data-ttu-id="290e6-222">확장 집합을 통해 동일한 VM 집합을 쉽게 배포하고 관리하며, 성능 메트릭에 따라 VM의 크기를 자동으로 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-222">A scale set makes it easy to deploy and manage a set of identical VMs, and autoscale the VMs based on performance metrics.</span></span> <span data-ttu-id="290e6-223">VM들의 부하가 늘어나면 부하 분산 장치에 VM이 자동으로 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-223">As the load on the VMs increases, additional VMs are automatically added to the load balancer.</span></span> <span data-ttu-id="290e6-224">VM을 신속하게 확장해야 하거나 자동 확장이 필요한 경우 확장 집합을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-224">Consider scale sets if you need to quickly scale out VMs, or need to autoscale.</span></span>

<span data-ttu-id="290e6-225">확장 집합에 배포된 VM을 구성하는 방법에는 두 가지가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-225">There are two basic ways to configure VMs deployed in a scale set:</span></span>

- <span data-ttu-id="290e6-226">VM이 배포된 후에 확장명을 사용하여 VM을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-226">Use extensions to configure the VM after it's deployed.</span></span> <span data-ttu-id="290e6-227">이렇게 하면 확장명이 없는 VM보다 새 VM 인스턴스를 시작하는 데 시간이 더 오래 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-227">With this approach, new VM instances may take longer to start up than a VM with no extensions.</span></span>

- <span data-ttu-id="290e6-228">사용자 지정 디스크 이미지를 사용하여 [관리 디스크](/azure/storage/storage-managed-disks-overview)를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-228">Deploy a [managed disk](/azure/storage/storage-managed-disks-overview) with a custom disk image.</span></span> <span data-ttu-id="290e6-229">이 옵션을 사용하면 배포 시간이 단축될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-229">This option may be quicker to deploy.</span></span> <span data-ttu-id="290e6-230">하지만 그러러면 이미지를 최신 상태로 유지해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-230">However, it requires you to keep the image up-to-date.</span></span>

<span data-ttu-id="290e6-231">자세한 내용은 [확장 집합 디자인 고려 사항][vmss-design]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-231">For more information, see [Design considerations for scale sets][vmss-design].</span></span>

> [!TIP]
> <span data-ttu-id="290e6-232">자동 확장 솔루션을 사용할 때는 사전에 미리 프로덕션 수준 워크로드로 테스트해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-232">When using any autoscale solution, test it with production-level workloads well in advance.</span></span>

<span data-ttu-id="290e6-233">각 Azure 구독에는 지역당 최대 VM 개수를 비롯해 기본적인 제한이 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-233">Each Azure subscription has default limits in place, including a maximum number of VMs per region.</span></span> <span data-ttu-id="290e6-234">지원 요청을 제출하여 제한을 늘릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-234">You can increase the limit by filing a support request.</span></span> <span data-ttu-id="290e6-235">자세한 내용은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건][subscription-limits]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-235">For more information, see [Azure subscription and service limits, quotas, and constraints][subscription-limits].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="290e6-236">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="290e6-236">Availability considerations</span></span>

<span data-ttu-id="290e6-237">가상 머신 확장 집합을 사용하지 않는 경우 동일한 계층의 VM을 가용성 집합에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-237">If you don't use virtual machine scale sets, put VMs for the same tier into an availability set.</span></span> <span data-ttu-id="290e6-238">[Azure VM의 가용성 SLA][vm-sla]를 지원하기 위해 가용성 집합에 2개 이상의 VM을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-238">Create at least two VMs in the availability set to support the [availability SLA for Azure VMs][vm-sla].</span></span> <span data-ttu-id="290e6-239">자세한 내용은 [가상 머신의 가용성 관리][availability-set]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-239">For more information, see [Manage the availability of virtual machines][availability-set].</span></span> <span data-ttu-id="290e6-240">확장 집합은 자동으로 *배치 그룹*을 사용합니다. 이 기능은 암시적인 가용성 집합으로 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-240">Scale sets automatically use *placement groups*, which act as an implicit availability set.</span></span>

<span data-ttu-id="290e6-241">부하 분산 장치는 [상태 프로브][health-probes]를 사용하여 VM 인스턴스의 가용성을 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-241">The load balancer uses [health probes][health-probes] to monitor the availability of VM instances.</span></span> <span data-ttu-id="290e6-242">프로브가 시간 제한 내에 인스턴스에 도달하지 못한 경우 부하 분산 장치는 해당 VM으로 전달되는 트래픽을 차단합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-242">If a probe can't reach an instance within a timeout period, the load balancer stops sending traffic to that VM.</span></span> <span data-ttu-id="290e6-243">이때도 부하 분산 장치가 프로브를 계속 진행하며, VM을 다시 사용할 수 있게 되면 해당 VM으로 전달되는 트래픽을 재개합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-243">However, the load balancer will continue to probe, and if the VM becomes available again, the load balancer resumes sending traffic to that VM.</span></span>

<span data-ttu-id="290e6-244">다음은 부하 분산 장치 상태 프로브에 대한 몇 가지 권장 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-244">Here are some recommendations on load balancer health probes:</span></span>

- <span data-ttu-id="290e6-245">프로브는 HTTP와 TCP를 모두 테스트할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-245">Probes can test either HTTP or TCP.</span></span> <span data-ttu-id="290e6-246">VM이 HTTP 서버를 실행 중인 경우에는 HTTP 프로브를 만들고,</span><span class="sxs-lookup"><span data-stu-id="290e6-246">If your VMs run an HTTP server, create an HTTP probe.</span></span> <span data-ttu-id="290e6-247">HTTP 서버를 실행하지 않는 경우에는 TCP 프로브를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-247">Otherwise create a TCP probe.</span></span>
- <span data-ttu-id="290e6-248">HTTP 프로브를 만들 때는 HTTP 엔드포인트로 향하는 경로를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-248">For an HTTP probe, specify the path to an HTTP endpoint.</span></span> <span data-ttu-id="290e6-249">프로브는 이 경로에서 HTTP 200 응답을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-249">The probe checks for an HTTP 200 response from this path.</span></span> <span data-ttu-id="290e6-250">이 경로는 루트 경로("/")일 수도 있고, 애플리케이션의 상태를 확인하는 일부 사용자 지정 로직을 구현하는 상태 모니터링 엔드포인트일 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-250">This path can be the root path ("/"), or a health-monitoring endpoint that implements some custom logic to check the health of the application.</span></span> <span data-ttu-id="290e6-251">엔드포인트는 익명의 HTTP 요청을 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-251">The endpoint must allow anonymous HTTP requests.</span></span>
- <span data-ttu-id="290e6-252">프로브는 [알려진 IP 주소][health-probe-ip]인 168.63.129.16에서 전송됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-252">The probe is sent from a [known IP address][health-probe-ip], 168.63.129.16.</span></span> <span data-ttu-id="290e6-253">모든 방화벽 정책 또는 NSG 규칙에서 이 IP 주소 간에 이동하는 트래픽을 차단하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-253">Don't block traffic to or from this IP address in any firewall policies or NSG rules.</span></span>
- <span data-ttu-id="290e6-254">[상태 프로브 로그][health-probe-log]를 사용하여 상태 프로브의 상태를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-254">Use [health probe logs][health-probe-log] to view the status of the health probes.</span></span> <span data-ttu-id="290e6-255">Azure Portal에서 각 부하 분산 장치에 대해 로깅을 활성화합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-255">Enable logging in the Azure portal for each load balancer.</span></span> <span data-ttu-id="290e6-256">로그는 Azure Blob Storage에 쓰기됩니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-256">Logs are written to Azure Blob storage.</span></span> <span data-ttu-id="290e6-257">로그에서는 실패한 프로브 응답으로 인해 네트워크 트래픽을 가져오지 않는 VM 개수를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-257">The logs show how many VMs aren't getting network traffic because of failed probe responses.</span></span>

<span data-ttu-id="290e6-258">[VM용 Azure SLA][vm-sla]에서 제공하는 가용성보다 더 높은 가용성이 필요한 경우, 장애 조치를 위해 Azure Traffic Manager를 사용하여 두 지역 간에 애플리케이션을 복제하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-258">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, consider replication the application across two regions, using Azure Traffic Manager for failover.</span></span> <span data-ttu-id="290e6-259">자세한 내용은 [고가용성을 위한 다중 지역 N 계층 애플리케이션][multi-dc]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-259">For more information, see [Multi-region N-tier application for high availability][multi-dc].</span></span>

## <a name="security-considerations"></a><span data-ttu-id="290e6-260">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="290e6-260">Security considerations</span></span>

<span data-ttu-id="290e6-261">가상 네트워크는 Azure의 트래픽 격리 경계입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-261">Virtual networks are a traffic isolation boundary in Azure.</span></span> <span data-ttu-id="290e6-262">하나의 VNet에 속한 VM은 다른 VNet에 속한 VM과 직접 통신할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-262">VMs in one VNet can't communicate directly with VMs in a different VNet.</span></span> <span data-ttu-id="290e6-263">하나의 VNet에 속한 여러 VM은 사용자가 트래픽을 제한하기 위해 NSG([네트워크 보안 그룹][nsg])를 만들지 않은 이상 서로 통신할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-263">VMs within the same VNet can communicate, unless you create [network security groups][nsg] (NSGs) to restrict traffic.</span></span> <span data-ttu-id="290e6-264">자세한 내용은 [Microsoft 클라우드 서비스 및 네트워크 보안][network-security]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-264">For more information, see [Microsoft cloud services and network security][network-security].</span></span>

<span data-ttu-id="290e6-265">**DMZ**</span><span class="sxs-lookup"><span data-stu-id="290e6-265">**DMZ**.</span></span> <span data-ttu-id="290e6-266">NVA(네트워크 가상 어플라이언스)를 추가하여 인터넷과 Azure 가상 네트워크 사이에 DMZ를 만드는 것도 좋은 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-266">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="290e6-267">NVA는 방화벽, 패킷 조사, 감사, 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행하는 가상 어플라이언스를 통칭하는 용어입니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-267">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="290e6-268">자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-268">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

<span data-ttu-id="290e6-269">**암호화**</span><span class="sxs-lookup"><span data-stu-id="290e6-269">**Encryption**.</span></span> <span data-ttu-id="290e6-270">중요한 미사용 데이터를 암호화하고 [Azure Key Vault][azure-key-vault]를 사용하여 데이터베이스 암호화 키를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-270">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="290e6-271">Key Vault는 암호화 키를 HSM(하드웨어 보안 모듈)에 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-271">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="290e6-272">자세한 내용은 [Azure VM에서 SQL Server에 대한 Azure Key Vault 통합 구성][sql-keyvault]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-272">For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault].</span></span> <span data-ttu-id="290e6-273">또한 Key Vault에 데이터베이스 연결 문자열과 같은 애플리케이션 비밀을 저장하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-273">It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

<span data-ttu-id="290e6-274">**DDoS 보호**</span><span class="sxs-lookup"><span data-stu-id="290e6-274">**DDoS protection**.</span></span> <span data-ttu-id="290e6-275">Azure 플랫폼은 기본적으로 기본 DDoS 보호를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-275">The Azure platform provides basic DDoS protection by default.</span></span> <span data-ttu-id="290e6-276">이 기본 보호는 전체 Azure 인프라를 보호할 대상으로 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-276">This basic protection is targeted at protecting the Azure infrastructure as a whole.</span></span> <span data-ttu-id="290e6-277">기본 DDoS 보호가 자동으로 설정되지만 [DDoS Protection 표준][ddos]을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-277">Although basic DDoS protection is automatically enabled, we recommend using [DDoS Protection Standard][ddos].</span></span> <span data-ttu-id="290e6-278">표준 보호는 애플리케이션의 네트워크 트래픽 패턴을 기반으로 적응형 조정을 사용하여 위협을 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-278">Standard protection uses adaptive tuning, based on your application's network traffic patterns, to detect threats.</span></span> <span data-ttu-id="290e6-279">그러면 인프라 수준의 DDoS 정책에 의해 알려지지 않을 수 있는 DDoS 공격에 대한 완화를 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-279">This allows it to apply mitigations against DDoS attacks that might go unnoticed by the infrastructure-wide DDoS policies.</span></span> <span data-ttu-id="290e6-280">표준 보호는 Azure Monitor를 통해 경고, 원격 분석 및 분석도 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-280">Standard protection also provides alerting, telemetry, and analytics through Azure Monitor.</span></span> <span data-ttu-id="290e6-281">자세한 내용은 [Azure DDoS Protection: 모범 사례 및 참조 아키텍처][ddos-best-practices]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-281">For more information, see [Azure DDoS Protection: Best practices and reference architectures][ddos-best-practices].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="290e6-282">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="290e6-282">Deploy the solution</span></span>

<span data-ttu-id="290e6-283">이 참조 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-283">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="290e6-284">AD DS, Windows Server 장애 조치(failover) 클러스터 및 SQL Server 가용성 그룹을 구성하는 스크립트 실행을 포함하여 전체 배포에는 최대 2시간이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-284">The entire deployment can take up to two hours, which includes running the scripts to configure AD DS, the Windows Server failover cluster, and the SQL Server availability group.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="290e6-285">필수 조건</span><span class="sxs-lookup"><span data-stu-id="290e6-285">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deployment-steps"></a><span data-ttu-id="290e6-286">배포 단계</span><span class="sxs-lookup"><span data-stu-id="290e6-286">Deployment steps</span></span>

1. <span data-ttu-id="290e6-287">다음 명령을 실행하여 리소스 그룹을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-287">Run the following command to create a resource group.</span></span>

    ```azurecli
    az group create --location <location> --name <resource-group-name>
    ```

2. <span data-ttu-id="290e6-288">클라우드 감시에 대한 저장소 계정을 만들려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-288">Run the following command to create a Storage account for the Cloud Witness.</span></span>

    ```azurecli
    az storage account create --location <location> \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --sku Standard_LRS
    ```

3. <span data-ttu-id="290e6-289">참조 아키텍처 GitHub 리포지토리의 `virtual-machines\n-tier-windows` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-289">Navigate to the `virtual-machines\n-tier-windows` folder of the reference architectures GitHub repository.</span></span>

4. <span data-ttu-id="290e6-290">`n-tier-windows.json` 파일을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-290">Open the `n-tier-windows.json` file.</span></span>

5. <span data-ttu-id="290e6-291">"witnessStorageBlobEndPoint"의 모든 인스턴스를 검색하고 2단계에서 자리 표시자 텍스트를 스토리지 계정의 이름으로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-291">Search for all instances of "witnessStorageBlobEndPoint" and replace the placeholder text with the name of the Storage account from step 2.</span></span>

    ```json
    "witnessStorageBlobEndPoint": "https://[replace-with-storageaccountname].blob.core.windows.net",
    ```

6. <span data-ttu-id="290e6-292">다음 명령을 실행하여 저장소 계정에 대한 계정 키를 나열합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-292">Run the following command to list the account keys for the storage account.</span></span>

    ```azurecli
    az storage account keys list \
      --account-name <storage-account-name> \
      --resource-group <resource-group-name>
    ```

    <span data-ttu-id="290e6-293">출력은 다음과 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-293">The output should look like the following.</span></span> <span data-ttu-id="290e6-294">`key1`의 값을 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-294">Copy the value of `key1`.</span></span>

    ```json
    [
    {
        "keyName": "key1",
        "permissions": "Full",
        "value": "..."
    },
    {
        "keyName": "key2",
        "permissions": "Full",
        "value": "..."
    }
    ]
    ```

7. <span data-ttu-id="290e6-295">`n-tier-windows.json` 파일에서 "witnessStorageAccountKey"의 모든 인스턴스를 검색하여 계정 키에 붙여넣습니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-295">In the `n-tier-windows.json` file, search for all instances of "witnessStorageAccountKey" and paste in the account key.</span></span>

    ```json
    "witnessStorageAccountKey": "[replace-with-storagekey]"
    ```

8. <span data-ttu-id="290e6-296">`n-tier-windows.json` 파일에서 `[replace-with-password]`의 모든 인스턴스를 검색하고 `[replace-with-sql-password]`를 강력한 암호로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-296">In the `n-tier-windows.json` file, search for all instances of `[replace-with-password]` and `[replace-with-sql-password]` replace them with a strong password.</span></span> <span data-ttu-id="290e6-297">파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-297">Save the file.</span></span>

    > [!NOTE]
    > <span data-ttu-id="290e6-298">관리자 사용자 이름을 변경하는 경우 JSON 파일에서 `extensions` 블록을 업데이트해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-298">If you change the administrator user name, you must also update the `extensions` blocks in the JSON file.</span></span>

9. <span data-ttu-id="290e6-299">다음 명령을 실행하여 아키텍처를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="290e6-299">Run the following command to deploy the architecture.</span></span>

    ```azurecli
    azbb -s <your subscription_id> -g <resource_group_name> -l <location> -p n-tier-windows.json --deploy
    ```

<span data-ttu-id="290e6-300">Azure 구성 요소를 사용하여 이 샘플 참조 아키텍처를 배포하는 방법에 대한 자세한 내용은 [GitHub 리포지토리][git]를 방문하세요.</span><span class="sxs-lookup"><span data-stu-id="290e6-300">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>

## <a name="next-steps"></a><span data-ttu-id="290e6-301">다음 단계</span><span class="sxs-lookup"><span data-stu-id="290e6-301">Next steps</span></span>

- [<span data-ttu-id="290e6-302">Microsoft 학습 모듈: N 계층 아키텍처 스타일 둘러보기</span><span class="sxs-lookup"><span data-stu-id="290e6-302">Microsoft Learn module: Tour the N-tier architecture style</span></span>](/learn/modules/n-tier-architecture/)

<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
[n-tier]: n-tier.md
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[요새 호스트]: https://en.wikipedia.org/wiki/Bastion_host
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[ddos]: /azure/virtual-network/ddos-protection-overview
[ddos-best-practices]: /azure/security/azure-ddos-best-practices
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[nsg]: /azure/virtual-network/virtual-networks-nsg
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[공용 IP 주소]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
