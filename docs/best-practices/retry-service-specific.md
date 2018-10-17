---
title: 서비스 관련 재시도 지침
description: 재시도 메커니즘 설정에 대한 서비스 관련 지침입니다.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: c5a9bc99c4693f35c38dabcf07b3465add6a8cb1
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429553"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="9e061-103">특정 서비스에 대한 다시 시도 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-103">Retry guidance for specific services</span></span>

<span data-ttu-id="9e061-104">대부분의 Azure 서비스 및 클라이언트 SDK는 재시도 메커니즘을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="9e061-105">그러나 서비스마다 특성 및 요구 사항이 다르기 때문에 이러한 메커니즘을 서로 다르므로 각 재시도 메커니즘은 특정 서비스에 맞게 조정됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="9e061-106">이 가이드에서는 대부분의 Azure 서비스에 대한 재시도 메커니즘 기능을 요약하고 해당 서비스에 대한 재시도 메커니즘을 사용, 적용 또는 확장하는 데 도움이 되는 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="9e061-107">일시적인 오류 처리, 서비스와 리소스에 대해 연결 및 작업 재시도에 대한 일반 지침은 [재시도 지침](./transient-faults.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="9e061-108">다음 표에는 이 지침에서 설명하는 Azure 서비스에 대한 재시도 기능이 요약되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="9e061-109">**서비스**</span><span class="sxs-lookup"><span data-stu-id="9e061-109">**Service**</span></span> | <span data-ttu-id="9e061-110">**재시도 기능**</span><span class="sxs-lookup"><span data-stu-id="9e061-110">**Retry capabilities**</span></span> | <span data-ttu-id="9e061-111">**정책 구성**</span><span class="sxs-lookup"><span data-stu-id="9e061-111">**Policy configuration**</span></span> | <span data-ttu-id="9e061-112">**범위**</span><span class="sxs-lookup"><span data-stu-id="9e061-112">**Scope**</span></span> | <span data-ttu-id="9e061-113">**원격 분석 기능**</span><span class="sxs-lookup"><span data-stu-id="9e061-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="9e061-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="9e061-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="9e061-115">ADAL 라이브러리에서 기본</span><span class="sxs-lookup"><span data-stu-id="9e061-115">Native in ADAL library</span></span> |<span data-ttu-id="9e061-116">ADAL 라이브러리에 포함</span><span class="sxs-lookup"><span data-stu-id="9e061-116">Embeded into ADAL library</span></span> |<span data-ttu-id="9e061-117">내부</span><span class="sxs-lookup"><span data-stu-id="9e061-117">Internal</span></span> |<span data-ttu-id="9e061-118">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-118">None</span></span> |
| <span data-ttu-id="9e061-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="9e061-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="9e061-120">서비스의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-120">Native in service</span></span> |<span data-ttu-id="9e061-121">구성할 수 없음</span><span class="sxs-lookup"><span data-stu-id="9e061-121">Non-configurable</span></span> |<span data-ttu-id="9e061-122">전역</span><span class="sxs-lookup"><span data-stu-id="9e061-122">Global</span></span> |<span data-ttu-id="9e061-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="9e061-123">TraceSource</span></span> |
| <span data-ttu-id="9e061-124">**Data Lake Store**</span><span class="sxs-lookup"><span data-stu-id="9e061-124">**Data Lake Store**</span></span> |<span data-ttu-id="9e061-125">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-125">Native in client</span></span> |<span data-ttu-id="9e061-126">구성할 수 없음</span><span class="sxs-lookup"><span data-stu-id="9e061-126">Non-configurable</span></span> |<span data-ttu-id="9e061-127">개별 작업</span><span class="sxs-lookup"><span data-stu-id="9e061-127">Individual operations</span></span> |<span data-ttu-id="9e061-128">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-128">None</span></span> |
| <span data-ttu-id="9e061-129">**[Event Hubs](#event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="9e061-129">**[Event Hubs](#event-hubs)**</span></span> |<span data-ttu-id="9e061-130">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-130">Native in client</span></span> |<span data-ttu-id="9e061-131">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-131">Programmatic</span></span> |<span data-ttu-id="9e061-132">클라이언트</span><span class="sxs-lookup"><span data-stu-id="9e061-132">Client</span></span> |<span data-ttu-id="9e061-133">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-133">None</span></span> |
| <span data-ttu-id="9e061-134">**[IoT Hub](#iot-hub)**</span><span class="sxs-lookup"><span data-stu-id="9e061-134">**[IoT Hub](#iot-hub)**</span></span> |<span data-ttu-id="9e061-135">클라이언트 SDK의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-135">Native in client SDK</span></span> |<span data-ttu-id="9e061-136">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-136">Programmatic</span></span> |<span data-ttu-id="9e061-137">클라이언트</span><span class="sxs-lookup"><span data-stu-id="9e061-137">Client</span></span> |<span data-ttu-id="9e061-138">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-138">None</span></span> |
| <span data-ttu-id="9e061-139">**[Redis Cache](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="9e061-139">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="9e061-140">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-140">Native in client</span></span> |<span data-ttu-id="9e061-141">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-141">Programmatic</span></span> |<span data-ttu-id="9e061-142">클라이언트</span><span class="sxs-lookup"><span data-stu-id="9e061-142">Client</span></span> |<span data-ttu-id="9e061-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="9e061-143">TextWriter</span></span> |
| <span data-ttu-id="9e061-144">**[Search](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="9e061-144">**[Search](#azure-search)**</span></span> |<span data-ttu-id="9e061-145">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-145">Native in client</span></span> |<span data-ttu-id="9e061-146">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-146">Programmatic</span></span> |<span data-ttu-id="9e061-147">클라이언트</span><span class="sxs-lookup"><span data-stu-id="9e061-147">Client</span></span> |<span data-ttu-id="9e061-148">ETW 또는 사용자 지정</span><span class="sxs-lookup"><span data-stu-id="9e061-148">ETW or Custom</span></span> |
| <span data-ttu-id="9e061-149">**[Service Bus](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="9e061-149">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="9e061-150">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-150">Native in client</span></span> |<span data-ttu-id="9e061-151">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-151">Programmatic</span></span> |<span data-ttu-id="9e061-152">네임스페이스 관리자, 메시징 팩터리 및 클라이언트</span><span class="sxs-lookup"><span data-stu-id="9e061-152">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="9e061-153">ETW</span><span class="sxs-lookup"><span data-stu-id="9e061-153">ETW</span></span> |
| <span data-ttu-id="9e061-154">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="9e061-154">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="9e061-155">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-155">Native in client</span></span> |<span data-ttu-id="9e061-156">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-156">Programmatic</span></span> |<span data-ttu-id="9e061-157">클라이언트</span><span class="sxs-lookup"><span data-stu-id="9e061-157">Client</span></span> |<span data-ttu-id="9e061-158">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-158">None</span></span> | 
| <span data-ttu-id="9e061-159">**[ADO.NET을 사용하는 SQL Database](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="9e061-159">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="9e061-160">Polly</span><span class="sxs-lookup"><span data-stu-id="9e061-160">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="9e061-161">선언적 방식 및 프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-161">Declarative and programmatic</span></span> |<span data-ttu-id="9e061-162">코드의 단일 문 또는 블록</span><span class="sxs-lookup"><span data-stu-id="9e061-162">Single statements or blocks of code</span></span> |<span data-ttu-id="9e061-163">사용자 지정</span><span class="sxs-lookup"><span data-stu-id="9e061-163">Custom</span></span> |
| <span data-ttu-id="9e061-164">**[Entity Framework를 사용하는 SQL Database](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="9e061-164">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="9e061-165">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-165">Native in client</span></span> |<span data-ttu-id="9e061-166">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-166">Programmatic</span></span> |<span data-ttu-id="9e061-167">AppDomain에 따라 전역</span><span class="sxs-lookup"><span data-stu-id="9e061-167">Global per AppDomain</span></span> |<span data-ttu-id="9e061-168">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-168">None</span></span> |
| <span data-ttu-id="9e061-169">**[Entity Framework Core를 사용하는 SQL Database](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="9e061-169">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="9e061-170">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-170">Native in client</span></span> |<span data-ttu-id="9e061-171">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-171">Programmatic</span></span> |<span data-ttu-id="9e061-172">AppDomain에 따라 전역</span><span class="sxs-lookup"><span data-stu-id="9e061-172">Global per AppDomain</span></span> |<span data-ttu-id="9e061-173">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-173">None</span></span> |
| <span data-ttu-id="9e061-174">**[Storage](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="9e061-174">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="9e061-175">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="9e061-175">Native in client</span></span> |<span data-ttu-id="9e061-176">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="9e061-176">Programmatic</span></span> |<span data-ttu-id="9e061-177">클라이언트 및 개별 작업</span><span class="sxs-lookup"><span data-stu-id="9e061-177">Client and individual operations</span></span> |<span data-ttu-id="9e061-178">TraceSource</span><span class="sxs-lookup"><span data-stu-id="9e061-178">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="9e061-179">대부분의 Azure 기본 제공 재시도 메커니즘에서는 현재 다른 유형의 오류 또는 예외에 대해 서로 다른 재시도 정책을 적용할 방법이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-179">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="9e061-180">최적의 평균 성능 및 가용성을 제공하는 정책을 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-180">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="9e061-181">정책을 미세 조정하는 한 가지 방법은 로그 파일을 분석하여 발생하는 일시적인 오류의 유형을 확인하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-181">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="9e061-182">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="9e061-182">Azure Active Directory</span></span>
<span data-ttu-id="9e061-183">Azure AD(Azure Active Directory)는 핵심 디렉터리 서비스, 고급 ID 관리, 보안 및 응용 프로그램 액세스 관리를 결합하는 포괄적인 ID 및 액세스 관리 클라우드 솔루션입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-183">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="9e061-184">또한 Azure AD는 개발자에게 ID 관리 플랫폼을 제공하여 중앙 집중식 정책 및 규칙에 따라 해당 응용 프로그램에 대한 액세스 제어 권한을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-184">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="9e061-185">관리 서비스 ID 엔드포인트에 대한 다시 시도 지침은 [토큰 획득을 위해 Azure VM MSI(관리 서비스 ID)를 사용하는 방법](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling)을 참조합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-185">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-186">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-186">Retry mechanism</span></span>
<span data-ttu-id="9e061-187">ADAL(Active Directory 인증 라이브러리)의 Azure Active Directory에 대한 기본 제공 재시도 메커니즘은 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-187">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="9e061-188">예기치 않은 잠김을 방지하기 위해, 타사 라이브러리와 응용 프로그램 코드가 실패한 연결을 재시도하지 **않고** ADAL이 재시도를 처리하게 하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-188">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="9e061-189">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-189">Retry usage guidance</span></span>
<span data-ttu-id="9e061-190">Azure Active Directory를 사용하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-190">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="9e061-191">가능하다면 재시도에 ADAL 라이브러리와 기본 제공 지원을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-191">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="9e061-192">Azure Active Directory용 REST API를 사용하는 경우, 결과 코드가 429(너무 많은 요청)이거나 5xx 범위의 오류인 경우 작업을 재시도합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-192">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="9e061-193">다른 오류의 경우에는 재시도하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-193">Do not retry for any other errors.</span></span>
* <span data-ttu-id="9e061-194">Azure Active Directory의 일괄 처리 시나리오에는 지수 백오프 정책을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-194">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="9e061-195">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-195">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="9e061-196">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-196">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="9e061-197">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="9e061-197">**Context**</span></span> | <span data-ttu-id="9e061-198">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="9e061-198">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="9e061-199">**재시도 전략**</span><span class="sxs-lookup"><span data-stu-id="9e061-199">**Retry strategy**</span></span> | <span data-ttu-id="9e061-200">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-200">**Settings**</span></span> | <span data-ttu-id="9e061-201">**값**</span><span class="sxs-lookup"><span data-stu-id="9e061-201">**Values**</span></span> | <span data-ttu-id="9e061-202">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="9e061-202">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9e061-203">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="9e061-203">Interactive, UI,</span></span><br /><span data-ttu-id="9e061-204">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-204">or foreground</span></span> |<span data-ttu-id="9e061-205">2초</span><span class="sxs-lookup"><span data-stu-id="9e061-205">2 sec</span></span> |<span data-ttu-id="9e061-206">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="9e061-206">FixedInterval</span></span> |<span data-ttu-id="9e061-207">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="9e061-207">Retry count</span></span><br /><span data-ttu-id="9e061-208">재시도 간격</span><span class="sxs-lookup"><span data-stu-id="9e061-208">Retry interval</span></span><br /><span data-ttu-id="9e061-209">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="9e061-209">First fast retry</span></span> |<span data-ttu-id="9e061-210">3</span><span class="sxs-lookup"><span data-stu-id="9e061-210">3</span></span><br /><span data-ttu-id="9e061-211">500ms</span><span class="sxs-lookup"><span data-stu-id="9e061-211">500 ms</span></span><br /><span data-ttu-id="9e061-212">true</span><span class="sxs-lookup"><span data-stu-id="9e061-212">true</span></span> |<span data-ttu-id="9e061-213">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-213">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="9e061-214">시도 2 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-214">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="9e061-215">시도 3 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-215">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="9e061-216">백그라운드 또는</span><span class="sxs-lookup"><span data-stu-id="9e061-216">Background or</span></span><br /><span data-ttu-id="9e061-217">일괄 처리</span><span class="sxs-lookup"><span data-stu-id="9e061-217">batch</span></span> |<span data-ttu-id="9e061-218">60초</span><span class="sxs-lookup"><span data-stu-id="9e061-218">60 sec</span></span> |<span data-ttu-id="9e061-219">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-219">ExponentialBackoff</span></span> |<span data-ttu-id="9e061-220">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="9e061-220">Retry count</span></span><br /><span data-ttu-id="9e061-221">최소 백오프</span><span class="sxs-lookup"><span data-stu-id="9e061-221">Min back-off</span></span><br /><span data-ttu-id="9e061-222">최대 백오프</span><span class="sxs-lookup"><span data-stu-id="9e061-222">Max back-off</span></span><br /><span data-ttu-id="9e061-223">델타 백오프</span><span class="sxs-lookup"><span data-stu-id="9e061-223">Delta back-off</span></span><br /><span data-ttu-id="9e061-224">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="9e061-224">First fast retry</span></span> |<span data-ttu-id="9e061-225">5</span><span class="sxs-lookup"><span data-stu-id="9e061-225">5</span></span><br /><span data-ttu-id="9e061-226">0초</span><span class="sxs-lookup"><span data-stu-id="9e061-226">0 sec</span></span><br /><span data-ttu-id="9e061-227">60초</span><span class="sxs-lookup"><span data-stu-id="9e061-227">60 sec</span></span><br /><span data-ttu-id="9e061-228">2초</span><span class="sxs-lookup"><span data-stu-id="9e061-228">2 sec</span></span><br /><span data-ttu-id="9e061-229">false</span><span class="sxs-lookup"><span data-stu-id="9e061-229">false</span></span> |<span data-ttu-id="9e061-230">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-230">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="9e061-231">시도 2 - ~2초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-231">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="9e061-232">시도 3 - ~6초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-232">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="9e061-233">시도 4 - ~14초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-233">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="9e061-234">시도 5 - ~30초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-234">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="9e061-235">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-235">More information</span></span>
* <span data-ttu-id="9e061-236">[Azure Active Directory 인증 라이브러리][adal]</span><span class="sxs-lookup"><span data-stu-id="9e061-236">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="9e061-237">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="9e061-237">Cosmos DB</span></span>

<span data-ttu-id="9e061-238">Cosmos DB는 스키마 없는 JSON 데이터를 지원하는 완전 관리 다중 model 데이터베이스입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-238">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="9e061-239">DocumentDB는 구성 가능하고 안정적인 성능, 네이티브 JavaScript 트랜잭션 처리를 제공하며 탄력적인 확장성으로 클라우드에 대해 구축됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-239">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-240">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-240">Retry mechanism</span></span>
<span data-ttu-id="9e061-241">`DocumentClient` 클래스는 실패 횟수를 자동으로 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-241">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="9e061-242">재시도 횟수와 최대 대기 시간을 설정하려면 [ConnectionPolicy.RetryOptions]을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-242">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="9e061-243">클라이언트에서 발생시키는 예외는 재시도 정책 시도 횟수를 초과하거나 일시적인 오류가 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-243">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="9e061-244">Cosmos DB에서 클라이언트를 제한하는 경우 HTTP 429 오류를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-244">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="9e061-245">`DocumentClientException`에서 상태 코드를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-245">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="9e061-246">정책 구성</span><span class="sxs-lookup"><span data-stu-id="9e061-246">Policy configuration</span></span>
<span data-ttu-id="9e061-247">다음 표에서는 `RetryOptions` 클래스의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-247">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="9e061-248">설정</span><span class="sxs-lookup"><span data-stu-id="9e061-248">Setting</span></span> | <span data-ttu-id="9e061-249">기본값</span><span class="sxs-lookup"><span data-stu-id="9e061-249">Default value</span></span> | <span data-ttu-id="9e061-250">설명</span><span class="sxs-lookup"><span data-stu-id="9e061-250">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="9e061-251">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="9e061-251">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="9e061-252">9</span><span class="sxs-lookup"><span data-stu-id="9e061-252">9</span></span> |<span data-ttu-id="9e061-253">Cosmos DB가 클라이언트에서 속도 제한을 적용하기 때문에 요청에 실패하면 다시 시도하는 최대 수입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-253">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="9e061-254">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="9e061-254">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="9e061-255">30</span><span class="sxs-lookup"><span data-stu-id="9e061-255">30</span></span> |<span data-ttu-id="9e061-256">다시 시도하는 최대 시간(초)입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-256">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="9e061-257">예</span><span class="sxs-lookup"><span data-stu-id="9e061-257">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="9e061-258">원격 분석</span><span class="sxs-lookup"><span data-stu-id="9e061-258">Telemetry</span></span>
<span data-ttu-id="9e061-259">재시도 횟수는 .NET **TraceSource**를 통해 구조화되지 않은 추적 메시지로 기록됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-259">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="9e061-260">이벤트를 캡처하여 적합한 대상 로그에 기록하려면 **TraceListener**를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-260">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="9e061-261">예를 들어 App.config 파일에 다음을 추가하는 경우 추적은 실행 파일과 동일한 위치에 있는 텍스트 파일에 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-261">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a><span data-ttu-id="9e061-262">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="9e061-262">Event Hubs</span></span>

<span data-ttu-id="9e061-263">Azure Event Hubs는 수백만 개의 이벤트를 수집, 변환 및 저장하는 하이퍼스케일(hyper-scale) 원격 분석 수집 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-263">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-264">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-264">Retry mechanism</span></span>
<span data-ttu-id="9e061-265">Azure Event Hubs 클라이언트 라이브러리의 재시도 동작은 `EventHubClient` 클래스의 `RetryPolicy` 속성에서 제어됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-265">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="9e061-266">기본 정책에서는 Azure Event Hub가 일시적 `EventHubsException` 또는 `OperationCanceledException`을 반환할 때 지수 백오프를 통해 재시도합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-266">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="9e061-267">예</span><span class="sxs-lookup"><span data-stu-id="9e061-267">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="9e061-268">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-268">More information</span></span>
[<span data-ttu-id="9e061-269">Azure Event Hubs용 .NET 표준 클라이언트 라이브러리</span><span class="sxs-lookup"><span data-stu-id="9e061-269"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="iot-hub"></a><span data-ttu-id="9e061-270">IoT Hub</span><span class="sxs-lookup"><span data-stu-id="9e061-270">IoT Hub</span></span>

<span data-ttu-id="9e061-271">Azure IoT Hub는 IoT(사물 인터넷) 응용 프로그램을 개발하기 위해 장치를 연결, 모니터링 및 관리하는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-271">Azure IoT Hub is a service for connecting, monitoring, and managing devices to develop Internet of Things (IoT) applications.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-272">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-272">Retry mechanism</span></span>

<span data-ttu-id="9e061-273">Azure IoT 장치 SDK는 네트워크, 프로토콜 또는 응용 프로그램에서 오류를 검색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-273">The Azure IoT device SDK can detect errors in the network, protocol, or application.</span></span> <span data-ttu-id="9e061-274">오류 형식에 따라 SDK는 다시 시도를 수행해야 하는지 여부를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-274">Based on the error type, the SDK checks whether a retry needs to be performed.</span></span> <span data-ttu-id="9e061-275">오류가 *복구 가능한* 경우, SDK는 구성된 재시도 정책을 사용하여 다시 시도를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-275">If the error is *recoverable*, the SDK begins to retry using the configured retry policy.</span></span>

<span data-ttu-id="9e061-276">기본 재시도 정책은 *임의 지터를 사용한 지수적 백오프*이지만 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-276">The default retry policy is *exponential back-off with random jitter*, but it can be configured.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="9e061-277">정책 구성</span><span class="sxs-lookup"><span data-stu-id="9e061-277">Policy configuration</span></span>

<span data-ttu-id="9e061-278">정책 구성은 언어에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-278">Policy configuration differs by language.</span></span> <span data-ttu-id="9e061-279">자세한 내용은 [IoT Hub 재시도 정책 구성](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-279">For more details, see [IoT Hub retry policy configuration](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).</span></span>

### <a name="more-information"></a><span data-ttu-id="9e061-280">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-280">More information</span></span>

* [<span data-ttu-id="9e061-281">IoT Hub 재시도 정책</span><span class="sxs-lookup"><span data-stu-id="9e061-281">IoT Hub retry policy</span></span>](/azure/iot-hub/iot-hub-reliability-features-in-sdks)
* [<span data-ttu-id="9e061-282">IoT Hub 장치 연결 끊김 문제 해결</span><span class="sxs-lookup"><span data-stu-id="9e061-282">Troubleshoot IoT Hub device disconnection</span></span>](/azure/iot-hub/iot-hub-troubleshoot-connectivity)

## <a name="azure-redis-cache"></a><span data-ttu-id="9e061-283">Azure Redis 캐시(영문)</span><span class="sxs-lookup"><span data-stu-id="9e061-283">Azure Redis Cache</span></span>
<span data-ttu-id="9e061-284">Azure Redis Cache는 많이 사용되는 오픈 소스 Redis 캐시에 기반한 캐시 서비스로 데이터 액세스 속도가 빠르고 대기 시간이 짧습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-284">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="9e061-285">이 캐시는 Microsoft에서 관리되어 안전하며 Azure의 모든 응용 프로그램에서 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-285">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="9e061-286">이 섹션의 지침은 StackExchange.Redis 클라이언트를 사용하여 캐시에 액세스하는 지침을 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-286">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="9e061-287">기타 적합한 클라이언트 목록은 [Redis 웹 사이트](https://redis.io/clients)에서 확인할 수 있으며 클라이언트마다 재시도 메커니즘이 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-287">A list of other suitable clients can be found on the [Redis website](https://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="9e061-288">StackExchange.Redis 클라이언트는 단일 연결을 통해 멀티플렉싱을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-288">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="9e061-289">권장되는 사용법은 응용 프로그램 시작 시 클라이언트의 인스턴스를 만들고 캐시에 대한 모든 작업에 이 인스턴스를 사용하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-289">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="9e061-290">따라서 캐시에 한 번만 연결되므로 이 섹션의 모든 지침은 캐시에 액세스하는 각 작업이 아니라 이 초기 연결에 대한 재시도 정책과 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-290">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-291">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-291">Retry mechanism</span></span>
<span data-ttu-id="9e061-292">StackExchange.Redis 클라이언트는 다음과 같이 옵션 집합을 통해 구성되는 연결 관리자 클래스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-292">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, including:</span></span>

- <span data-ttu-id="9e061-293">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="9e061-293">**ConnectRetry**.</span></span> <span data-ttu-id="9e061-294">실패한 캐시 연결을 재시도한 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-294">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="9e061-295">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="9e061-295">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="9e061-296">사용하는 재시도 전략입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-296">The retry strategy to use.</span></span>
- <span data-ttu-id="9e061-297">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="9e061-297">**ConnectTimeout**.</span></span> <span data-ttu-id="9e061-298">최대 대기 시간(밀리초)입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-298">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="9e061-299">정책 구성</span><span class="sxs-lookup"><span data-stu-id="9e061-299">Policy configuration</span></span>
<span data-ttu-id="9e061-300">재시도 정책은 캐시에 연결하기 전에 클라이언트에 대한 옵션을 설정하여 프로그래밍 방식으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-300">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="9e061-301">이 작업은 **ConfigurationOptions** 클래스의 인스턴스를 만들고 해당 속성을 채운 후 **Connect** 메서드에 전달하여 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-301">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="9e061-302">기본 제공 클래스는 임의 재시도 간격을 통해 선형(일정) 지연 및 지수 백오프를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-302">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="9e061-303">**IReconnectRetryPolicy** 인터페이스를 구현하여 사용자 지정 재시도 정책을 만들 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-303">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="9e061-304">다음 예제에서는 지수 백오프를 사용하여 재시도 전략을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-304">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="9e061-305">또는 옵션을 문자열로 지정하고 이를 **Connect** 메서드에 전달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-305">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="9e061-306">**ReconnectRetryPolicy** 속성은 이 방식으로 설정할 수 없고 코드를 통해서만 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-306">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="9e061-307">캐시에 연결할 때 직접 옵션을 지정할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-307">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="9e061-308">자세한 내용은 StackExchange.Redis 설명서의 [Stack Exchange Redis 구성](https://stackexchange.github.io/StackExchange.Redis/Configuration)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-308">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="9e061-309">다음 표에서는 기본 제공 재시도 정책의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-309">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="9e061-310">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="9e061-310">**Context**</span></span> | <span data-ttu-id="9e061-311">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-311">**Setting**</span></span> | <span data-ttu-id="9e061-312">**기본값**</span><span class="sxs-lookup"><span data-stu-id="9e061-312">**Default value**</span></span><br /><span data-ttu-id="9e061-313">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="9e061-313">(v 1.2.2)</span></span> | <span data-ttu-id="9e061-314">**의미**</span><span class="sxs-lookup"><span data-stu-id="9e061-314">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="9e061-315">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="9e061-315">ConfigurationOptions</span></span> |<span data-ttu-id="9e061-316">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="9e061-316">ConnectRetry</span></span><br /><br /><span data-ttu-id="9e061-317">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="9e061-317">ConnectTimeout</span></span><br /><br /><span data-ttu-id="9e061-318">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="9e061-318">SyncTimeout</span></span><br /><br /><span data-ttu-id="9e061-319">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="9e061-319">ReconnectRetryPolicy</span></span> |<span data-ttu-id="9e061-320">3</span><span class="sxs-lookup"><span data-stu-id="9e061-320">3</span></span><br /><br /><span data-ttu-id="9e061-321">최대 5000ms와 SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="9e061-321">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="9e061-322">1000</span><span class="sxs-lookup"><span data-stu-id="9e061-322">1000</span></span><br /><br /><span data-ttu-id="9e061-323">LinearRetry 5000ms</span><span class="sxs-lookup"><span data-stu-id="9e061-323">LinearRetry 5000 ms</span></span> |<span data-ttu-id="9e061-324">초기 연결 작업 중 연결 시도 반복 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-324">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="9e061-325">연결 작업에 대한 제한 시간(ms)입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-325">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="9e061-326">재시도 사이에 지연은 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-326">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="9e061-327">동기 작업을 허용하는 시간(ms)입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-327">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="9e061-328">5000 밀리초마다 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-328">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="9e061-329">동기 작업의 경우 `SyncTimeout`을 종단 간 대기 시간에 추가할 수 있으나 이 값을 너무 낮게 설정하면 과도한 시간 제한이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-329">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="9e061-330">[Azure Redis Cache 문제를 해결하는 방법][redis-cache-troubleshoot]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-330">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="9e061-331">일반적으로 동기 작업보다는 비동기 작업을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-331">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="9e061-332">자세한 내용은 [파이프라인 및 멀티플렉서](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md)(영문)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-332">For more information see [Pipelines and Multiplexers](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="9e061-333">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-333">Retry usage guidance</span></span>
<span data-ttu-id="9e061-334">Azure Redis Cache를 사용하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-334">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="9e061-335">StackExchange Redis 클라이언트는 고유한 재시도를 관리하지만 응용 프로그램이 처음 시작될 때 캐시에 대한 연결을 설정할 때만 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-335">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="9e061-336">연결 제한 시간, 재시도 횟수 및 재시도 사이의 시간을 구성하여 이 연결을 설정할 수 있지만 재시도 정책은 캐시에 대한 작업에 적용되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-336">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="9e061-337">많은 수의 재시도 횟수를 사용하는 대신 원래 데이터 소스에 액세스하여 폴백하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-337">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="9e061-338">원격 분석</span><span class="sxs-lookup"><span data-stu-id="9e061-338">Telemetry</span></span>
<span data-ttu-id="9e061-339">**TextWriter**를 사용하여 다른 작업이 아닌 연결에 대한 정보를 수집할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-339">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="9e061-340">생성되는 출력의 예는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-340">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="9e061-341">예</span><span class="sxs-lookup"><span data-stu-id="9e061-341">Examples</span></span>
<span data-ttu-id="9e061-342">다음 코드 예제에서는 StackExchange.Redis 클라이언트를 초기화할 때 재시도 사이에 일정(선형) 지연을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-342">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="9e061-343">이 예제에서는 **ConfigurationOptions** 인스턴스를 사용하여 구성을 설정하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-343">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="9e061-344">다음 예제에서는 옵션을 문자열로 지정하여 구성을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-344">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="9e061-345">연결 제한 시간은 재시도 간의 지연이 아니라 캐시에 대한 연결을 대기하는 최대 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-345">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="9e061-346">**ReconnectRetryPolicy** 속성은 코드로만 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-346">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="9e061-347">더 많은 예제는 프로젝트 웹 사이트의 [구성](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md)(영문)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-347">For more examples, see [Configuration](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="9e061-348">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-348">More information</span></span>
* [<span data-ttu-id="9e061-349">Redis 웹 사이트</span><span class="sxs-lookup"><span data-stu-id="9e061-349">Redis website</span></span>](https://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="9e061-350">Azure Search</span><span class="sxs-lookup"><span data-stu-id="9e061-350">Azure Search</span></span>
<span data-ttu-id="9e061-351">Azure Search를 사용하면 강력하고 정교한 검색 기능을 웹 사이트 또는 응용 프로그램에 추가하고, 검색 결과를 쉽고 빠르게 조정하며, 풍부하고 미세 조정된 순위 모델을 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-351">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-352">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-352">Retry mechanism</span></span>
<span data-ttu-id="9e061-353">Azure Search SDK의 재시도 동작은 [SearchServiceClient] 및 [SearchIndexClient] 클래스의 `SetRetryPolicy` 메서드에 의해 제어됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-353">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="9e061-354">Azure Search가 5xx 또는 408(요청 시간 초과) 응답을 반환하는 경우 기본 정책은 지수 백오프를 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-354">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="9e061-355">원격 분석</span><span class="sxs-lookup"><span data-stu-id="9e061-355">Telemetry</span></span>
<span data-ttu-id="9e061-356">ETW를 사용하거나 사용자 지정 추적 공급자를 등록하는 추적입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-356">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="9e061-357">자세한 내용은 [AutoRest 설명서][autorest]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-357">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="9e061-358">Service Bus</span><span class="sxs-lookup"><span data-stu-id="9e061-358">Service Bus</span></span>
<span data-ttu-id="9e061-359">Service Bus는 클라우드에서 호스트되는지 온-프레미스에서 호스트되는지에 관계없이 향상된 확장성 및 복원력으로 느슨하게 결합된 메시지 교환을 응용 프로그램의 구성 요소에 제공하는 클라우드 메시징 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-359">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-360">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-360">Retry mechanism</span></span>
<span data-ttu-id="9e061-361">Service Bus는 [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) 기본 클래스의 구현을 사용하여 재시도를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-361">Service Bus implements retries using implementations of the [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) base class.</span></span> <span data-ttu-id="9e061-362">모든 Service Bus 클라이언트는 **RetryPolicy** 기본 클래스의 구현 중 하나로 설정할 수 있는 **RetryPolicy** 속성을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-362">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="9e061-363">기본 제공 구현은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-363">The built-in implementations are:</span></span>

* <span data-ttu-id="9e061-364">[RetryExponential 클래스](/dotnet/api/microsoft.servicebus.retryexponential).</span><span class="sxs-lookup"><span data-stu-id="9e061-364">The [RetryExponential Class](/dotnet/api/microsoft.servicebus.retryexponential).</span></span> <span data-ttu-id="9e061-365">이 클래스는 백오프 간격, 재시도 횟수 및 작업이 완료되는 총 시간을 제한하는 데 사용되는 **TerminationTimeBuffer** 속성을 제어하는 속성을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-365">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="9e061-366">[NoRetry 클래스](/dotnet/api/microsoft.servicebus.noretry).</span><span class="sxs-lookup"><span data-stu-id="9e061-366">The [NoRetry Class](/dotnet/api/microsoft.servicebus.noretry).</span></span> <span data-ttu-id="9e061-367">이 클래스는 재시도가 일괄 처리 또는 다단계 작업의 일부로 다른 프로세스에서 관리되는 경우처럼 Service Bus API 수준의 재시도가 필요하지 않은 경우에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-367">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="9e061-368">Service Bus 작업은 [Service Bus 메시징 예외](/azure/service-bus-messaging/service-bus-messaging-exceptions)에 나열된 일련의 예외를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-368">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="9e061-369">이 목록에서는 작업 재시도가 적절한지 여부를 나타내는 예외에 대한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-369">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="9e061-370">예를 들어 **ServerBusyException** 은 클라이언트가 일정 기간 동안 대기한 후 작업을 재시도해야 함을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-370">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="9e061-371">또한 **ServerBusyException** 이 발생하면 Service Bus가 다른 모드로 전환되어 추가 10초의 지연이 계산된 재시도 지연에 추가될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-371">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="9e061-372">이 모드는 잠시 후 다시 설정됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-372">This mode is reset after a short period.</span></span>

<span data-ttu-id="9e061-373">Service Bus에서 반환된 예외는 클라이언트가 작업을 재시도해야 하는지 여부를 나타내는 **IsTransient** 속성을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-373">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="9e061-374">기본 제공 **RetryExponential** 정책은 모든 Service Bus 예외에 대한 기본 클래스인 **MessagingException** 클래스의 **IsTransient** 속성을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-374">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="9e061-375">**RetryPolicy** 기본 클래스의 사용자 지정 구현을 만드는 경우 예외 유형과 **IsTransient** 속성의 조합을 사용하여 재시도 작업을 보다 세부적으로 제어할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-375">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="9e061-376">예를 들어 **QuotaExceededException**을 검색하고 조치를 취하여 메시지 보내기를 재시도하기 전에 큐를 비울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-376">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="9e061-377">정책 구성</span><span class="sxs-lookup"><span data-stu-id="9e061-377">Policy configuration</span></span>
<span data-ttu-id="9e061-378">재시도 정책은 프로그래밍 방식으로 설정되며 **NamespaceManager** 및 **MessagingFactory**에 대한 기본 정책으로 설정하거나 각 메시징 클라이언트에 대해 개별적으로 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-378">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="9e061-379">메시징 세션에 대해 기본 재시도 정책을 설정하려면 **NamespaceManager**의 **RetryPolicy**를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-379">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="9e061-380">메시징 팩터리에서 만든 모든 클라이언트에 대해 기본 재시도 정책을 설정하려면 **MessagingFactory**의 **RetryPolicy**를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-380">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="9e061-381">메시징 클라이언트에 대한 재시도 정책을 설정하거나 기본 정책을 재정의하려면 필요한 정책 클래스의 인스턴스를 사용하여 **RetryPolicy** 속성을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-381">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="9e061-382">재시도 정책은 개별 작업 수준에서 설정할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-382">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="9e061-383">메시징 클라이언트에 대한 모든 작업에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-383">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="9e061-384">다음 표에서는 기본 제공 재시도 정책의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-384">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="9e061-385">설정</span><span class="sxs-lookup"><span data-stu-id="9e061-385">Setting</span></span> | <span data-ttu-id="9e061-386">기본값</span><span class="sxs-lookup"><span data-stu-id="9e061-386">Default value</span></span> | <span data-ttu-id="9e061-387">의미</span><span class="sxs-lookup"><span data-stu-id="9e061-387">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="9e061-388">정책</span><span class="sxs-lookup"><span data-stu-id="9e061-388">Policy</span></span> | <span data-ttu-id="9e061-389">지수</span><span class="sxs-lookup"><span data-stu-id="9e061-389">Exponential</span></span> | <span data-ttu-id="9e061-390">지수적 백오프.</span><span class="sxs-lookup"><span data-stu-id="9e061-390">Exponential back-off.</span></span> |
| <span data-ttu-id="9e061-391">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-391">MinimalBackoff</span></span> | <span data-ttu-id="9e061-392">0</span><span class="sxs-lookup"><span data-stu-id="9e061-392">0</span></span> | <span data-ttu-id="9e061-393">최소 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-393">Minimum back-off interval.</span></span> <span data-ttu-id="9e061-394">deltaBackoff에서 계산된 재시도 간격에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-394">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="9e061-395">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-395">MaximumBackoff</span></span> | <span data-ttu-id="9e061-396">30초</span><span class="sxs-lookup"><span data-stu-id="9e061-396">30 seconds</span></span> | <span data-ttu-id="9e061-397">최대 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-397">Maximum back-off interval.</span></span> <span data-ttu-id="9e061-398">MaximumBackoff는 계산된 재시도 간격이 MaxBackoff보다 큰 경우 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-398">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="9e061-399">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-399">DeltaBackoff</span></span> | <span data-ttu-id="9e061-400">3초</span><span class="sxs-lookup"><span data-stu-id="9e061-400">3 seconds</span></span> | <span data-ttu-id="9e061-401">재시도 사이의 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-401">Back-off interval between retries.</span></span> <span data-ttu-id="9e061-402">후속 재시도에 이 시간 범위의 배수가 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-402">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="9e061-403">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="9e061-403">TimeBuffer</span></span> | <span data-ttu-id="9e061-404">5초</span><span class="sxs-lookup"><span data-stu-id="9e061-404">5 seconds</span></span> | <span data-ttu-id="9e061-405">재시도와 연결된 종료 시간 버퍼입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-405">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="9e061-406">남은 시간이 TimeBuffer보다 적으면 재시도가 중지됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-406">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="9e061-407">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="9e061-407">MaxRetryCount</span></span> | <span data-ttu-id="9e061-408">10</span><span class="sxs-lookup"><span data-stu-id="9e061-408">10</span></span> | <span data-ttu-id="9e061-409">최대 재시도 수</span><span class="sxs-lookup"><span data-stu-id="9e061-409">The maximum number of retries.</span></span> |
| <span data-ttu-id="9e061-410">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="9e061-410">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="9e061-411">10초</span><span class="sxs-lookup"><span data-stu-id="9e061-411">10 seconds</span></span> | <span data-ttu-id="9e061-412">마지막으로 발생한 예외가 **ServerBusyException**인 경우 이 값이 계산된 재시도 간격에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-412">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="9e061-413">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-413">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="9e061-414">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-414">Retry usage guidance</span></span>
<span data-ttu-id="9e061-415">Service Bus를 사용하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-415">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="9e061-416">기본 제공 **RetryExponential** 구현을 사용하는 경우 정책이 서버 사용 중 예외에 반응하고 적절한 재시도 모드로 자동으로 전환되므로 폴백 작업을 구현하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-416">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="9e061-417">Service Bus는 기본 네임스페이스의 큐가 실패할 경우 별도의 네임스페이스에 있는 백업 큐로 자동 장애 조치(failover)를 구현하는 쌍을 이루는 네임스페이스라는 기능을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-417">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="9e061-418">기본 큐가 복구되면 보조 큐의 메시지를 다시 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-418">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="9e061-419">이 기능은 일시적인 오류를 해결하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-419">This feature helps to address transient failures.</span></span> <span data-ttu-id="9e061-420">자세한 내용은 [비동기 메시징 패턴 및 고가용성](/azure/service-bus-messaging/service-bus-async-messaging)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-420">For more information, see [Asynchronous Messaging Patterns and High Availability](/azure/service-bus-messaging/service-bus-async-messaging).</span></span>

<span data-ttu-id="9e061-421">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-421">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="9e061-422">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-422">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="9e061-423">Context</span><span class="sxs-lookup"><span data-stu-id="9e061-423">Context</span></span> | <span data-ttu-id="9e061-424">예제 최대 대기 시간</span><span class="sxs-lookup"><span data-stu-id="9e061-424">Example maximum latency</span></span> | <span data-ttu-id="9e061-425">다시 시도 정책</span><span class="sxs-lookup"><span data-stu-id="9e061-425">Retry policy</span></span> | <span data-ttu-id="9e061-426">설정</span><span class="sxs-lookup"><span data-stu-id="9e061-426">Settings</span></span> | <span data-ttu-id="9e061-427">작동 방법</span><span class="sxs-lookup"><span data-stu-id="9e061-427">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="9e061-428">대화형, UI 또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-428">Interactive, UI, or foreground</span></span> | <span data-ttu-id="9e061-429">2초\*</span><span class="sxs-lookup"><span data-stu-id="9e061-429">2 seconds\*</span></span>  | <span data-ttu-id="9e061-430">지수</span><span class="sxs-lookup"><span data-stu-id="9e061-430">Exponential</span></span> | <span data-ttu-id="9e061-431">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="9e061-431">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="9e061-432">MaximumBackoff = 30 sec.</span><span class="sxs-lookup"><span data-stu-id="9e061-432">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="9e061-433">DeltaBackoff = 300 msec.</span><span class="sxs-lookup"><span data-stu-id="9e061-433">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="9e061-434">TimeBuffer = 300 msec.</span><span class="sxs-lookup"><span data-stu-id="9e061-434">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="9e061-435">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="9e061-435">MaxRetryCount = 2</span></span> | <span data-ttu-id="9e061-436">Attempt 1: Delay 0 sec.</span><span class="sxs-lookup"><span data-stu-id="9e061-436">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="9e061-437">Attempt 2: Delay ~300 msec.</span><span class="sxs-lookup"><span data-stu-id="9e061-437">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="9e061-438">Attempt 3: Delay ~900 msec.</span><span class="sxs-lookup"><span data-stu-id="9e061-438">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="9e061-439">백그라운드 또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="9e061-439">Background or batch</span></span> | <span data-ttu-id="9e061-440">30초</span><span class="sxs-lookup"><span data-stu-id="9e061-440">30 seconds</span></span> | <span data-ttu-id="9e061-441">지수</span><span class="sxs-lookup"><span data-stu-id="9e061-441">Exponential</span></span> | <span data-ttu-id="9e061-442">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="9e061-442">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="9e061-443">MaximumBackoff = 30 sec.</span><span class="sxs-lookup"><span data-stu-id="9e061-443">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="9e061-444">DeltaBackoff = 1.75 sec.</span><span class="sxs-lookup"><span data-stu-id="9e061-444">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="9e061-445">TimeBuffer = 5 sec.</span><span class="sxs-lookup"><span data-stu-id="9e061-445">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="9e061-446">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="9e061-446">MaxRetryCount = 3</span></span> | <span data-ttu-id="9e061-447">Attempt 1: Delay ~1 sec.</span><span class="sxs-lookup"><span data-stu-id="9e061-447">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="9e061-448">Attempt 2: Delay ~3 sec.</span><span class="sxs-lookup"><span data-stu-id="9e061-448">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="9e061-449">Attempt 3: Delay ~6 msec.</span><span class="sxs-lookup"><span data-stu-id="9e061-449">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="9e061-450">Attempt 4: Delay ~13 msec.</span><span class="sxs-lookup"><span data-stu-id="9e061-450">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="9e061-451">\* 서버 사용 중 응답을 받은 경우 추가된 추가 지연을 포함하지 않음</span><span class="sxs-lookup"><span data-stu-id="9e061-451">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="9e061-452">원격 분석</span><span class="sxs-lookup"><span data-stu-id="9e061-452">Telemetry</span></span>
<span data-ttu-id="9e061-453">Service Bus는 **EventSource**를 사용하여 재시도를 ETW 이벤트로 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-453">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="9e061-454">이벤트를 캡처하여 성능 뷰어에서 보거나 적합한 대상 로그에 기록하려면 **EventListener**를 이벤트 소스에 연결해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-454">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="9e061-455">재시도 이벤트는 다음과 같은 형식입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-455">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="9e061-456">예</span><span class="sxs-lookup"><span data-stu-id="9e061-456">Examples</span></span>
<span data-ttu-id="9e061-457">다음 코드 예제에서는 다음에 대해 재시도 정책을 설정하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-457">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="9e061-458">네임스페이스 관리자.</span><span class="sxs-lookup"><span data-stu-id="9e061-458">A namespace manager.</span></span> <span data-ttu-id="9e061-459">정책은 해당 관리자의 모든 작업에 적용되며 개별 작업에 대해 재정의할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-459">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="9e061-460">메시징 팩터리.</span><span class="sxs-lookup"><span data-stu-id="9e061-460">A messaging factory.</span></span> <span data-ttu-id="9e061-461">정책은 해당 팩터리에서 만든 모든 클라이언트에 적용되며 개별 클라이언트를 만드는 경우 재정의할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-461">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="9e061-462">개별 메시징 클라이언트.</span><span class="sxs-lookup"><span data-stu-id="9e061-462">An individual messaging client.</span></span> <span data-ttu-id="9e061-463">클라이언트를 만든 후 해당 클라이언트에 대한 재시도 정책을 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-463">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="9e061-464">정책은 해당 클라이언트의 모든 작업에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-464">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="9e061-465">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-465">More information</span></span>
* [<span data-ttu-id="9e061-466">비동기 메시징 패턴 및 고가용성</span><span class="sxs-lookup"><span data-stu-id="9e061-466">Asynchronous Messaging Patterns and High Availability</span></span>](/azure/service-bus-messaging/service-bus-async-messaging)

## <a name="service-fabric"></a><span data-ttu-id="9e061-467">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="9e061-467">Service Fabric</span></span>

<span data-ttu-id="9e061-468">Service Fabric 클러스터에서 신뢰할 수 있는 서비스를 배포하면 이 문서에서 논의된 대부분의 가능한 일시적 오류를 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-468">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="9e061-469">그러나 일부 일시적 오류는 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-469">Some transient faults are still possible, however.</span></span> <span data-ttu-id="9e061-470">예를 들어 명명 서비스가 변경을 전달하는 도중에 요청을 받으면 예외가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-470">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="9e061-471">같은 요청을 100밀리초 후에 받으면 성공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-471">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="9e061-472">Service Fabric은 내부적으로 이런 종류의 일시적 오류를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-472">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="9e061-473">서비스를 설정할 때 `OperationRetrySettings` 클래스를 사용하여 일부 설정을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-473">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="9e061-474">다음은 예를 보여 주는 코드입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-474">The following code shows an example.</span></span> <span data-ttu-id="9e061-475">이것은 대부분의 경우 불필요하며 기본 설정으로 충분합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-475">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a><span data-ttu-id="9e061-476">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-476">More information</span></span>

* [<span data-ttu-id="9e061-477">원격 예외 처리</span><span class="sxs-lookup"><span data-stu-id="9e061-477">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="9e061-478">ADO.NET을 사용하는 SQL Database</span><span class="sxs-lookup"><span data-stu-id="9e061-478">SQL Database using ADO.NET</span></span>
<span data-ttu-id="9e061-479">SQL Database는 다양한 크기와 표준(공유) 및 프리미엄(비공유) 서비스로 사용할 수 있는 호스트된 SQL Database입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-479">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-480">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-480">Retry mechanism</span></span>
<span data-ttu-id="9e061-481">SQL Database는 ADO.NET을 사용하여 액세스하는 경우 재시도에 대한 기본 제공 지원을 제공하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-481">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="9e061-482">그러나 요청에서 반환된 코드를 사용하여 요청이 실패한 이유를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-482">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="9e061-483">SQL Database 제한에 대한 자세한 내용은 [Azure SQL Database 리소스 제한](/azure/sql-database/sql-database-resource-limits)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-483">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="9e061-484">관련 오류 코드 목록은 [SQL Database 클라이언트 응용 프로그램에 대한 SQL 오류 코드](/azure/sql-database/sql-database-develop-error-messages)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-484">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="9e061-485">SQL Database에 대한 재시도 구현에 Polly 라이브러리를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-485">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="9e061-486">[Polly를 통한 일시적인 오류 처리](#transient-fault-handling-with-polly)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-486">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="9e061-487">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-487">Retry usage guidance</span></span>
<span data-ttu-id="9e061-488">ADO.NET을 사용하는 SQL Database에 액세스하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-488">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="9e061-489">적절한 서비스 옵션(공유 또는 프리미엄)을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-489">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="9e061-490">공유 인스턴스는 공유 서버의 다른 테넌트에서 사용되기 때문에 일반적인 연결 지연 및 제한보다 더 길게 영향을 받을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-490">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="9e061-491">예측 가능한 성능 및 대기 시간이 짧은 안정적인 작업이 필요한 경우 프리미엄 옵션을 선택하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-491">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="9e061-492">적절한 수준 또는 범위에서 재시도를 수행하여 데이터 불일치를 발생시키는 비멱등 작업을 방지해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-492">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="9e061-493">이상적으로는 불일치를 발생시키지 않고 반복할 수 있도록 모든 작업이 멱등이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-493">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="9e061-494">그렇지 않으면 한 작업이 실패할 경우 모든 관련 변경 내용이 실행 취소되도록 하는 수준 또는 범위(예: 트랜잭션 범위 내)에서 재시도를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-494">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="9e061-495">자세한 내용은 [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적인 오류 처리](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee)(영문)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-495">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="9e061-496">고정 간격 전략은 매우 짧은 간격에 몇 번의 재시도만 수행되는 대화형 시나리오를 제외하고 Azure SQL Database에서 사용하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-496">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="9e061-497">대신 대부분의 시나리오에 지수 백오프 전략을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-497">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="9e061-498">연결을 정의할 때 연결 및 명령 제한 시간에 적합한 값을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-498">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="9e061-499">제한 시간이 너무 짧으면 데이터베이스가 사용 중일 때 중간에 연결 오류가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-499">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="9e061-500">제한 시간이 너무 길면 실패한 연결을 검색할 때까지 너무 오래 대기하여 재시도 논리가 올바르게 작동하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-500">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="9e061-501">제한 시간 값은 종단 간 대기 시간의 구성 요소이지므로 모든 재시도에 대한 재시도 정책에 지정된 재시도 지연에 효과적으로 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-501">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="9e061-502">지수 백오프 재시도 논리를 사용하는 경우에도 특정 횟수의 재시도 후에는 연결을 닫고 새 연결에서 작업을 재시도합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-502">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="9e061-503">동일한 연결에서 동일한 작업을 여러 번 재시도하면 연결 문제에 영향을 주는 요인이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-503">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="9e061-504">이 기술에 대한 예제는 [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적인 오류 처리](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)(영문)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-504">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="9e061-505">연결 풀링이 사용 중(기본값)인 경우 연결을 닫았다가 다시 연 후에도 풀에서 동일한 연결을 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-505">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="9e061-506">이런 경우 해결하는 방법은 **SqlConnection** 클래스의 **ClearPool** 메서드를 호출하여 연결을 재사용할 수 없음으로 표시하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-506">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="9e061-507">그러나 이 방법은 여러 번의 연결 시도가 실패한 후에만 수행하고 오류가 발생한 연결과 관련된 SQL 제한 시간(오류 코드 -2)과 같은 특정 클래스의 일시적인 오류가 발생하는 경우에만 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-507">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="9e061-508">데이터 액세스 코드가 **TransactionScope** 인스턴스로 시작된 트랜잭션을 사용하는 경우 재시도 논리에서 연결을 다시 열고 새 트랜잭션 범위를 시작해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-508">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="9e061-509">이러한 이유로 재시도 가능한 코드 블록은 트랜잭션의 전체 범위를 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-509">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="9e061-510">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-510">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="9e061-511">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-511">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="9e061-512">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="9e061-512">**Context**</span></span> | <span data-ttu-id="9e061-513">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="9e061-513">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="9e061-514">**재시도 전략**</span><span class="sxs-lookup"><span data-stu-id="9e061-514">**Retry strategy**</span></span> | <span data-ttu-id="9e061-515">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-515">**Settings**</span></span> | <span data-ttu-id="9e061-516">**값**</span><span class="sxs-lookup"><span data-stu-id="9e061-516">**Values**</span></span> | <span data-ttu-id="9e061-517">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="9e061-517">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9e061-518">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="9e061-518">Interactive, UI,</span></span><br /><span data-ttu-id="9e061-519">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-519">or foreground</span></span> |<span data-ttu-id="9e061-520">2초</span><span class="sxs-lookup"><span data-stu-id="9e061-520">2 sec</span></span> |<span data-ttu-id="9e061-521">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="9e061-521">FixedInterval</span></span> |<span data-ttu-id="9e061-522">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="9e061-522">Retry count</span></span><br /><span data-ttu-id="9e061-523">재시도 간격</span><span class="sxs-lookup"><span data-stu-id="9e061-523">Retry interval</span></span><br /><span data-ttu-id="9e061-524">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="9e061-524">First fast retry</span></span> |<span data-ttu-id="9e061-525">3</span><span class="sxs-lookup"><span data-stu-id="9e061-525">3</span></span><br /><span data-ttu-id="9e061-526">500ms</span><span class="sxs-lookup"><span data-stu-id="9e061-526">500 ms</span></span><br /><span data-ttu-id="9e061-527">true</span><span class="sxs-lookup"><span data-stu-id="9e061-527">true</span></span> |<span data-ttu-id="9e061-528">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-528">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="9e061-529">시도 2 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-529">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="9e061-530">시도 3 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-530">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="9e061-531">백그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-531">Background</span></span><br /><span data-ttu-id="9e061-532">또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="9e061-532">or batch</span></span> |<span data-ttu-id="9e061-533">30초</span><span class="sxs-lookup"><span data-stu-id="9e061-533">30 sec</span></span> |<span data-ttu-id="9e061-534">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-534">ExponentialBackoff</span></span> |<span data-ttu-id="9e061-535">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="9e061-535">Retry count</span></span><br /><span data-ttu-id="9e061-536">최소 백오프</span><span class="sxs-lookup"><span data-stu-id="9e061-536">Min back-off</span></span><br /><span data-ttu-id="9e061-537">최대 백오프</span><span class="sxs-lookup"><span data-stu-id="9e061-537">Max back-off</span></span><br /><span data-ttu-id="9e061-538">델타 백오프</span><span class="sxs-lookup"><span data-stu-id="9e061-538">Delta back-off</span></span><br /><span data-ttu-id="9e061-539">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="9e061-539">First fast retry</span></span> |<span data-ttu-id="9e061-540">5</span><span class="sxs-lookup"><span data-stu-id="9e061-540">5</span></span><br /><span data-ttu-id="9e061-541">0초</span><span class="sxs-lookup"><span data-stu-id="9e061-541">0 sec</span></span><br /><span data-ttu-id="9e061-542">60초</span><span class="sxs-lookup"><span data-stu-id="9e061-542">60 sec</span></span><br /><span data-ttu-id="9e061-543">2초</span><span class="sxs-lookup"><span data-stu-id="9e061-543">2 sec</span></span><br /><span data-ttu-id="9e061-544">false</span><span class="sxs-lookup"><span data-stu-id="9e061-544">false</span></span> |<span data-ttu-id="9e061-545">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-545">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="9e061-546">시도 2 - ~2초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-546">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="9e061-547">시도 3 - ~6초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-547">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="9e061-548">시도 4 - ~14초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-548">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="9e061-549">시도 5 - ~30초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-549">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="9e061-550">종단 간 대기 시간 대상은 서비스에 대한 연결의 기본 제한 시간을 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-550">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="9e061-551">연결 제한 시간을 더 길게 지정하면 종단 간 대기 시간이 모든 재시도에 대해 이 추가 시간만큼 확장됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-551">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="9e061-552">예</span><span class="sxs-lookup"><span data-stu-id="9e061-552">Examples</span></span>
<span data-ttu-id="9e061-553">이 섹션에서는 `Policy` 클래스에 구성된 재시도 정책 집합을 사용하는 Azure SQL Database에 Polly를 사용하여 액세스하는 방법을 보여 줍니다. </span><span class="sxs-lookup"><span data-stu-id="9e061-553">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="9e061-554">다음 코드는 지수 백오프를 통해 `ExecuteAsync`를 호출하는 `SqlCommand` 클래스의 확장 메서드를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-554">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

<span data-ttu-id="9e061-555">이 비동기 확장 메서드는 다음과 같이 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-555">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="9e061-556">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-556">More information</span></span>
* [<span data-ttu-id="9e061-557">클라우드 서비스의 기본 데이터 액세스 계층 – 일시적인 오류 처리(영문)</span><span class="sxs-lookup"><span data-stu-id="9e061-557">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="9e061-558">SQL Database를 최대한 활용하기 위한 일반적인 지침은 [Azure SQL Database 성능 및 탄력성 가이드](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-558">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="9e061-559">Entity Framework 6을 사용하는 SQL Database</span><span class="sxs-lookup"><span data-stu-id="9e061-559">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="9e061-560">SQL Database는 다양한 크기와 표준(공유) 및 프리미엄(비공유) 서비스로 사용할 수 있는 호스트된 SQL Database입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-560">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="9e061-561">Entity Framework는 .NET 개발자가 도메인별 개체를 사용하여 관계형 데이터로 작업할 수 있는 개체 관계형 매퍼입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-561">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="9e061-562">여기서는 개발자가 일반적으로 작성해야 하는 대부분의 데이터 액세스 코드가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-562">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-563">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-563">Retry mechanism</span></span>
<span data-ttu-id="9e061-564">재시도는 [연결 복원/재시도 논리](/ef/ef6/fundamentals/connection-resiliency/retry-logic)(영문)라는 메커니즘을 통해 Entity Framework 6.0 이상을 사용하는 SQL Database에 액세스할 때 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-564">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span> <span data-ttu-id="9e061-565">재시도 메커니즘의 주요 기능은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-565">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="9e061-566">기본 추상화는 **IDbExecutionStrategy** 인터페이스입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-566">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="9e061-567">이 인터페이스는 다음을 수행합니다.:</span><span class="sxs-lookup"><span data-stu-id="9e061-567">This interface:</span></span>
  * <span data-ttu-id="9e061-568">동기 및 비동기 **Execute**\* 메서드를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-568">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="9e061-569">직접 사용하거나 데이터베이스 컨텍스트에서 기본 전략으로 구성하거나, 공급자 이름에 매핑하거나, 공급자 이름 및 서버 이름에 매핑할 수 있는 클래스를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-569">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="9e061-570">컨텍스트에서 구성할 경우 재시도는 지정된 컨텍스트 작업에 대해 여러 개가 있을 수 있는 개별 데이터베이스 작업 수준에서 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-570">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="9e061-571">실패한 연결을 재시도할 시기 및 방법을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-571">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="9e061-572">여기에는 다음과 같은 **IDbExecutionStrategy** 인터페이스의 여러 가지 기본 제공 구현이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-572">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="9e061-573">기본값 - 재시도하지 않음.</span><span class="sxs-lookup"><span data-stu-id="9e061-573">Default - no retrying.</span></span>
  * <span data-ttu-id="9e061-574">SQL Database(자동)에 대한 기본값 - 재시도하지 않지만 예외를 검사하고 SQL Database 전략을 사용하는 제안으로 래핑합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-574">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="9e061-575">SQL Database에 대한 기본값 - 지수(기본 클래스에서 상속됨) 및 SQL Database 검색 논리.</span><span class="sxs-lookup"><span data-stu-id="9e061-575">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="9e061-576">불규칙을 포함하는 지수 백오프 전략을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-576">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="9e061-577">기본 제공 재시도 클래스는 상태 저장 클래스이며 스레드로부터 안전하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-577">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="9e061-578">그러나 현재 작업이 완료된 후 다시 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-578">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="9e061-579">지정된 재시도 횟수를 초과하는 경우 결과는 새 예외에 래핑됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-579">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="9e061-580">현재 예외를 버블 업하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-580">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="9e061-581">정책 구성</span><span class="sxs-lookup"><span data-stu-id="9e061-581">Policy configuration</span></span>
<span data-ttu-id="9e061-582">재시도는 Entity Framework 6.0 이상을 사용하는 SQL Database에 액세스할 때 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-582">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="9e061-583">재시도 정책은 프로그래밍 방식으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-583">Retry policies are configured programmatically.</span></span> <span data-ttu-id="9e061-584">구성은 작업 단위로 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-584">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="9e061-585">컨텍스트에서 기본값으로 전략을 구성하는 경우 필요에 따라 새 전략을 만드는 함수를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-585">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="9e061-586">다음 코드에서는 **DbConfiguration** 기본 클래스를 확장하는 재시도 구성 클래스를 만드는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-586">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="9e061-587">그런 다음 응용 프로그램이 시작될 때 **DbConfiguration** 인스턴스의 **SetConfiguration** 메서드를 사용하여 모든 작업에 대해 기본 재시도 전략으로 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-587">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="9e061-588">기본적으로 EF는 구성 클래스를 자동으로 검색하고 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-588">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="9e061-589">**DbConfigurationType** 특성으로 컨텍스트 클래스에 주석을 추가하여 컨텍스트에 대한 재시도 구성 클래스를 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-589">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="9e061-590">그러나 구성 클래스가 하나만 있는 경우 EF는 컨텍스트에 주석을 추가할 필요 없이 해당 클래스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-590">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="9e061-591">특정 작업에 다른 재시도 전략을 사용하거나 특정 작업에 대해 재시도를 사용하지 않도록 설정해야 하는 경우 **CallContext**에서 플래그를 설정하여 전략을 일시 중단하거나 교환할 수 있도록 하는 구성 클래스를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-591">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="9e061-592">구성 클래스는 이 플래그를 사용하여 전략을 전환하거나, 제공된 전략을 사용하지 않도록 설정하고 기본 전략을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-592">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="9e061-593">자세한 내용은 [실행 전략 일시 중단](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy)(EF6 이상)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-593">For more information, see [Suspend Execution Strategy](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (EF6 onwards).</span></span>

<span data-ttu-id="9e061-594">개별 작업에 특정 재시도 전략을 사용하기 위한 다른 기술은 필요한 전략 클래스의 인스턴스를 만들고 매개 변수를 통해 원하는 설정을 제공하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-594">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="9e061-595">그런 다음 **ExecuteAsync** 메서드를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-595">You then invoke its **ExecuteAsync** method.</span></span>

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

<span data-ttu-id="9e061-596">**DbConfiguration** 클래스를 사용하는 가장 간단한 방법은 **DbContext** 클래스와 동일한 어셈블리에 배치하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-596">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="9e061-597">그러나 다양한 대화형 및 백그라운드 재시도 전략과 같이 서로 다른 시나리오에서 동일한 컨텍스트 필요한 경우에는 적합하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-597">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="9e061-598">서로 다른 컨텍스트가 별도의 AppDomain에서 실행되는 경우 구성 파일에서 구성 클래스를 지정하는 데 기본 제공 지원을 사용하거나 코드를 사용하여 명시적으로 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-598">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="9e061-599">다른 컨텍스트가 동일한 AppDomain에서 실행되어야 하는 경우 사용자 지정 솔루션이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-599">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="9e061-600">자세한 내용은 [코드 기반 구성](/ef/ef6/fundamentals/configuring/code-based)(EF6 이상)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-600">For more information, see [Code-Based Configuration](/ef/ef6/fundamentals/configuring/code-based) (EF6 onwards).</span></span>

<span data-ttu-id="9e061-601">다음 표에서는 EF6을 사용하는 경우 기본 제공 재시도 정책의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-601">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="9e061-602">설정</span><span class="sxs-lookup"><span data-stu-id="9e061-602">Setting</span></span> | <span data-ttu-id="9e061-603">기본값</span><span class="sxs-lookup"><span data-stu-id="9e061-603">Default value</span></span> | <span data-ttu-id="9e061-604">의미</span><span class="sxs-lookup"><span data-stu-id="9e061-604">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="9e061-605">정책</span><span class="sxs-lookup"><span data-stu-id="9e061-605">Policy</span></span> | <span data-ttu-id="9e061-606">지수</span><span class="sxs-lookup"><span data-stu-id="9e061-606">Exponential</span></span> | <span data-ttu-id="9e061-607">지수적 백오프.</span><span class="sxs-lookup"><span data-stu-id="9e061-607">Exponential back-off.</span></span> |
| <span data-ttu-id="9e061-608">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="9e061-608">MaxRetryCount</span></span> | <span data-ttu-id="9e061-609">5</span><span class="sxs-lookup"><span data-stu-id="9e061-609">5</span></span> | <span data-ttu-id="9e061-610">최대 재시도 수</span><span class="sxs-lookup"><span data-stu-id="9e061-610">The maximum number of retries.</span></span> |
| <span data-ttu-id="9e061-611">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="9e061-611">MaxDelay</span></span> | <span data-ttu-id="9e061-612">30초</span><span class="sxs-lookup"><span data-stu-id="9e061-612">30 seconds</span></span> | <span data-ttu-id="9e061-613">재시도 사이의 최대 지연.</span><span class="sxs-lookup"><span data-stu-id="9e061-613">The maximum delay between retries.</span></span> <span data-ttu-id="9e061-614">이 값은 일련의 지연이 계산되는 방식에는 영향이 없으며</span><span class="sxs-lookup"><span data-stu-id="9e061-614">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="9e061-615">상한 값만 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-615">It only defines an upper bound.</span></span> |
| <span data-ttu-id="9e061-616">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="9e061-616">DefaultCoefficient</span></span> | <span data-ttu-id="9e061-617">1초</span><span class="sxs-lookup"><span data-stu-id="9e061-617">1 second</span></span> | <span data-ttu-id="9e061-618">지수 백오프 계산을 위한 계수.</span><span class="sxs-lookup"><span data-stu-id="9e061-618">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="9e061-619">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-619">This value cannot be changed.</span></span> |
| <span data-ttu-id="9e061-620">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="9e061-620">DefaultRandomFactor</span></span> | <span data-ttu-id="9e061-621">1.1</span><span class="sxs-lookup"><span data-stu-id="9e061-621">1.1</span></span> | <span data-ttu-id="9e061-622">각 항목에 대해 임의 지연을 추가하는 데 사용되는 승수입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-622">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="9e061-623">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-623">This value cannot be changed.</span></span> |
| <span data-ttu-id="9e061-624">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="9e061-624">DefaultExponentialBase</span></span> | <span data-ttu-id="9e061-625">2</span><span class="sxs-lookup"><span data-stu-id="9e061-625">2</span></span> | <span data-ttu-id="9e061-626">다음 지연을 계산하는 데 사용되는 승수입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-626">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="9e061-627">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-627">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="9e061-628">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-628">Retry usage guidance</span></span>
<span data-ttu-id="9e061-629">EF6을 사용하는 SQL Database에 액세스하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-629">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="9e061-630">적절한 서비스 옵션(공유 또는 프리미엄)을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-630">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="9e061-631">공유 인스턴스는 공유 서버의 다른 테넌트에서 사용되기 때문에 일반적인 연결 지연 및 제한보다 더 길게 영향을 받을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-631">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="9e061-632">예측 가능한 성능 및 대기 시간이 짧은 안정적인 작업이 필요한 경우 프리미엄 옵션을 선택하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-632">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="9e061-633">Azure SQL Database에는 고정 간격 전략을 사용하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-633">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="9e061-634">대신 서비스가 오버로드될 수 있고 더 긴 지연으로 인해 복구에 더 많은 시간이 필요할 수 있으므로 지수 백오프 전략을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-634">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="9e061-635">연결을 정의할 때 연결 및 명령 제한 시간에 적합한 값을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-635">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="9e061-636">제한 시간은 비즈니스 논리 디자인 및 테스트를 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-636">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="9e061-637">시간에 따라 데이터의 양이나 비즈니스 프로세스가 변경되므로 이 값을 수정해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-637">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="9e061-638">제한 시간이 너무 짧으면 데이터베이스가 사용 중일 때 중간에 연결 오류가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-638">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="9e061-639">제한 시간이 너무 길면 실패한 연결을 검색할 때까지 너무 오래 대기하여 재시도 논리가 올바르게 작동하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-639">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="9e061-640">제한 시간 값은 종단 간 대기 시간의 구성 요소이지만 컨텍스트를 저장할 때 실행될 명령 수를 쉽게 확인할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-640">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="9e061-641">**DbContext** 인스턴스의 **CommandTimeout** 속성을 설정하여 기본 제한 시간을 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-641">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="9e061-642">Entity Framework는 구성 파일에 정의된 재시도 구성을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-642">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="9e061-643">그러나 Azure에서 최대한의 유연성을 제공하려면 응용 프로그램 내에서 프로그래밍 방식으로 구성을 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-643">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="9e061-644">재시도 정책에 대한 특정 매개 변수(예: 재시도 횟수 및 다시 시도 간격)는 서비스 구성 파일에 저장하여 런타임에 적절한 정책을 만드는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-644">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="9e061-645">이렇게 하면 응용 프로그램을 다시 시작하지 않고도 설정을 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-645">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="9e061-646">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-646">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="9e061-647">재시도 횟수 사이에는 지연을 지정할 수 없습니다. 이 값은 고정이며 지수 시퀀스로 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-647">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="9e061-648">다음과 같이 사용자 지정 재시도 전략을 만들지 않는 경우 최대값만 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-648">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="9e061-649">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-649">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="9e061-650">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="9e061-650">**Context**</span></span> | <span data-ttu-id="9e061-651">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="9e061-651">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="9e061-652">**다시 시도 정책**</span><span class="sxs-lookup"><span data-stu-id="9e061-652">**Retry policy**</span></span> | <span data-ttu-id="9e061-653">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-653">**Settings**</span></span> | <span data-ttu-id="9e061-654">**값**</span><span class="sxs-lookup"><span data-stu-id="9e061-654">**Values**</span></span> | <span data-ttu-id="9e061-655">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="9e061-655">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9e061-656">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="9e061-656">Interactive, UI,</span></span><br /><span data-ttu-id="9e061-657">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-657">or foreground</span></span> |<span data-ttu-id="9e061-658">2초</span><span class="sxs-lookup"><span data-stu-id="9e061-658">2 seconds</span></span> |<span data-ttu-id="9e061-659">지수</span><span class="sxs-lookup"><span data-stu-id="9e061-659">Exponential</span></span> |<span data-ttu-id="9e061-660">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="9e061-660">MaxRetryCount</span></span><br /><span data-ttu-id="9e061-661">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="9e061-661">MaxDelay</span></span> |<span data-ttu-id="9e061-662">3</span><span class="sxs-lookup"><span data-stu-id="9e061-662">3</span></span><br /><span data-ttu-id="9e061-663">750ms</span><span class="sxs-lookup"><span data-stu-id="9e061-663">750 ms</span></span> |<span data-ttu-id="9e061-664">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-664">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="9e061-665">시도 2 - 750ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-665">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="9e061-666">시도 3 – 750ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-666">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="9e061-667">백그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-667">Background</span></span><br /> <span data-ttu-id="9e061-668">또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="9e061-668">or batch</span></span> |<span data-ttu-id="9e061-669">30초</span><span class="sxs-lookup"><span data-stu-id="9e061-669">30 seconds</span></span> |<span data-ttu-id="9e061-670">지수</span><span class="sxs-lookup"><span data-stu-id="9e061-670">Exponential</span></span> |<span data-ttu-id="9e061-671">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="9e061-671">MaxRetryCount</span></span><br /><span data-ttu-id="9e061-672">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="9e061-672">MaxDelay</span></span> |<span data-ttu-id="9e061-673">5</span><span class="sxs-lookup"><span data-stu-id="9e061-673">5</span></span><br /><span data-ttu-id="9e061-674">12초</span><span class="sxs-lookup"><span data-stu-id="9e061-674">12 seconds</span></span> |<span data-ttu-id="9e061-675">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-675">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="9e061-676">시도 2 - ~1초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-676">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="9e061-677">시도 3 - ~3초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-677">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="9e061-678">시도 4 - ~7초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-678">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="9e061-679">시도 5 - ~12초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-679">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="9e061-680">종단 간 대기 시간 대상은 서비스에 대한 연결의 기본 제한 시간을 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-680">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="9e061-681">연결 제한 시간을 더 길게 지정하면 종단 간 대기 시간이 모든 재시도에 대해 이 추가 시간만큼 확장됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-681">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="9e061-682">예</span><span class="sxs-lookup"><span data-stu-id="9e061-682">Examples</span></span>
<span data-ttu-id="9e061-683">다음 코드 예제에서는 Entity Framework를 사용하는 간단한 데이터 액세스 솔루션을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-683">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="9e061-684">이 예제에서는 **DbConfiguration**을 확장하는 **BlogConfiguration**이라는 클래스의 인스턴스를 정의하여 특정 재시도 전략을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-684">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="9e061-685">Entity Framework 재시도 메커니즘 사용에 대한 더 많은 예제는 [연결 복구/재시도 논리](/ef/ef6/fundamentals/connection-resiliency/retry-logic)(영문)에서 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-685">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span>

### <a name="more-information"></a><span data-ttu-id="9e061-686">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-686">More information</span></span>
* [<span data-ttu-id="9e061-687">Azure SQL Database 성능 및 탄력성 가이드(영문)</span><span class="sxs-lookup"><span data-stu-id="9e061-687">Azure SQL Database Performance and Elasticity Guide</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="9e061-688">Entity Framework Core를 사용하는 SQL Database</span><span class="sxs-lookup"><span data-stu-id="9e061-688">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="9e061-689">[Entity Framework Core](/ef/core/)는 .NET Core 개발자가 도메인별 개체를 사용하여 데이터로 작업할 수 있는 개체 관계형 매퍼입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-689">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="9e061-690">여기서는 개발자가 일반적으로 작성해야 하는 대부분의 데이터 액세스 코드가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-690">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="9e061-691">이 Entity Framework 버전은 처음부터 새로 작성되었으며 EF6.x의 모든 기능을 자동으로 상속하지는 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-691">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-692">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-692">Retry mechanism</span></span>
<span data-ttu-id="9e061-693">재시도 지원은 [연결 복원력](/ef/core/miscellaneous/connection-resiliency)이라는 메커니즘을 통해 Entity Framework Core를 사용하는 SQL Database에 액세스할 때 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-693">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="9e061-694">연결 복원력은 EF Core 1.1.0에서 도입되었습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-694">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="9e061-695">기본 추상화는 `IExecutionStrategy` 인터페이스입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-695">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="9e061-696">SQL Azure를 포함한 SQL Server 실행 전략은 항상 재시도할 수 있는 예외 유형을 인지하며 최대 재시도, 재시도 간 지연 등에 대한 인식 가능한 기본값을 갖습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-696">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="9e061-697">예</span><span class="sxs-lookup"><span data-stu-id="9e061-697">Examples</span></span>

<span data-ttu-id="9e061-698">다음 코드는 데이터베이스와의 세션을 나타내는 DbContext 개체를 구성할 때 자동 재시도를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-698">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="9e061-699">다음 코드는 실행 전략을 사용하여 자동 재시도가 있는 트랜잭션 실행 방법을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-699">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="9e061-700">트랜잭션은 대리자에서 정의됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-700">The transaction is defined in a delegate.</span></span> <span data-ttu-id="9e061-701">일시적인 오류가 발생하면 실행 적략이 대리자를 다시 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-701">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="9e061-702">Azure Storage</span><span class="sxs-lookup"><span data-stu-id="9e061-702">Azure Storage</span></span>
<span data-ttu-id="9e061-703">Azure Storage 서비스에는 테이블 및 Blob Storage, 파일 및 저장소 큐가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-703">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="9e061-704">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="9e061-704">Retry mechanism</span></span>
<span data-ttu-id="9e061-705">재시도는 개별 REST 작업 수준에서 수행되며 클라이언트 API 구현의 중요한 부분입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-705">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="9e061-706">클라이언트 저장소 SDK는 [IExtendedRetryPolicy 인터페이스](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy)를 구현하는 클래스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-706">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).</span></span>

<span data-ttu-id="9e061-707">인터페이스 구현은 여러 가지가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-707">There are different implementations of the interface.</span></span> <span data-ttu-id="9e061-708">Storage 클라이언트는 테이블, blob 및 큐에 액세스하기 위해 특별히 설계된 정책에서 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-708">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="9e061-709">각 구현에서는 기본적으로 다시 시도 간격 및 기타 세부 정보를 정의하는 다양한 재시도 전략을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-709">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="9e061-710">기본 제공 클래스는 불규칙 다시 시도 간격으로 선형(일정한 지연) 및 지수를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-710">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="9e061-711">또한 다른 프로세스가 더 높은 수준에서 재시도를 처리하는 경우 사용할 재시도 정책이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-711">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="9e061-712">그러나 기본 제공 클래스에서 제공하지 않는 특정 요구 사항이 있는 경우 고유한 재시도 클래스를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-712">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="9e061-713">RA-GRS(읽기 액세스 지역 중복 저장소)를 사용하고 요청의 결과가 재시도 가능한 오류인 경우 기본과 보조 저장소 서비스 위치 간에 대체 재시도가 전환합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-713">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="9e061-714">자세한 내용은 [Azure Storage 중복 옵션](/azure/storage/common/storage-redundancy)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-714">See [Azure Storage Redundancy Options](/azure/storage/common/storage-redundancy) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="9e061-715">정책 구성</span><span class="sxs-lookup"><span data-stu-id="9e061-715">Policy configuration</span></span>
<span data-ttu-id="9e061-716">재시도 정책은 프로그래밍 방식으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-716">Retry policies are configured programmatically.</span></span> <span data-ttu-id="9e061-717">일반적인 프로시저는 **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** 또는 **QueueRequestOptions** 인스턴스를 만들고 채우는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-717">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="9e061-718">그런 다음 클라이언트에서 요청 옵션 인스턴스를 설정할 수 있으며 클라이언트와 관련된 모든 작업에서 지정된 요청 옵션을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-718">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="9e061-719">요청 옵션 클래스의 채워진 인스턴스를 매개 변수로 작업 메서드에 전달하여 클라이언트 요청 옵션을 재정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-719">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="9e061-720">**OperationContext** 인스턴스를 사용하여 재시도가 수행될 때와 작업이 완료되었을 때 실행할 코드를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-720">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="9e061-721">이 코드는 로그 및 원격 분석에서 사용하기 위해 작업에 대한 정보를 수집할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-721">This code can collect information about the operation for use in logs and telemetry.</span></span>

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

<span data-ttu-id="9e061-722">확장된 재시도 정책은 오류가 재시도에 적합한지 여부를 나타낼 뿐만 아니라 재시도 횟수, 마지막 요청의 결과, 다음 재시도가 기본 위치에서 발생되는지 보조 위치에서 발생되는지를 나타내는 **RetryContext** 개체를 반환합니다(자세한 내용은 아래 표 참조).</span><span class="sxs-lookup"><span data-stu-id="9e061-722">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="9e061-723">**RetryContext** 개체의 속성은 재시도를 시도할 경우 및 시기를 결정하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-723">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="9e061-724">자세한 내용은 [IExtendedRetryPolicy.Evaluate 메서드](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-724">For more details, see [IExtendedRetryPolicy.Evaluate Method](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).</span></span>

<span data-ttu-id="9e061-725">다음 표는 기본 제공 재시도 정책의 기본 설정을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-725">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="9e061-726">**요청 옵션**</span><span class="sxs-lookup"><span data-stu-id="9e061-726">**Request options**</span></span>

| <span data-ttu-id="9e061-727">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-727">**Setting**</span></span> | <span data-ttu-id="9e061-728">**기본값**</span><span class="sxs-lookup"><span data-stu-id="9e061-728">**Default value**</span></span> | <span data-ttu-id="9e061-729">**의미**</span><span class="sxs-lookup"><span data-stu-id="9e061-729">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="9e061-730">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="9e061-730">MaximumExecutionTime</span></span> | <span data-ttu-id="9e061-731">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-731">None</span></span> | <span data-ttu-id="9e061-732">모든 잠재적인 재시도 횟수를 포함하여 요청의 최대 실행 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-732">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="9e061-733">지정하지 않으면 요청에 허용되는 시간이 제한되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-733">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="9e061-734">즉, 요청이 중단될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-734">In other words, the request might hang.</span></span> |
| <span data-ttu-id="9e061-735">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="9e061-735">ServerTimeout</span></span> | <span data-ttu-id="9e061-736">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-736">None</span></span> | <span data-ttu-id="9e061-737">요청에 대한 서버 제한 시간 간격(값은 초로 반올림됨)입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-737">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="9e061-738">지정하지 않으면 서버에 대한 모든 요청에 기본값이 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-738">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="9e061-739">일반적으로 최상의 옵션은 서버 기본값이 사용되도록 이 설정을 생략하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-739">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="9e061-740">LocationMode</span><span class="sxs-lookup"><span data-stu-id="9e061-740">LocationMode</span></span> | <span data-ttu-id="9e061-741">없음</span><span class="sxs-lookup"><span data-stu-id="9e061-741">None</span></span> | <span data-ttu-id="9e061-742">RA-GRS(읽기 액세스 지역 중복 저장소) 복제 옵션을 사용하여 저장소 계정을 만드는 경우 위치 모드를 사용하여 요청을 수신할 위치를 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-742">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="9e061-743">예를 들어 **PrimaryThenSecondary**를 지정하면 요청은 항상 기본 위치로 먼저 전송됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-743">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="9e061-744">요청이 실패하면 보조 위치로 전송됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-744">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="9e061-745">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="9e061-745">RetryPolicy</span></span> | <span data-ttu-id="9e061-746">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="9e061-746">ExponentialPolicy</span></span> | <span data-ttu-id="9e061-747">각 옵션에 대한 자세한 내용은 아래를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-747">See below for details of each option.</span></span> |

<span data-ttu-id="9e061-748">**지수 정책**</span><span class="sxs-lookup"><span data-stu-id="9e061-748">**Exponential policy**</span></span> 

| <span data-ttu-id="9e061-749">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-749">**Setting**</span></span> | <span data-ttu-id="9e061-750">**기본값**</span><span class="sxs-lookup"><span data-stu-id="9e061-750">**Default value**</span></span> | <span data-ttu-id="9e061-751">**의미**</span><span class="sxs-lookup"><span data-stu-id="9e061-751">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="9e061-752">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="9e061-752">maxAttempt</span></span> | <span data-ttu-id="9e061-753">3</span><span class="sxs-lookup"><span data-stu-id="9e061-753">3</span></span> | <span data-ttu-id="9e061-754">재시도 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-754">Number of retry attempts.</span></span> |
| <span data-ttu-id="9e061-755">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-755">deltaBackoff</span></span> | <span data-ttu-id="9e061-756">4초</span><span class="sxs-lookup"><span data-stu-id="9e061-756">4 seconds</span></span> | <span data-ttu-id="9e061-757">재시도 사이의 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-757">Back-off interval between retries.</span></span> <span data-ttu-id="9e061-758">임의 요소를 포함하여 이 timespan의 배수가 이후 재시도 횟수에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-758">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="9e061-759">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-759">MinBackoff</span></span> | <span data-ttu-id="9e061-760">3초</span><span class="sxs-lookup"><span data-stu-id="9e061-760">3 seconds</span></span> | <span data-ttu-id="9e061-761">deltaBackoff에서 계산된 모든 다시 시도 간격에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-761">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="9e061-762">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-762">This value cannot be changed.</span></span>
| <span data-ttu-id="9e061-763">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-763">MaxBackoff</span></span> | <span data-ttu-id="9e061-764">120초</span><span class="sxs-lookup"><span data-stu-id="9e061-764">120 seconds</span></span> | <span data-ttu-id="9e061-765">계산된 다시 시도 간격이 MaxBackoff보다 큰 경우 MaxBackoff가 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-765">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="9e061-766">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-766">This value cannot be changed.</span></span> |

<span data-ttu-id="9e061-767">**선형 정책**</span><span class="sxs-lookup"><span data-stu-id="9e061-767">**Linear policy**</span></span>

| <span data-ttu-id="9e061-768">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-768">**Setting**</span></span> | <span data-ttu-id="9e061-769">**기본값**</span><span class="sxs-lookup"><span data-stu-id="9e061-769">**Default value**</span></span> | <span data-ttu-id="9e061-770">**의미**</span><span class="sxs-lookup"><span data-stu-id="9e061-770">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="9e061-771">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="9e061-771">maxAttempt</span></span> | <span data-ttu-id="9e061-772">3</span><span class="sxs-lookup"><span data-stu-id="9e061-772">3</span></span> | <span data-ttu-id="9e061-773">재시도 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-773">Number of retry attempts.</span></span> |
| <span data-ttu-id="9e061-774">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-774">deltaBackoff</span></span> | <span data-ttu-id="9e061-775">30초</span><span class="sxs-lookup"><span data-stu-id="9e061-775">30 seconds</span></span> | <span data-ttu-id="9e061-776">재시도 사이의 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-776">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="9e061-777">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-777">Retry usage guidance</span></span>
<span data-ttu-id="9e061-778">저장소 클라이언트 API를 사용하여 Azure 저장소 서비스에 액세스하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-778">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="9e061-779">요구 사항에 적합한 Microsoft.WindowsAzure.Storage.RetryPolicies 네임스페이스에서 기본 제공 재시도 정책을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-779">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="9e061-780">대부분의 경우 이러한 정책이면 충분합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-780">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="9e061-781">일괄 작업, 백그라운드 작업 또는 비대화형 시나리오에서 **ExponentialRetry** 정책을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-781">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="9e061-782">이러한 시나리오에서는 일반적으로 서비스가 복구되는 데 더 많은 시간을 허용하여 결과적으로 작업이 성공할 가능성이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-782">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="9e061-783">**RequestOptions** 매개 변수의 **MaximumExecutionTime** 속성을 지정하여 총 실행 시간을 제한하고 제한 시간 값을 선택할 때 작업의 유형 및 크기를 고려하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-783">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="9e061-784">사용자 지정 재시도를 구현해야 하는 경우 저장소 클라이언트 클래스 주위에 래퍼를 만들지 마세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-784">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="9e061-785">대신 **IExtendedRetryPolicy** 인터페이스를 통해 기존 정책을 확장하는 기능을 사용하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-785">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="9e061-786">RA-GRS(읽기 액세스 지역 중복 저장소)를 사용하는 경우 기본 액세스에 실패하면 **LocationMode**를 사용하여 재시도에서 저장소의 보조 읽기 전용 복사본에 액세스하도록 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-786">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="9e061-787">그러나 이 옵션을 사용할 때 기본 저장소에서 복제가 아직 완료되지 않은 경우 응용 프로그램이 오래된 데이터에서 성공적으로 작동하는지 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-787">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="9e061-788">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-788">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="9e061-789">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-789">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="9e061-790">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="9e061-790">**Context**</span></span> | <span data-ttu-id="9e061-791">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="9e061-791">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="9e061-792">**다시 시도 정책**</span><span class="sxs-lookup"><span data-stu-id="9e061-792">**Retry policy**</span></span> | <span data-ttu-id="9e061-793">**설정**</span><span class="sxs-lookup"><span data-stu-id="9e061-793">**Settings**</span></span> | <span data-ttu-id="9e061-794">**값**</span><span class="sxs-lookup"><span data-stu-id="9e061-794">**Values**</span></span> | <span data-ttu-id="9e061-795">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="9e061-795">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9e061-796">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="9e061-796">Interactive, UI,</span></span><br /><span data-ttu-id="9e061-797">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-797">or foreground</span></span> |<span data-ttu-id="9e061-798">2초</span><span class="sxs-lookup"><span data-stu-id="9e061-798">2 seconds</span></span> |<span data-ttu-id="9e061-799">선형</span><span class="sxs-lookup"><span data-stu-id="9e061-799">Linear</span></span> |<span data-ttu-id="9e061-800">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="9e061-800">maxAttempt</span></span><br /><span data-ttu-id="9e061-801">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-801">deltaBackoff</span></span> |<span data-ttu-id="9e061-802">3</span><span class="sxs-lookup"><span data-stu-id="9e061-802">3</span></span><br /><span data-ttu-id="9e061-803">500ms</span><span class="sxs-lookup"><span data-stu-id="9e061-803">500 ms</span></span> |<span data-ttu-id="9e061-804">시도 1 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-804">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="9e061-805">시도 2 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-805">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="9e061-806">시도 3 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-806">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="9e061-807">백그라운드</span><span class="sxs-lookup"><span data-stu-id="9e061-807">Background</span></span><br /><span data-ttu-id="9e061-808">또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="9e061-808">or batch</span></span> |<span data-ttu-id="9e061-809">30초</span><span class="sxs-lookup"><span data-stu-id="9e061-809">30 seconds</span></span> |<span data-ttu-id="9e061-810">지수</span><span class="sxs-lookup"><span data-stu-id="9e061-810">Exponential</span></span> |<span data-ttu-id="9e061-811">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="9e061-811">maxAttempt</span></span><br /><span data-ttu-id="9e061-812">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="9e061-812">deltaBackoff</span></span> |<span data-ttu-id="9e061-813">5</span><span class="sxs-lookup"><span data-stu-id="9e061-813">5</span></span><br /><span data-ttu-id="9e061-814">4초</span><span class="sxs-lookup"><span data-stu-id="9e061-814">4 seconds</span></span> |<span data-ttu-id="9e061-815">시도 1 - ~3초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-815">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="9e061-816">시도 2 - ~7초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-816">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="9e061-817">시도 3 - ~15초 지연</span><span class="sxs-lookup"><span data-stu-id="9e061-817">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="9e061-818">원격 분석</span><span class="sxs-lookup"><span data-stu-id="9e061-818">Telemetry</span></span>
<span data-ttu-id="9e061-819">재시도 횟수는 **TraceSource**에 기록됩니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-819">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="9e061-820">이벤트를 캡처하여 적합한 대상 로그에 기록하려면 **TraceListener**를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-820">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="9e061-821">**TextWriterTraceListener** 또는 **XmlWriterTraceListener**를 사용하여 데이터를 로그 파일에 기록하거나, **EventLogTraceListener**를 사용하여 Windows 이벤트 로그에 기록하거나, **EventProviderTraceListener**를 사용하여 추적 데이터를 ETW 하위 시스템에 기록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-821">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="9e061-822">버퍼의 자동 플러시 및 기록될 이벤트의 자세한 정도(예: 오류, 경고, 정보 및 세부 정보 표시)를 구성할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-822">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="9e061-823">자세한 내용은 [.NET Storage 클라이언트 라이브러리를 사용한 클라이언트 쪽 로깅](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-823">For more information, see [Client-side Logging with the .NET Storage Client Library](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).</span></span>

<span data-ttu-id="9e061-824">작업은 **OperationContext** 인스턴스를 수신하여 사용자 지정 원격 분석 논리를 추가하는 데 사용할 수 있는 **Retrying** 이벤트를 표시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-824">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="9e061-825">자세한 내용은 [OperationContext.Retrying 이벤트](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-825">For more information, see [OperationContext.Retrying Event](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).</span></span>

### <a name="examples"></a><span data-ttu-id="9e061-826">예</span><span class="sxs-lookup"><span data-stu-id="9e061-826">Examples</span></span>
<span data-ttu-id="9e061-827">다음 코드 예제에서는 서로 다른 재시도 설정을 사용하여 대화형 요청과 백그라운드 요청에 대해 각각 하나씩 두 개의 **TableRequestOptions** 인스턴스를 만드는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-827">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="9e061-828">이 예제에서는 클라이언트에서 이러한 두 재시도 정책을 설정하여 모든 요청에 대해 적용하고 특정 요청에서 대화형 전략을 설정하여 클라이언트에 적용된 기본 설정을 재정의합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-828">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="9e061-829">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-829">More information</span></span>
* [<span data-ttu-id="9e061-830">Azure Storage 클라이언트 라이브러리 재시도 정책 권장 사항(영문)</span><span class="sxs-lookup"><span data-stu-id="9e061-830">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="9e061-831">Storage 클라이언트 라이브러리 2.0 – 재시도 정책 구현(영문)</span><span class="sxs-lookup"><span data-stu-id="9e061-831">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](https://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="9e061-832">일반 REST 및 다시 시도 지침</span><span class="sxs-lookup"><span data-stu-id="9e061-832">General REST and retry guidelines</span></span>
<span data-ttu-id="9e061-833">Azure 또는 타사 서비스에 액세스하는 경우 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-833">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="9e061-834">재사용 가능한 코드로 재시도를 관리하는 체계적인 방법을 사용하여 모든 클라이언트 및 모든 솔루션에서 일관된 방법론을 적용할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-834">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="9e061-835">대상 서비스 또는 클라이언트에 기본 제공 재시도 메커니즘이 없는 경우 [Polly][polly]와 같은 재시도 프레임워크를 사용하여 재시도를 관리하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-835">Consider using a retry framework such as [Polly][polly] to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="9e061-836">그러면 일관된 재시도 동작을 구현하는 데 도움이 되며 대상 서비스에 적합한 기본 재시도 전략을 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-836">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="9e061-837">그러나 비표준 동작을 사용하는 서비스, 일시적인 오류를 나타내는 데 예외를 사용하지 않는 서비스 또는 **Retry-Response** 회신을 사용하여 재시도 동작을 관리하려는 경우 사용자 지정 재시도 코드를 만들어야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-837">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="9e061-838">일시적인 검색 논리는 REST를 호출하는 데 사용하는 실제 클라이언트 API에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-838">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="9e061-839">최신 **HttpClient** 클래스와 같은 일부 클라이언트는 성공이 아닌 HTTP 상태 코드를 사용하여 완료된 요청에 대한 예외를 throw하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-839">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> 
* <span data-ttu-id="9e061-840">서비스에서 반환된 HTTP 상태 코드는 오류가 일시적인지 여부를 나타내는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-840">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="9e061-841">클라이언트 또는 재시도 프레임워크에서 생성된 예외를 검사하여 상태 코드에 액세스하거나 해당되는 예외 유형을 확인해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-841">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="9e061-842">다음 HTTP 코드는 일반적으로 재시도가 적합함을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-842">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="9e061-843">408 요청 시간 초과</span><span class="sxs-lookup"><span data-stu-id="9e061-843">408 Request Timeout</span></span>
  * <span data-ttu-id="9e061-844">429 요청이 너무 많음</span><span class="sxs-lookup"><span data-stu-id="9e061-844">429 Too Many Requests</span></span>
  * <span data-ttu-id="9e061-845">500 내부 서버 오류</span><span class="sxs-lookup"><span data-stu-id="9e061-845">500 Internal Server Error</span></span>
  * <span data-ttu-id="9e061-846">502 잘못된 게이트웨이</span><span class="sxs-lookup"><span data-stu-id="9e061-846">502 Bad Gateway</span></span>
  * <span data-ttu-id="9e061-847">503 서비스를 사용할 수 없음</span><span class="sxs-lookup"><span data-stu-id="9e061-847">503 Service Unavailable</span></span>
  * <span data-ttu-id="9e061-848">504 게이트웨이 시간 초과</span><span class="sxs-lookup"><span data-stu-id="9e061-848">504 Gateway Timeout</span></span>
* <span data-ttu-id="9e061-849">재시도 논리가 예외를 기반으로 하는 경우 다음은 일반적으로 연결을 설정할 수 없는 일시적인 오류를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-849">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="9e061-850">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="9e061-850">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="9e061-851">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="9e061-851">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="9e061-852">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="9e061-852">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="9e061-853">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="9e061-853">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="9e061-854">서비스를 사용할 수 없음 상태의 경우 서비스는 **Retry-After** 응답 헤더 또는 다른 사용자 지정 헤더에서 재시도하기 전에 적절한 지연 시간을 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-854">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="9e061-855">또한 서비스는 추가 정보를 사용자 지정 헤더로 전송하거나 응답의 내용에 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-855">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> 
* <span data-ttu-id="9e061-856">408 요청 시간 초과를 제외한 클라이언트 오류(4xx 범위의 오류)를 나타내는 상태 코드에 대해서는 재시도하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="9e061-856">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="9e061-857">서로 다른 네트워크 상태 및 다양한 시스템 부하와 같은 다양한 조건에서 재시도 전략 및 메커니즘을 철저히 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-857">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="9e061-858">재시도 전략</span><span class="sxs-lookup"><span data-stu-id="9e061-858">Retry strategies</span></span>
<span data-ttu-id="9e061-859">다음은 일반적인 재시도 전략 간격의 유형입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-859">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="9e061-860">**지수**.</span><span class="sxs-lookup"><span data-stu-id="9e061-860">**Exponential**.</span></span> <span data-ttu-id="9e061-861">재시도 사이의 간격을 결정하는 무작위 추출 지수 백오프 방법을 사용하여 지정된 횟수의 재시도를 수행하는 재시도 정책입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-861">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="9e061-862">예: </span><span class="sxs-lookup"><span data-stu-id="9e061-862">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="9e061-863">**증분**.</span><span class="sxs-lookup"><span data-stu-id="9e061-863">**Incremental**.</span></span> <span data-ttu-id="9e061-864">재시도 횟수가 지정되고 재시도 간의 시간 간격이 증분되는 재시도 전략입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-864">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="9e061-865">예: </span><span class="sxs-lookup"><span data-stu-id="9e061-865">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="9e061-866">**LinearRetry**.</span><span class="sxs-lookup"><span data-stu-id="9e061-866">**LinearRetry**.</span></span> <span data-ttu-id="9e061-867">재시도 간에 지정된 고정 시간 간격을 사용하여 지정된 횟수의 재시도를 수행하는 재시도 정책입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-867">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="9e061-868">예: </span><span class="sxs-lookup"><span data-stu-id="9e061-868">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="9e061-869">Polly를 통한 일시적인 오류 처리</span><span class="sxs-lookup"><span data-stu-id="9e061-869">Transient fault handling with Polly</span></span>
<span data-ttu-id="9e061-870">[Polly][polly]는 재시도 및 [회로 차단기][circuit-breaker] 전략을 프로그래밍 방식으로 처리하는 라이브러리입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-870">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="9e061-871">Polly 프로젝트는 [.NET Foundation][dotnet-foundation]의 멤버입니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-871">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="9e061-872">Polly는 클라이언트가 기본적으로 재시도를 지원하지 않는 서비스의 유효한 대안이 되며 정확한 구현이 까다로운 사용자 지정 재시도 코드를 작성할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-872">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="9e061-873">Polly는 발생한 오류를 추적하는 방법도 제공하므로 재시도를 기록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9e061-873">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a><span data-ttu-id="9e061-877">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="9e061-877">More information</span></span>
* [<span data-ttu-id="9e061-878">연결 복원력</span><span class="sxs-lookup"><span data-stu-id="9e061-878">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="9e061-879">데이터 요소 - EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="9e061-879">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


