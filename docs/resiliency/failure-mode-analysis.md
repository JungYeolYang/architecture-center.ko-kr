---
title: 실패 모드 분석
description: Azure에 기반한 클라우드 솔루션에 대한 장애 모드 분석을 수행하기 위한 지침입니다.
author: MikeWasson
ms.date: 05/07/2018
ms.custom: resiliency
ms.openlocfilehash: e74a98ed1d57c3bd0b3b518ff4fae743dd12f02b
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113215"
---
# <a name="failure-mode-analysis"></a><span data-ttu-id="96fac-103">실패 모드 분석</span><span class="sxs-lookup"><span data-stu-id="96fac-103">Failure mode analysis</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="96fac-104">FMA(장애 모드 분석)는 가능한 장애 지점을 식별하여 시스템에 복원력을 구축하는 프로세스입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-104">Failure mode analysis (FMA) is a process for building resiliency into a system, by identifying possible failure points in the system.</span></span> <span data-ttu-id="96fac-105">FMA는 아키텍처 및 디자인 단계의 일부여야 하므로 처음부터 장애 복구를 시스템에 구축할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-105">The FMA should be part of the architecture and design phases, so that you can build failure recovery into the system from the beginning.</span></span>

<span data-ttu-id="96fac-106">FMA를 수행하는 일반적인 프로세스는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-106">Here is the general process to conduct an FMA:</span></span>

1. <span data-ttu-id="96fac-107">시스템의 모든 구성 요소를 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-107">Identify all of the components in the system.</span></span> <span data-ttu-id="96fac-108">ID 공급자, 타사 서비스 등과 같은 외부 종속성을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-108">Include external dependencies, such as as identity providers, third-party services, and so on.</span></span>
2. <span data-ttu-id="96fac-109">각 구성 요소마다 발생할 수 있는 잠재적 장애를 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-109">For each component, identify potential failures that could occur.</span></span> <span data-ttu-id="96fac-110">단일 구성 요소에 둘 이상의 장애 모드가 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-110">A single component may have more than one failure mode.</span></span> <span data-ttu-id="96fac-111">예를 들어 영향력 및 가능한 완화 방식이 서로 다르기 때문에 읽기 오류와 쓰기 오류를 별도로 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-111">For example, you should consider read failures and write failures separately, because the impact and possible mitigations will be different.</span></span>
3. <span data-ttu-id="96fac-112">전체적인 위험도에 따라 각 장애 모드를 평가합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-112">Rate each failure mode according to its overall risk.</span></span> <span data-ttu-id="96fac-113">다음 항목을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-113">Consider these factors:</span></span>

   - <span data-ttu-id="96fac-114">장애의 가능성은 무엇인가요?</span><span class="sxs-lookup"><span data-stu-id="96fac-114">What is the likelihood of the failure.</span></span> <span data-ttu-id="96fac-115">비교적 일반적인가요?</span><span class="sxs-lookup"><span data-stu-id="96fac-115">Is it relatively common?</span></span> <span data-ttu-id="96fac-116">매우 드물게 발생하나요?</span><span class="sxs-lookup"><span data-stu-id="96fac-116">Extrememly rare?</span></span> <span data-ttu-id="96fac-117">우선 순위를 지정하기 위한 것이므로 정확한 숫자는 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-117">You don't need exact numbers; the purpose is to help rank the priority.</span></span>
   - <span data-ttu-id="96fac-118">가용성, 데이터 손실, 금전적 비용 및 비즈니스 중단과 관련하여 애플리케이션에 미치는 영향은 무엇인가요?</span><span class="sxs-lookup"><span data-stu-id="96fac-118">What is the impact on the application, in terms of availability, data loss, monetary cost, and business disruption?</span></span>

4. <span data-ttu-id="96fac-119">각 장애 모드에 대해 애플리케이션에서 응답하고 복구하는 방법을 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-119">For each failure mode, determine how the application will respond and recover.</span></span> <span data-ttu-id="96fac-120">비용 및 애플리케이션 복잡성의 장단점을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-120">Consider tradeoffs in cost and application complexity.</span></span>

<span data-ttu-id="96fac-121">FMA 프로세스에 대한 시작점으로, 이 문서에는 잠재적인 장애 모드의 카탈로그 및 해당 완화 방법이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-121">As a starting point for your FMA process, this article contains a catalog of potential failure modes and their mitigations.</span></span> <span data-ttu-id="96fac-122">카탈로그는 기술 또는 Azure 서비스와 애플리케이션 수준 디자인을 위한 일반 범주로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-122">The catalog is organized by technology or Azure service, plus a general category for application-level design.</span></span> <span data-ttu-id="96fac-123">카탈로그는 완벽하지 않지만 Azure 핵심 서비스의 많은 부분을 다루고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-123">The catalog is not exhaustive, but covers many of the core Azure services.</span></span>

## <a name="app-service"></a><span data-ttu-id="96fac-124">App Service</span><span class="sxs-lookup"><span data-stu-id="96fac-124">App Service</span></span>

<!-- markdownlint-disable MD026 -->

### <a name="app-service-app-shuts-down"></a><span data-ttu-id="96fac-125">App Service 앱이 종료됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-125">App Service app shuts down.</span></span>

<span data-ttu-id="96fac-126">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-126">**Detection**.</span></span> <span data-ttu-id="96fac-127">가능한 원인:</span><span class="sxs-lookup"><span data-stu-id="96fac-127">Possible causes:</span></span>

- <span data-ttu-id="96fac-128">예상된 종료</span><span class="sxs-lookup"><span data-stu-id="96fac-128">Expected shutdown</span></span>

  - <span data-ttu-id="96fac-129">운영자가 애플리케이션(예: Azure Portal 사용)을 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-129">An operator shuts down the application; for example, using the Azure portal.</span></span>
  - <span data-ttu-id="96fac-130">앱이 유휴 상태이므로 언로드되었습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-130">The app was unloaded because it was idle.</span></span> <span data-ttu-id="96fac-131">(`Always On` 설정이 사용 안 함인 경우에만)</span><span class="sxs-lookup"><span data-stu-id="96fac-131">(Only if the `Always On` setting is disabled.)</span></span>

- <span data-ttu-id="96fac-132">예기치 않은 종료</span><span class="sxs-lookup"><span data-stu-id="96fac-132">Unexpected shutdown</span></span>

  - <span data-ttu-id="96fac-133">앱이 충돌합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-133">The app crashes.</span></span>
  - <span data-ttu-id="96fac-134">App Service VM 인스턴스를 사용할 수 없게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-134">An App Service VM instance becomes unavailable.</span></span>

<span data-ttu-id="96fac-135">Application_End 로깅은 애플리케이션 도메인 종료(소프트 프로세스 크래시)를 catch하며, 애플리케이션 도메인 종료를 catch할 수 있는 유일한 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-135">Application_End logging will catch the app domain shutdown (soft process crash) and is the only way to catch the application domain shutdowns.</span></span>

<span data-ttu-id="96fac-136">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-136">**Recovery:**</span></span>

- <span data-ttu-id="96fac-137">종료가 예상되면 애플리케이션의 종료 이벤트를 사용하여 정상적으로 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-137">If the shutdown was expected, use the application's shutdown event to shut down gracefully.</span></span> <span data-ttu-id="96fac-138">예를 들어 ASP.NET에서는 `Application_End` 메서드를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-138">For example, in ASP.NET, use the `Application_End` method.</span></span>
- <span data-ttu-id="96fac-139">유휴 상태에 있는 애플리케이션이 언로드되었으면 다음 요청 시 자동으로 다시 시작됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-139">If the application was unloaded while idle, it is automatically restarted on the next request.</span></span> <span data-ttu-id="96fac-140">그러나 "콜드 부팅" 비용이 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-140">However, you will incur the "cold start" cost.</span></span>
- <span data-ttu-id="96fac-141">유휴 상태에 있는 애플리케이션이 언로드되지 않도록 방지하려면 웹앱에서 `Always On` 설정을 활성화합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-141">To prevent the application from being unloaded while idle, enable the `Always On` setting in the web app.</span></span> <span data-ttu-id="96fac-142">[Azure App Service에서 웹앱 구성][app-service-configure]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-142">See [Configure web apps in Azure App Service][app-service-configure].</span></span>
- <span data-ttu-id="96fac-143">운영자가 앱을 종료하지 못하도록 하려면 `ReadOnly` 수준의 리소스 잠금을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-143">To prevent an operator from shutting down the app, set a resource lock with `ReadOnly` level.</span></span> <span data-ttu-id="96fac-144">[Azure Resource Manager를 사용하여 리소스 잠그기][rm-locks]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-144">See [Lock resources with Azure Resource Manager][rm-locks].</span></span>
- <span data-ttu-id="96fac-145">앱의 작동이 중단되거나 App Service VM을 사용할 수 없게 되면 App Service에서 앱을 자동으로 다시 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-145">If the app crashes or an App Service VM becomes unavailable, App Service automatically restarts the app.</span></span>

<span data-ttu-id="96fac-146">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-146">**Diagnostics**.</span></span> <span data-ttu-id="96fac-147">애플리케이션 및 웹 서버 로그.</span><span class="sxs-lookup"><span data-stu-id="96fac-147">Application logs and web server logs.</span></span> <span data-ttu-id="96fac-148">[Azure App Service에서 웹앱에 대한 진단 로깅 설정][app-service-logging]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-148">See [Enable diagnostics logging for web apps in Azure App Service][app-service-logging].</span></span>

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a><span data-ttu-id="96fac-149">특정 사용자가 반복적으로 잘못 요청하거나 시스템을 오버로드합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-149">A particular user repeatedly makes bad requests or overloads the system.</span></span>

<span data-ttu-id="96fac-150">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-150">**Detection**.</span></span> <span data-ttu-id="96fac-151">사용자를 인증하고 애플리케이션 로그에 사용자 ID를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-151">Authenticate users and include user ID in application logs.</span></span>

<span data-ttu-id="96fac-152">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-152">**Recovery:**</span></span>

- <span data-ttu-id="96fac-153">[Azure API Management][api-management]를 사용하여 사용자의 요청을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-153">Use [Azure API Management][api-management] to throttle requests from the user.</span></span> <span data-ttu-id="96fac-154">[Azure API Management로 고급 요청 제한][api-management-throttling]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-154">See [Advanced request throttling with Azure API Management][api-management-throttling]</span></span>
- <span data-ttu-id="96fac-155">사용자를 차단합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-155">Block the user.</span></span>

<span data-ttu-id="96fac-156">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-156">**Diagnostics**.</span></span> <span data-ttu-id="96fac-157">모든 인증 요청을 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-157">Log all authentication requests.</span></span>

### <a name="a-bad-update-was-deployed"></a><span data-ttu-id="96fac-158">잘못된 업데이트가 배포되었습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-158">A bad update was deployed.</span></span>

<span data-ttu-id="96fac-159">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-159">**Detection**.</span></span> <span data-ttu-id="96fac-160">Azure Portal을 통해 애플리케이션 상태를 모니터링하거나([Azure 웹앱 성능 모니터링][app-insights-web-apps] 참조) [상태 엔드포인트 모니터링 패턴][health-endpoint-monitoring-pattern]을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-160">Monitor the application health through the Azure Portal (see [Monitor Azure web app performance][app-insights-web-apps]) or implement the [health endpoint monitoring pattern][health-endpoint-monitoring-pattern].</span></span>

<span data-ttu-id="96fac-161">**복구:**.</span><span class="sxs-lookup"><span data-stu-id="96fac-161">**Recovery:**.</span></span> <span data-ttu-id="96fac-162">여러 [배포 슬롯][app-service-slots]을 사용하고 마지막으로 성공한 배포로 롤백합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-162">Use multiple [deployment slots][app-service-slots] and roll back to the last-known-good deployment.</span></span> <span data-ttu-id="96fac-163">자세한 내용은 [기본 웹 애플리케이션][ra-web-apps-basic]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-163">For more information, see [Basic web application][ra-web-apps-basic].</span></span>

## <a name="azure-active-directory"></a><span data-ttu-id="96fac-164">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="96fac-164">Azure Active Directory</span></span>

### <a name="openid-connect-oidc-authentication-fails"></a><span data-ttu-id="96fac-165">OIDC(OpenID Connect) 인증이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-165">OpenID Connect (OIDC) authentication fails.</span></span>

<span data-ttu-id="96fac-166">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-166">**Detection**.</span></span> <span data-ttu-id="96fac-167">가능한 장애 모드는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-167">Possible failure modes include:</span></span>

1. <span data-ttu-id="96fac-168">Azure AD를 사용할 수 없거나 네트워크 문제로 인해 연결할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-168">Azure AD is not available, or cannot be reached due to a network problem.</span></span> <span data-ttu-id="96fac-169">인증 엔드포인트로의 리디렉션이 실패하고 OIDC 미들웨어에서 예외를 throw합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-169">Redirection to the authentication endpoint fails, and the OIDC middleware throws an exception.</span></span>
2. <span data-ttu-id="96fac-170">Azure AD 테넌트가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-170">Azure AD tenant does not exist.</span></span> <span data-ttu-id="96fac-171">인증 엔드포인트로 리디렉션하면 HTTP 오류 코드가 반환되고 OIDC 미들웨어에서 예외를 throw합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-171">Redirection to the authentication endpoint returns an HTTP error code, and the OIDC middleware throws an exception.</span></span>
3. <span data-ttu-id="96fac-172">사용자를 인증할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-172">User cannot authenticate.</span></span> <span data-ttu-id="96fac-173">검색 전략이 필요하지 않으며, Azure AD에서 로그인 실패를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-173">No detection strategy is necessary; Azure AD handles login failures.</span></span>

<span data-ttu-id="96fac-174">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-174">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-175">미들웨어에서 처리되지 않은 예외를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-175">Catch unhandled exceptions from the middleware.</span></span>
2. <span data-ttu-id="96fac-176">`AuthenticationFailed` 이벤트를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-176">Handle `AuthenticationFailed` events.</span></span>
3. <span data-ttu-id="96fac-177">사용자를 오류 페이지로 리디렉션합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-177">Redirect the user to an error page.</span></span>
4. <span data-ttu-id="96fac-178">사용자가 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-178">User retries.</span></span>

## <a name="azure-search"></a><span data-ttu-id="96fac-179">Azure Search</span><span class="sxs-lookup"><span data-stu-id="96fac-179">Azure Search</span></span>

### <a name="writing-data-to-azure-search-fails"></a><span data-ttu-id="96fac-180">Azure Search에 데이터를 쓰는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-180">Writing data to Azure Search fails.</span></span>

<span data-ttu-id="96fac-181">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-181">**Detection**.</span></span> <span data-ttu-id="96fac-182">`Microsoft.Rest.Azure.CloudException` 오류를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-182">Catch `Microsoft.Rest.Azure.CloudException` errors.</span></span>

<span data-ttu-id="96fac-183">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-183">**Recovery:**</span></span>

<span data-ttu-id="96fac-184">일시적 장애 후에 [Search .NET SDK][search-sdk]에서 자동으로 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-184">The [Search .NET SDK][search-sdk] automatically retries after transient failures.</span></span> <span data-ttu-id="96fac-185">클라이언트 SDK에서 throw된 모든 예외는 일시적이지 않은 오류로 처리되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-185">Any exceptions thrown by the client SDK should be treated as non-transient errors.</span></span>

<span data-ttu-id="96fac-186">기본 다시 시도 정책은 지수 백오프를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-186">The default retry policy uses exponential back-off.</span></span> <span data-ttu-id="96fac-187">다른 다시 시도 정책을 사용하려면 `SearchIndexClient` 또는 `SearchServiceClient` 클래스에서 `SetRetryPolicy`를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-187">To use a different retry policy, call `SetRetryPolicy` on the `SearchIndexClient` or `SearchServiceClient` class.</span></span> <span data-ttu-id="96fac-188">자세한 내용은 [자동 다시 시도][auto-rest-client-retry]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-188">For more information, see [Automatic Retries][auto-rest-client-retry].</span></span>

<span data-ttu-id="96fac-189">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-189">**Diagnostics**.</span></span> <span data-ttu-id="96fac-190">[Search 트래픽 분석][search-analytics]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-190">Use [Search Traffic Analytics][search-analytics].</span></span>

### <a name="reading-data-from-azure-search-fails"></a><span data-ttu-id="96fac-191">Azure Search에서 데이터를 읽는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-191">Reading data from Azure Search fails.</span></span>

<span data-ttu-id="96fac-192">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-192">**Detection**.</span></span> <span data-ttu-id="96fac-193">`Microsoft.Rest.Azure.CloudException` 오류를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-193">Catch `Microsoft.Rest.Azure.CloudException` errors.</span></span>

<span data-ttu-id="96fac-194">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-194">**Recovery:**</span></span>

<span data-ttu-id="96fac-195">일시적 장애 후에 [Search .NET SDK][search-sdk]에서 자동으로 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-195">The [Search .NET SDK][search-sdk]  automatically retries after transient failures.</span></span> <span data-ttu-id="96fac-196">클라이언트 SDK에서 throw된 모든 예외는 일시적이지 않은 오류로 처리되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-196">Any exceptions thrown by the client SDK should be treated as non-transient errors.</span></span>

<span data-ttu-id="96fac-197">기본 다시 시도 정책은 지수 백오프를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-197">The default retry policy uses exponential back-off.</span></span> <span data-ttu-id="96fac-198">다른 다시 시도 정책을 사용하려면 `SearchIndexClient` 또는 `SearchServiceClient` 클래스에서 `SetRetryPolicy`를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-198">To use a different retry policy, call `SetRetryPolicy` on the `SearchIndexClient` or `SearchServiceClient` class.</span></span> <span data-ttu-id="96fac-199">자세한 내용은 [자동 다시 시도][auto-rest-client-retry]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-199">For more information, see [Automatic Retries][auto-rest-client-retry].</span></span>

<span data-ttu-id="96fac-200">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-200">**Diagnostics**.</span></span> <span data-ttu-id="96fac-201">[Search 트래픽 분석][search-analytics]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-201">Use [Search Traffic Analytics][search-analytics].</span></span>

## <a name="cassandra"></a><span data-ttu-id="96fac-202">Cassandra</span><span class="sxs-lookup"><span data-stu-id="96fac-202">Cassandra</span></span>

### <a name="reading-or-writing-to-a-node-fails"></a><span data-ttu-id="96fac-203">노드에서 읽거나 쓰는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-203">Reading or writing to a node fails.</span></span>

<span data-ttu-id="96fac-204">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-204">**Detection**.</span></span> <span data-ttu-id="96fac-205">예외를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-205">Catch the exception.</span></span> <span data-ttu-id="96fac-206">.NET 클라이언트의 경우 일반적으로 `System.Web.HttpException`입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-206">For .NET clients, this will typically be `System.Web.HttpException`.</span></span> <span data-ttu-id="96fac-207">다른 클라이언트에는 다른 예외 형식이 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-207">Other client may have other exception types.</span></span>  <span data-ttu-id="96fac-208">자세한 내용은 [올바른 Cassandra 오류 처리](https://www.datastax.com/dev/blog/cassandra-error-handling-done-right)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-208">For more information, see [Cassandra error handling done right](https://www.datastax.com/dev/blog/cassandra-error-handling-done-right).</span></span>

<span data-ttu-id="96fac-209">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-209">**Recovery:**</span></span>

- <span data-ttu-id="96fac-210">각 [Cassandra 클라이언트](https://wiki.apache.org/cassandra/ClientOptions)에는 자체의 다시 시도 정책 및 기능이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-210">Each [Cassandra client](https://wiki.apache.org/cassandra/ClientOptions) has its own retry policies and capabilities.</span></span> <span data-ttu-id="96fac-211">자세한 내용은 [올바른 Cassandra 오류 처리][cassandra-error-handling]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-211">For more information, see [Cassandra error handling done right][cassandra-error-handling].</span></span>
- <span data-ttu-id="96fac-212">데이터 노드가 장애 도메인에 분산되어 있는 랙 인식 배포를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-212">Use a rack-aware deployment, with data nodes distributed across the fault domains.</span></span>
- <span data-ttu-id="96fac-213">로컬 쿼럼 일관성을 사용하여 여러 지역에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-213">Deploy to multiple regions with local quorum consistency.</span></span> <span data-ttu-id="96fac-214">일시적이지 않은 장애가 발생하면 다른 지역으로 장애 조치합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-214">If a non-transient failure occurs, fail over to another region.</span></span>

<span data-ttu-id="96fac-215">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-215">**Diagnostics**.</span></span> <span data-ttu-id="96fac-216">애플리케이션 로그 전송 사용</span><span class="sxs-lookup"><span data-stu-id="96fac-216">Application logs</span></span>

## <a name="cloud-service"></a><span data-ttu-id="96fac-217">클라우드 서비스</span><span class="sxs-lookup"><span data-stu-id="96fac-217">Cloud Service</span></span>

### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a><span data-ttu-id="96fac-218">웹 또는 작업자 역할이 예기치 않게 종료됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-218">Web or worker roles are unexpectedly being shut down.</span></span>

<span data-ttu-id="96fac-219">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-219">**Detection**.</span></span> <span data-ttu-id="96fac-220">[RoleEnvironment.Stopping][RoleEnvironment.Stopping] 이벤트가 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-220">The [RoleEnvironment.Stopping][RoleEnvironment.Stopping] event is fired.</span></span>

<span data-ttu-id="96fac-221">**복구**.</span><span class="sxs-lookup"><span data-stu-id="96fac-221">**Recovery**.</span></span> <span data-ttu-id="96fac-222">[RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] 메서드를 재정의하여 정상적으로 정리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-222">Override the [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] method to gracefully clean up.</span></span> <span data-ttu-id="96fac-223">자세한 내용은 [Azure OnStop 이벤트를 처리하는 올바른 방법][onstop-events](블로그)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-223">For more information, see [The Right Way to Handle Azure OnStop Events][onstop-events] (blog).</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="96fac-224">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="96fac-224">Cosmos DB</span></span>

### <a name="reading-data-fails"></a><span data-ttu-id="96fac-225">데이터 읽기가 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-225">Reading data fails.</span></span>

<span data-ttu-id="96fac-226">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-226">**Detection**.</span></span> <span data-ttu-id="96fac-227">`System.Net.Http.HttpRequestException` 또는 `Microsoft.Azure.Documents.DocumentClientException`을 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-227">Catch `System.Net.Http.HttpRequestException` or `Microsoft.Azure.Documents.DocumentClientException`.</span></span>

<span data-ttu-id="96fac-228">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-228">**Recovery:**</span></span>

- <span data-ttu-id="96fac-229">SDK에서 실패한 시도를 자동으로 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-229">The SDK automatically retries failed attempts.</span></span> <span data-ttu-id="96fac-230">다시 시도 횟수 및 최대 대기 시간을 설정하려면 `ConnectionPolicy.RetryOptions`를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-230">To set the number of retries and the maximum wait time, configure `ConnectionPolicy.RetryOptions`.</span></span> <span data-ttu-id="96fac-231">클라이언트에서 발생시키는 예외는 재시도 정책 시도 횟수를 초과하거나 일시적인 오류가 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-231">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>
- <span data-ttu-id="96fac-232">Cosmos DB에서 클라이언트를 제한하는 경우 HTTP 429 오류를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-232">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="96fac-233">`DocumentClientException`에서 상태 코드를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-233">Check the status code in the `DocumentClientException`.</span></span> <span data-ttu-id="96fac-234">429 오류가 지속적으로 발생하면 컬렉션의 처리량 값을 늘리는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-234">If you are getting error 429 consistently, consider increasing the throughput value of the collection.</span></span>
  - <span data-ttu-id="96fac-235">MongoDB API를 사용하는 경우 제한할 때 서비스에서 16500 오류 코드를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-235">If you are using the MongoDB API, the service returns error code 16500 when throttling.</span></span>
- <span data-ttu-id="96fac-236">둘 이상의 지역에서 Cosmos DB 데이터베이스를 복제합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-236">Replicate the Cosmos DB database across two or more regions.</span></span> <span data-ttu-id="96fac-237">모든 복제본은 읽을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-237">All replicas are readable.</span></span> <span data-ttu-id="96fac-238">클라이언트 SDK를 사용하여 `PreferredLocations` 매개 변수를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-238">Using the client SDKs, specify the `PreferredLocations` parameter.</span></span> <span data-ttu-id="96fac-239">이는 순서가 지정된 Azure 지역 목록입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-239">This is an ordered list of Azure regions.</span></span> <span data-ttu-id="96fac-240">모든 읽기는 목록에서 사용 가능한 첫 번째 지역으로 보내집니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-240">All reads will be sent to the first available region in the list.</span></span> <span data-ttu-id="96fac-241">요청이 실패하면 클라이언트에서 목록의 다른 지역을 순서대로 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-241">If the request fails, the client will try the other regions in the list, in order.</span></span> <span data-ttu-id="96fac-242">자세한 내용은 [SQL API를 사용하여 Azure Cosmos DB 전역 배포를 설정하는 방법][cosmosdb-multi-region]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-242">For more information, see [How to setup Azure Cosmos DB global distribution using the SQL API][cosmosdb-multi-region].</span></span>

<span data-ttu-id="96fac-243">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-243">**Diagnostics**.</span></span> <span data-ttu-id="96fac-244">클라이언트 쪽에서 모든 오류를 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-244">Log all errors on the client side.</span></span>

### <a name="writing-data-fails"></a><span data-ttu-id="96fac-245">데이터 쓰기가 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-245">Writing data fails.</span></span>

<span data-ttu-id="96fac-246">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-246">**Detection**.</span></span> <span data-ttu-id="96fac-247">`System.Net.Http.HttpRequestException` 또는 `Microsoft.Azure.Documents.DocumentClientException`을 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-247">Catch `System.Net.Http.HttpRequestException` or `Microsoft.Azure.Documents.DocumentClientException`.</span></span>

<span data-ttu-id="96fac-248">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-248">**Recovery:**</span></span>

- <span data-ttu-id="96fac-249">SDK에서 실패한 시도를 자동으로 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-249">The SDK automatically retries failed attempts.</span></span> <span data-ttu-id="96fac-250">다시 시도 횟수 및 최대 대기 시간을 설정하려면 `ConnectionPolicy.RetryOptions`를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-250">To set the number of retries and the maximum wait time, configure `ConnectionPolicy.RetryOptions`.</span></span> <span data-ttu-id="96fac-251">클라이언트에서 발생시키는 예외는 재시도 정책 시도 횟수를 초과하거나 일시적인 오류가 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-251">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>
- <span data-ttu-id="96fac-252">Cosmos DB에서 클라이언트를 제한하는 경우 HTTP 429 오류를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-252">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="96fac-253">`DocumentClientException`에서 상태 코드를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-253">Check the status code in the `DocumentClientException`.</span></span> <span data-ttu-id="96fac-254">429 오류가 지속적으로 발생하면 컬렉션의 처리량 값을 늘리는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-254">If you are getting error 429 consistently, consider increasing the throughput value of the collection.</span></span>
- <span data-ttu-id="96fac-255">둘 이상의 지역에서 Cosmos DB 데이터베이스를 복제합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-255">Replicate the Cosmos DB database across two or more regions.</span></span> <span data-ttu-id="96fac-256">주 지역이 실패하면 다른 지역이 쓰기 지역으로 승격됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-256">If the primary region fails, another region will be promoted to write.</span></span> <span data-ttu-id="96fac-257">또한 장애 조치를 수동으로 트리거할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-257">You can also trigger a failover manually.</span></span> <span data-ttu-id="96fac-258">SDK에서 자동 검색 및 라우팅을 수행하므로 장애 조치 후에 애플리케이션 코드가 계속 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-258">The SDK does automatic discovery and routing, so application code continues to work after a failover.</span></span> <span data-ttu-id="96fac-259">장애 조치 기간(일반적으로 몇 분) 동안 SDK에서 새 쓰기 지역을 찾으므로 쓰기 작업의 대기 시간이 길어집니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-259">During the failover period (typically minutes), write operations will have higher latency, as the SDK finds the new write region.</span></span> <span data-ttu-id="96fac-260">자세한 내용은 [SQL API를 사용하여 Azure Cosmos DB 전역 배포를 설정하는 방법][cosmosdb-multi-region]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-260">For more information, see [How to setup Azure Cosmos DB global distribution using the SQL API][cosmosdb-multi-region].</span></span>
- <span data-ttu-id="96fac-261">대체(fallback) 방식으로, 백업 큐에 문서를 유지하고 나중에 큐를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-261">As a fallback, persist the document to a backup queue, and process the queue later.</span></span>

<span data-ttu-id="96fac-262">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-262">**Diagnostics**.</span></span> <span data-ttu-id="96fac-263">클라이언트 쪽에서 모든 오류를 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-263">Log all errors on the client side.</span></span>

## <a name="elasticsearch"></a><span data-ttu-id="96fac-264">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="96fac-264">Elasticsearch</span></span>

### <a name="reading-data-from-elasticsearch-fails"></a><span data-ttu-id="96fac-265">Elasticsearch에서 데이터를 읽는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-265">Reading data from Elasticsearch fails.</span></span>

<span data-ttu-id="96fac-266">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-266">**Detection**.</span></span> <span data-ttu-id="96fac-267">사용 중인 특정 [Elasticsearch 클라이언트][elasticsearch-client]에 대한 적절한 예외를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-267">Catch the appropriate exception for the particular [Elasticsearch client][elasticsearch-client] being used.</span></span>

<span data-ttu-id="96fac-268">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-268">**Recovery:**</span></span>

- <span data-ttu-id="96fac-269">다시 시도 메커니즘을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-269">Use a retry mechanism.</span></span> <span data-ttu-id="96fac-270">각 클라이언트에는 자체의 다시 시도 정책이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-270">Each client has its own retry policies.</span></span>
- <span data-ttu-id="96fac-271">여러 Elasticsearch 노드를 배포하고 고가용성을 위해 복제를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-271">Deploy multiple Elasticsearch nodes and use replication for high availability.</span></span>

<span data-ttu-id="96fac-272">자세한 내용은 [Azure에서 Elasticsearch 실행][elasticsearch-azure]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-272">For more information, see [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

<span data-ttu-id="96fac-273">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-273">**Diagnostics**.</span></span> <span data-ttu-id="96fac-274">Elasticsearch에 대한 모니터링 도구를 사용하거나 클라이언트 쪽에서 페이로드를 사용하여 모든 오류를 기록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-274">You can use monitoring tools for Elasticsearch, or log all errors on the client side with the payload.</span></span> <span data-ttu-id="96fac-275">[Azure에서 Elasticsearch 실행][elasticsearch-azure]의 '모니터링' 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-275">See the 'Monitoring' section in [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

### <a name="writing-data-to-elasticsearch-fails"></a><span data-ttu-id="96fac-276">Elasticsearch에 데이터를 쓰는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-276">Writing data to Elasticsearch fails.</span></span>

<span data-ttu-id="96fac-277">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-277">**Detection**.</span></span> <span data-ttu-id="96fac-278">사용 중인 특정 [Elasticsearch 클라이언트][elasticsearch-client]에 대한 적절한 예외를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-278">Catch the appropriate exception for the particular [Elasticsearch client][elasticsearch-client] being used.</span></span>

<span data-ttu-id="96fac-279">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-279">**Recovery:**</span></span>

- <span data-ttu-id="96fac-280">다시 시도 메커니즘을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-280">Use a retry mechanism.</span></span> <span data-ttu-id="96fac-281">각 클라이언트에는 자체의 다시 시도 정책이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-281">Each client has its own retry policies.</span></span>
- <span data-ttu-id="96fac-282">애플리케이션에서 축소된 일관성 수준을 허용할 수 있으면 `quorum`의 `write_consistency` 설정으로 쓰기 작업을 수행하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-282">If the application can tolerate a reduced consistency level, consider writing with `write_consistency` setting of `quorum`.</span></span>

<span data-ttu-id="96fac-283">자세한 내용은 [Azure에서 Elasticsearch 실행][elasticsearch-azure]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-283">For more information, see [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

<span data-ttu-id="96fac-284">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-284">**Diagnostics**.</span></span> <span data-ttu-id="96fac-285">Elasticsearch에 대한 모니터링 도구를 사용하거나 클라이언트 쪽에서 페이로드를 사용하여 모든 오류를 기록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-285">You can use monitoring tools for Elasticsearch, or log all errors on the client side with the payload.</span></span> <span data-ttu-id="96fac-286">[Azure에서 Elasticsearch 실행][elasticsearch-azure]의 '모니터링' 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-286">See the 'Monitoring' section in [Running Elasticsearch on Azure][elasticsearch-azure].</span></span>

## <a name="queue-storage"></a><span data-ttu-id="96fac-287">큐 저장소</span><span class="sxs-lookup"><span data-stu-id="96fac-287">Queue storage</span></span>

### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a><span data-ttu-id="96fac-288">Azure Queue 저장소에 메시지를 쓰는 작업이 일관되게 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-288">Writing a message to Azure Queue storage fails consistently.</span></span>

<span data-ttu-id="96fac-289">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-289">**Detection**.</span></span> <span data-ttu-id="96fac-290">*N*회의 다시 시도를 수행한 후에도 쓰기 작업이 계속 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-290">After *N* retry attempts, the write operation still fails.</span></span>

<span data-ttu-id="96fac-291">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-291">**Recovery:**</span></span>

- <span data-ttu-id="96fac-292">로컬 캐시에 데이터를 저장하고, 나중에 서비스를 사용할 수 있게 되면 저장소에 쓰기를 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-292">Store the data in a local cache, and forward the writes to storage later, when the service becomes available.</span></span>
- <span data-ttu-id="96fac-293">보조 큐를 만들고, 기본 큐를 사용할 수 없는 경우 이 보조 큐에 씁니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-293">Create a secondary queue, and write to that queue if the primary queue is unavailable.</span></span>

<span data-ttu-id="96fac-294">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-294">**Diagnostics**.</span></span> <span data-ttu-id="96fac-295">[저장소 메트릭][storage-metrics]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-295">Use [storage metrics][storage-metrics].</span></span>

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a><span data-ttu-id="96fac-296">애플리케이션에서 큐의 특정 메시지를 처리할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-296">The application cannot process a particular message from the queue.</span></span>

<span data-ttu-id="96fac-297">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-297">**Detection**.</span></span> <span data-ttu-id="96fac-298">애플리케이션 특정입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-298">Application specific.</span></span> <span data-ttu-id="96fac-299">예를 들어 메시지에 유효하지 않은 데이터가 있거나 비즈니스 논리가 어떤 이유로 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-299">For example, the message contains invalid data, or the business logic fails for some reason.</span></span>

<span data-ttu-id="96fac-300">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-300">**Recovery:**</span></span>

<span data-ttu-id="96fac-301">메시지를 별도의 큐로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-301">Move the message to a separate queue.</span></span> <span data-ttu-id="96fac-302">별도의 프로세스를 실행하여 해당 큐의 메시지를 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-302">Run a separate process to examine the messages in that queue.</span></span>

<span data-ttu-id="96fac-303">이 목적을 위해 [배달 못 한 편지 큐][sb-dead-letter-queue] 기능을 제공하는 Azure Service Bus 메시지 큐를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-303">Consider using Azure Service Bus Messaging queues, which provides a [dead-letter queue][sb-dead-letter-queue] functionality for this purpose.</span></span>

> [!NOTE]
> <span data-ttu-id="96fac-304">WebJobs를 통해 저장소 큐를 사용하는 경우 WebJobs SDK에서 기본적으로 포이즌 메시지 처리를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-304">If you are using Storage queues with WebJobs, the WebJobs SDK provides built-in poison message handling.</span></span> <span data-ttu-id="96fac-305">[WebJobs SDK를 통해 Azure 큐 저장소를 사용하는 방법][sb-poison-message]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-305">See [How to use Azure queue storage with the WebJobs SDK][sb-poison-message].</span></span>

<span data-ttu-id="96fac-306">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-306">**Diagnostics**.</span></span> <span data-ttu-id="96fac-307">애플리케이션 로깅을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-307">Use application logging.</span></span>

## <a name="redis-cache"></a><span data-ttu-id="96fac-308">Redis Cache</span><span class="sxs-lookup"><span data-stu-id="96fac-308">Redis Cache</span></span>

### <a name="reading-from-the-cache-fails"></a><span data-ttu-id="96fac-309">캐시에서 읽는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-309">Reading from the cache fails.</span></span>

<span data-ttu-id="96fac-310">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-310">**Detection**.</span></span> <span data-ttu-id="96fac-311">`StackExchange.Redis.RedisConnectionException`을 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-311">Catch `StackExchange.Redis.RedisConnectionException`.</span></span>

<span data-ttu-id="96fac-312">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-312">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-313">일시적 장애 시 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-313">Retry on transient failures.</span></span> <span data-ttu-id="96fac-314">Azure Redis Cache는 기본 제공 다시 시도를 지원합니다. [Redis 캐시 다시 시도 지침][redis-retry]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-314">Azure Redis cache supports built-in retry through See [Redis Cache retry guidelines][redis-retry].</span></span>
2. <span data-ttu-id="96fac-315">일시적이지 않은 장애를 캐시 누락으로 처리하고 원래 데이터 원본으로 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-315">Treat non-transient failures as a cache miss, and fall back to the original data source.</span></span>

<span data-ttu-id="96fac-316">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-316">**Diagnostics**.</span></span> <span data-ttu-id="96fac-317">[Redis Cache 진단][redis-monitor]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-317">Use [Redis Cache diagnostics][redis-monitor].</span></span>

### <a name="writing-to-the-cache-fails"></a><span data-ttu-id="96fac-318">캐시에 쓰는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-318">Writing to the cache fails.</span></span>

<span data-ttu-id="96fac-319">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-319">**Detection**.</span></span> <span data-ttu-id="96fac-320">`StackExchange.Redis.RedisConnectionException`을 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-320">Catch `StackExchange.Redis.RedisConnectionException`.</span></span>

<span data-ttu-id="96fac-321">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-321">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-322">일시적 장애 시 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-322">Retry on transient failures.</span></span> <span data-ttu-id="96fac-323">Azure Redis Cache는 기본 제공 다시 시도를 지원합니다. [Redis 캐시 다시 시도 지침][redis-retry]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-323">Azure Redis cache supports built-in retry through See [Redis Cache retry guidelines][redis-retry].</span></span>
2. <span data-ttu-id="96fac-324">오류가 일시적이지 않으면 이를 무시하고 나중에 다른 트랜잭션에서 캐시에 쓰도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-324">If the error is non-transient, ignore it and let other transactions write to the cache later.</span></span>

<span data-ttu-id="96fac-325">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-325">**Diagnostics**.</span></span> <span data-ttu-id="96fac-326">[Redis Cache 진단][redis-monitor]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-326">Use [Redis Cache diagnostics][redis-monitor].</span></span>

## <a name="sql-database"></a><span data-ttu-id="96fac-327">SQL Database</span><span class="sxs-lookup"><span data-stu-id="96fac-327">SQL Database</span></span>

### <a name="cannot-connect-to-the-database-in-the-primary-region"></a><span data-ttu-id="96fac-328">주 지역의 데이터베이스에 연결할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-328">Cannot connect to the database in the primary region.</span></span>

<span data-ttu-id="96fac-329">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-329">**Detection**.</span></span> <span data-ttu-id="96fac-330">연결이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-330">Connection fails.</span></span>

<span data-ttu-id="96fac-331">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-331">**Recovery:**</span></span>

<span data-ttu-id="96fac-332">필수 조건: 활성 지역 복제를 위한 데이터베이스를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-332">Prerequisite: The database must be configured for active geo-replication.</span></span> <span data-ttu-id="96fac-333">[SQL Database 활성 지역 복제][sql-db-replication]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-333">See [SQL Database Active Geo-Replication][sql-db-replication].</span></span>

- <span data-ttu-id="96fac-334">쿼리의 경우 보조 복제본에서 읽습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-334">For queries, read from a secondary replica.</span></span>
- <span data-ttu-id="96fac-335">삽입 및 업데이트의 경우 수동으로 보조 복제본에 장애 조치합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-335">For inserts and updates, manually fail over to a secondary replica.</span></span> <span data-ttu-id="96fac-336">[Azure SQL Database에 대해 계획되거나 계획되지 않은 장애 조치 시작][sql-db-failover]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-336">See [Initiate a planned or unplanned failover for Azure SQL Database][sql-db-failover].</span></span>

<span data-ttu-id="96fac-337">복제본은 다른 연결 문자열을 사용하므로 애플리케이션에서 연결 문자열을 업데이트해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-337">The replica uses a different connection string, so you will need to update the connection string in your application.</span></span>

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a><span data-ttu-id="96fac-338">클라이언트에 연결 풀의 연결이 부족합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-338">Client runs out of connections in the connection pool.</span></span>

<span data-ttu-id="96fac-339">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-339">**Detection**.</span></span> <span data-ttu-id="96fac-340">`System.InvalidOperationException` 오류를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-340">Catch `System.InvalidOperationException` errors.</span></span>

<span data-ttu-id="96fac-341">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-341">**Recovery:**</span></span>

- <span data-ttu-id="96fac-342">작업을 다시 시도하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-342">Retry the operation.</span></span>
- <span data-ttu-id="96fac-343">완화 계획으로, 각 사용 사례에 대한 연결 풀을 격리하여 하나의 사용 사례에서 모든 연결을 점유할 수 없도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-343">As a mitigation plan, isolate the connection pools for each use case, so that one use case can't dominate all the connections.</span></span>
- <span data-ttu-id="96fac-344">최대 연결 풀 수를 늘립니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-344">Increase the maximum connection pools.</span></span>

<span data-ttu-id="96fac-345">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-345">**Diagnostics**.</span></span> <span data-ttu-id="96fac-346">애플리케이션 로그.</span><span class="sxs-lookup"><span data-stu-id="96fac-346">Application logs.</span></span>

### <a name="database-connection-limit-is-reached"></a><span data-ttu-id="96fac-347">데이터베이스 연결 제한에 도달했습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-347">Database connection limit is reached.</span></span>

<span data-ttu-id="96fac-348">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-348">**Detection**.</span></span> <span data-ttu-id="96fac-349">Azure SQL Database는 동시 작업자, 로그인 및 세션의 수를 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-349">Azure SQL Database limits the number of concurrent workers, logins, and sessions.</span></span> <span data-ttu-id="96fac-350">이 제한은 서비스 계층에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-350">The limits depend on the service tier.</span></span> <span data-ttu-id="96fac-351">자세한 내용은 [Azure SQL Database 리소스 제한][sql-db-limits]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-351">For more information, see [Azure SQL Database resource limits][sql-db-limits].</span></span>

<span data-ttu-id="96fac-352">이러한 오류를 감지하려면 `System.Data.SqlClient.SqlException`을 catch하고 SQL 오류 코드에서 `SqlException.Number` 값을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-352">To detect these errors, catch `System.Data.SqlClient.SqlException` and check the value of `SqlException.Number` for the SQL error code.</span></span> <span data-ttu-id="96fac-353">관련 오류 코드 목록은 [SQL Database 클라이언트 애플리케이션에 대한 SQL 오류 코드: 데이터베이스 연결 오류 및 기타 문제][sql-db-errors]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-353">For a list of relevant error codes, see [SQL error codes for SQL Database client applications: Database connection error and other issues][sql-db-errors].</span></span>

<span data-ttu-id="96fac-354">**복구**.</span><span class="sxs-lookup"><span data-stu-id="96fac-354">**Recovery**.</span></span> <span data-ttu-id="96fac-355">이러한 오류는 일시적인 것으로 간주되므로 다시 시도하면 문제가 해결될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-355">These errors are considered transient, so retrying may resolve the issue.</span></span> <span data-ttu-id="96fac-356">이러한 오류가 지속적으로 발생하면 데이터베이스 크기를 조정하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-356">If you consistently hit these errors, consider scaling the database.</span></span>

<span data-ttu-id="96fac-357">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-357">**Diagnostics**.</span></span> <span data-ttu-id="96fac-358">[sys.event_log][sys.event_log] 쿼리에서 성공적인 데이터베이스 연결, 연결 실패 및 교착 상태를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-358">- The [sys.event_log][sys.event_log] query returns successful database connections, connection failures, and deadlocks.</span></span>

- <span data-ttu-id="96fac-359">실패한 연결에 대한 [경고 규칙][azure-alerts]을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-359">Create an [alert rule][azure-alerts] for failed connections.</span></span>
- <span data-ttu-id="96fac-360">[SQL Database 감사][sql-db-audit]를 사용하도록 설정하고 실패한 로그인을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-360">Enable [SQL Database auditing][sql-db-audit] and check for failed logins.</span></span>

## <a name="service-bus-messaging"></a><span data-ttu-id="96fac-361">Service Bus 메시징</span><span class="sxs-lookup"><span data-stu-id="96fac-361">Service Bus Messaging</span></span>

### <a name="reading-a-message-from-a-service-bus-queue-fails"></a><span data-ttu-id="96fac-362">Service Bus 큐에서 메시지를 읽는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-362">Reading a message from a Service Bus queue fails.</span></span>

<span data-ttu-id="96fac-363">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-363">**Detection**.</span></span> <span data-ttu-id="96fac-364">클라이언트 SDK에서 예외를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-364">Catch exceptions from the client SDK.</span></span> <span data-ttu-id="96fac-365">Service Bus 예외에 대한 기본 클래스는 [MessagingException][sb-messagingexception-class]입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-365">The base class for Service Bus exceptions is [MessagingException][sb-messagingexception-class].</span></span> <span data-ttu-id="96fac-366">일시적인 오류인 경우 `IsTransient` 속성이 true입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-366">If the error is transient, the `IsTransient` property is true.</span></span>

<span data-ttu-id="96fac-367">자세한 내용은 [Service Bus 메시징 예외][sb-messaging-exceptions]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-367">For more information, see [Service Bus messaging exceptions][sb-messaging-exceptions].</span></span>

<span data-ttu-id="96fac-368">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-368">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-369">일시적 장애 시 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-369">Retry on transient failures.</span></span> <span data-ttu-id="96fac-370">[Service Bus 다시 시도 지침][sb-retry]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-370">See [Service Bus retry guidelines][sb-retry].</span></span>
2. <span data-ttu-id="96fac-371">받는 사람에게 배달할 수 없는 메시지는 *배달 못 한 편지 큐*에 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-371">Messages that cannot be delivered to any receiver are placed in a *dead-letter queue*.</span></span> <span data-ttu-id="96fac-372">이 큐를 사용하여 받지 못한 메시지를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-372">Use this queue to see which messages could not be received.</span></span> <span data-ttu-id="96fac-373">배달 못 한 편지 큐에 대한 자동 정리는 지원되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-373">There is no automatic cleanup of the dead-letter queue.</span></span> <span data-ttu-id="96fac-374">메시지는 명시적으로 검색할 때까지 이 큐에 남아 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-374">Messages remain there until you explicitly retrieve them.</span></span> <span data-ttu-id="96fac-375">[Service Bus 배달 못 한 편지 큐의 개요][sb-dead-letter-queue]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-375">See [Overview of Service Bus dead-letter queues][sb-dead-letter-queue].</span></span>

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a><span data-ttu-id="96fac-376">Service Bus 큐에 메시지를 쓰는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-376">Writing a message to a Service Bus queue fails.</span></span>

<span data-ttu-id="96fac-377">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-377">**Detection**.</span></span> <span data-ttu-id="96fac-378">클라이언트 SDK에서 예외를 catch합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-378">Catch exceptions from the client SDK.</span></span> <span data-ttu-id="96fac-379">Service Bus 예외에 대한 기본 클래스는 [MessagingException][sb-messagingexception-class]입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-379">The base class for Service Bus exceptions is [MessagingException][sb-messagingexception-class].</span></span> <span data-ttu-id="96fac-380">일시적인 오류인 경우 `IsTransient` 속성이 true입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-380">If the error is transient, the `IsTransient` property is true.</span></span>

<span data-ttu-id="96fac-381">자세한 내용은 [Service Bus 메시징 예외][sb-messaging-exceptions]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-381">For more information, see [Service Bus messaging exceptions][sb-messaging-exceptions].</span></span>

<span data-ttu-id="96fac-382">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-382">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-383">일시적인 오류 후에 Service Bus 클라이언트에서 자동으로 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-383">The Service Bus client automatically retries after transient errors.</span></span> <span data-ttu-id="96fac-384">기본적으로 지수는 백오프를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-384">By default, it uses exponential back-off.</span></span> <span data-ttu-id="96fac-385">최대 다시 시도 횟수 또는 최대 제한 시간 후에 클라이언트에서 예외를 throw합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-385">After the maximum retry count or maximum timeout period, the client throws an exception.</span></span> <span data-ttu-id="96fac-386">자세한 내용은 [Service Bus 다시 시도 지침][sb-retry]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-386">For more information, see [Service Bus retry guidelines][sb-retry].</span></span>
2. <span data-ttu-id="96fac-387">큐 할당량이 초과되면 클라이언트에서 [QuotaExceededException][QuotaExceededException]을 throw합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-387">If the queue quota is exceeded, the client throws [QuotaExceededException][QuotaExceededException].</span></span> <span data-ttu-id="96fac-388">예외 메시지에서 자세한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-388">The exception message gives more details.</span></span> <span data-ttu-id="96fac-389">다시 시도하기 전에 큐에서 일부 메시지를 배출하고, 회로 차단기 패턴을 사용하여 할당량이 초과되는 동안 계속 다시 시도하지 않도록 방지하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-389">Drain some messages from the queue before retrying, and consider using the Circuit Breaker pattern to avoid continued retries while the quota is exceeded.</span></span> <span data-ttu-id="96fac-390">또한 [BrokeredMessage.TimeToLive] 속성이 너무 높게 설정되지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-390">Also, make sure the [BrokeredMessage.TimeToLive] property is not set too high.</span></span>
3. <span data-ttu-id="96fac-391">지역 내에서 [분할된 큐 또는 토픽][sb-partition]을 사용하여 복원력을 향상시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-391">Within a region, resiliency can be improved by using [partitioned queues or topics][sb-partition].</span></span> <span data-ttu-id="96fac-392">분할되지 않은 큐나 항목은 하나의 메시징 저장소에 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-392">A non-partitioned queue or topic is assigned to one messaging store.</span></span> <span data-ttu-id="96fac-393">이 메시지 저장소를 사용할 수 없게 되면 해당 큐 또는 항목의 모든 작업이 실패하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-393">If this messaging store is unavailable, all operations on that queue or topic will fail.</span></span> <span data-ttu-id="96fac-394">분할된 큐 또는 토픽은 여러 메시지 저장소로 분할됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-394">A partitioned queue or topic is partitioned across multiple messaging stores.</span></span>
4. <span data-ttu-id="96fac-395">추가 복원력을 위해 서로 다른 지역에 두 개의 Service Bus 네임스페이스를 만들고 메시지를 복제합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-395">For additional resiliency, create two Service Bus namespaces in different regions, and replicate the messages.</span></span> <span data-ttu-id="96fac-396">활성 복제 또는 수동 복제를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-396">You can use either active replication or passive replication.</span></span>

    - <span data-ttu-id="96fac-397">활성 복제: 클라이언트에서 모든 메시지를 두 큐로 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-397">Active replication: The client sends every message to both queues.</span></span> <span data-ttu-id="96fac-398">수신기는 두 큐 모두에서 수신 대기합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-398">The receiver listens on both queues.</span></span> <span data-ttu-id="96fac-399">고유 식별자를 사용하여 메시지에 태그를 지정하면 클라이언트에서 중복 메시지를 버릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-399">Tag messages with a unique identifier, so the client can discard duplicate messages.</span></span>
    - <span data-ttu-id="96fac-400">수동 복제: 클라이언트에서 메시지를 한 큐로 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-400">Passive replication: The client sends the message to one queue.</span></span> <span data-ttu-id="96fac-401">오류가 있으면 클라이언트에서 다른 큐로 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-401">If there is an error, the client falls back to the other queue.</span></span> <span data-ttu-id="96fac-402">수신기는 두 큐 모두에서 수신 대기합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-402">The receiver listens on both queues.</span></span> <span data-ttu-id="96fac-403">이 방식은 전송되는 중복 메시지의 수를 줄입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-403">This approach reduces the number of duplicate messages that are sent.</span></span> <span data-ttu-id="96fac-404">그러나 여전히 수신기에서 중복 메시지를 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-404">However, the receiver must still handle duplicate messages.</span></span>

    <span data-ttu-id="96fac-405">자세한 내용은 [GeoReplication 샘플][sb-georeplication-sample] 및 [Service Bus 가동 중단 및 재해로부터 애플리케이션을 보호하기 위한 모범 사례](/azure/service-bus-messaging/service-bus-outages-disasters/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-405">For more information, see [GeoReplication sample][sb-georeplication-sample] and [Best practices for insulating applications against Service Bus outages and disasters](/azure/service-bus-messaging/service-bus-outages-disasters/).</span></span>

### <a name="duplicate-message"></a><span data-ttu-id="96fac-406">메시지가 중복되었습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-406">Duplicate message.</span></span>

<span data-ttu-id="96fac-407">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-407">**Detection**.</span></span> <span data-ttu-id="96fac-408">메시지의 `MessageId` 및 `DeliveryCount` 속성을 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-408">Examine the `MessageId` and `DeliveryCount` properties of the message.</span></span>

<span data-ttu-id="96fac-409">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-409">**Recovery:**</span></span>

- <span data-ttu-id="96fac-410">가능하면 메시지 처리 작업이 idempotent(멱등원)가 되도록 설계합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-410">If possible, design your message processing operations to be idempotent.</span></span> <span data-ttu-id="96fac-411">그렇지 않으면 이미 처리된 메시지의 메시지 ID를 저장하고, 메시지를 처리하기 전에 이 ID를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-411">Otherwise, store message IDs of messages that are already processed, and check the ID before processing a message.</span></span>
- <span data-ttu-id="96fac-412">`RequiresDuplicateDetection`이 true로 설정된 큐를 만들어 중복 검색을 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-412">Enable duplicate detection, by creating the queue with `RequiresDuplicateDetection` set to true.</span></span> <span data-ttu-id="96fac-413">이 설정을 사용하면 Service Bus에서 이전 메시지와 동일한 `MessageId`로 보낸 모든 메시지를 자동으로 삭제합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-413">With this setting, Service Bus automatically deletes any message that is sent with the same `MessageId` as a previous message.</span></span>  <span data-ttu-id="96fac-414">다음 사항에 유의하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-414">Note the following:</span></span>

  - <span data-ttu-id="96fac-415">이 설정은 중복 메시지를 큐에 넣지 않도록 방지합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-415">This setting prevents duplicate messages from being put into the queue.</span></span> <span data-ttu-id="96fac-416">수신기에서 동일한 메시지를 두 번 이상 처리하지 않도록 방지합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-416">It doesn't prevent a receiver from processing the same message more than once.</span></span>
  - <span data-ttu-id="96fac-417">중복 검색에는 기간이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-417">Duplicate detection has a time window.</span></span> <span data-ttu-id="96fac-418">중복 메시지가 이 기간을 초과하여 전송되면 검색되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-418">If a duplicate is sent beyond this window, it won't be detected.</span></span>

<span data-ttu-id="96fac-419">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-419">**Diagnostics**.</span></span> <span data-ttu-id="96fac-420">중복 메시지를 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-420">Log duplicated messages.</span></span>

### <a name="the-application-cant-process-a-particular-message-from-the-queue"></a><span data-ttu-id="96fac-421">애플리케이션에서 큐의 특정 메시지를 처리할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-421">The application can't process a particular message from the queue.</span></span>

<span data-ttu-id="96fac-422">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-422">**Detection**.</span></span> <span data-ttu-id="96fac-423">애플리케이션 특정입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-423">Application specific.</span></span> <span data-ttu-id="96fac-424">예를 들어 메시지에 유효하지 않은 데이터가 있거나 비즈니스 논리가 어떤 이유로 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-424">For example, the message contains invalid data, or the business logic fails for some reason.</span></span>

<span data-ttu-id="96fac-425">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-425">**Recovery:**</span></span>

<span data-ttu-id="96fac-426">고려해야 할 두 가지 장애 모드가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-426">There are two failure modes to consider.</span></span>

- <span data-ttu-id="96fac-427">수신기에서 장애를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-427">The receiver detects the failure.</span></span> <span data-ttu-id="96fac-428">이 경우 메시지가 배달 못 한 편지 큐로 이동됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-428">In this case, move the message to the dead-letter queue.</span></span> <span data-ttu-id="96fac-429">나중에 배달 못 한 편지 큐의 메시지를 검사하는 별도의 프로세스를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-429">Later, run a separate process to examine the messages in the dead-letter queue.</span></span>
- <span data-ttu-id="96fac-430">예를 들어 처리되지 않은 예외로 인해 수신기가 메시지를 처리하는 중에 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-430">The receiver fails in the middle of processing the message &mdash; for example, due to an unhandled exception.</span></span> <span data-ttu-id="96fac-431">이 경우를 처리하려면 `PeekLock` 모드를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-431">To handle this case, use `PeekLock` mode.</span></span> <span data-ttu-id="96fac-432">이 모드에서는 잠금이 만료되는 경우 다른 수신기에서 메시지를 사용할 수 있게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-432">In this mode, if the lock expires, the message becomes available to other receivers.</span></span> <span data-ttu-id="96fac-433">메시지가 최대 배달 횟수 또는 TTL(Time to Live)을 초과하면 메시지는 자동으로 배달 못 한 편지 큐로 이동됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-433">If the message exceeds the maximum delivery count or the time-to-live, the message is automatically moved to the dead-letter queue.</span></span>

<span data-ttu-id="96fac-434">자세한 내용은 [Service Bus 배달 못 한 편지 큐의 개요][sb-dead-letter-queue]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-434">For more information, see [Overview of Service Bus dead-letter queues][sb-dead-letter-queue].</span></span>

<span data-ttu-id="96fac-435">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-435">**Diagnostics**.</span></span> <span data-ttu-id="96fac-436">애플리케이션에서 메시지를 배달 못 한 편지 큐로 이동할 때마다 애플리케이션 로그에 이벤트를 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-436">Whenever the application moves a message to the dead-letter queue, write an event to the application logs.</span></span>

## <a name="service-fabric"></a><span data-ttu-id="96fac-437">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="96fac-437">Service Fabric</span></span>

### <a name="a-request-to-a-service-fails"></a><span data-ttu-id="96fac-438">서비스에 대한 요청이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-438">A request to a service fails.</span></span>

<span data-ttu-id="96fac-439">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-439">**Detection**.</span></span> <span data-ttu-id="96fac-440">서비스에서 오류를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-440">The service returns an error.</span></span>

<span data-ttu-id="96fac-441">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-441">**Recovery:**</span></span>

- <span data-ttu-id="96fac-442">프록시(`ServiceProxy` 또는 `ActorProxy`)를 다시 찾은 다음 서비스/행위자 메서드를 다시 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-442">Locate a proxy again (`ServiceProxy` or `ActorProxy`) and call the service/actor method again.</span></span>
- <span data-ttu-id="96fac-443">**상태 저장 서비스**.</span><span class="sxs-lookup"><span data-stu-id="96fac-443">**Stateful service**.</span></span> <span data-ttu-id="96fac-444">트랜잭션의 신뢰할 수 있는 컬렉션에 대한 작업을 래핑합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-444">Wrap operations on reliable collections in a transaction.</span></span> <span data-ttu-id="96fac-445">오류가 있으면 트랜잭션이 롤백됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-445">If there is an error, the transaction will be rolled back.</span></span> <span data-ttu-id="96fac-446">큐에서 끌어온 요청은 다시 처리됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-446">The request, if pulled from a queue, will be processed again.</span></span>
- <span data-ttu-id="96fac-447">**상태 비저장 서비스**.</span><span class="sxs-lookup"><span data-stu-id="96fac-447">**Stateless service**.</span></span> <span data-ttu-id="96fac-448">서비스에서 외부 저장소에 데이터를 유지하는 경우 모든 작업이 idempotent가 되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-448">If the service persists data to an external store, all operations need to be idempotent.</span></span>

<span data-ttu-id="96fac-449">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-449">**Diagnostics**.</span></span> <span data-ttu-id="96fac-450">애플리케이션 로그</span><span class="sxs-lookup"><span data-stu-id="96fac-450">Application log</span></span>

### <a name="service-fabric-node-is-shut-down"></a><span data-ttu-id="96fac-451">Service Fabric 노드가 종료됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-451">Service Fabric node is shut down.</span></span>

<span data-ttu-id="96fac-452">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-452">**Detection**.</span></span> <span data-ttu-id="96fac-453">취소 토큰이 서비스의 `RunAsync` 메서드로 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-453">A cancellation token is passed to the service's `RunAsync` method.</span></span> <span data-ttu-id="96fac-454">Service Fabric에서 노드를 종료하기 전에 작업을 취소합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-454">Service Fabric cancels the task before shutting down the node.</span></span>

<span data-ttu-id="96fac-455">**복구**.</span><span class="sxs-lookup"><span data-stu-id="96fac-455">**Recovery**.</span></span> <span data-ttu-id="96fac-456">취소 토큰을 사용하여 종료를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-456">Use the cancellation token to detect shutdown.</span></span> <span data-ttu-id="96fac-457">Service Fabric에서 취소를 요청하면 가능한 빨리 모든 작업을 끝내고  `RunAsync`을 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-457">When Service Fabric requests cancellation, finish any work and exit `RunAsync` as quickly as possible.</span></span>

<span data-ttu-id="96fac-458">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-458">**Diagnostics**.</span></span> <span data-ttu-id="96fac-459">애플리케이션 로그 전송 사용</span><span class="sxs-lookup"><span data-stu-id="96fac-459">Application logs</span></span>

## <a name="storage"></a><span data-ttu-id="96fac-460">Storage</span><span class="sxs-lookup"><span data-stu-id="96fac-460">Storage</span></span>

### <a name="writing-data-to-azure-storage-fails"></a><span data-ttu-id="96fac-461">Azure Storage에 데이터를 쓰는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-461">Writing data to Azure Storage fails</span></span>

<span data-ttu-id="96fac-462">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-462">**Detection**.</span></span> <span data-ttu-id="96fac-463">쓰기 작업 중에 클라이언트에서 오류를 받습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-463">The client receives errors when writing.</span></span>

<span data-ttu-id="96fac-464">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-464">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-465">작업을 다시 시도하여 일시적인 장애로부터 복구합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-465">Retry the operation, to recover from transient failures.</span></span> <span data-ttu-id="96fac-466">클라이언트 SDK의 [다시 시도 정책][Storage.RetryPolicies]에서 이 작업을 자동으로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-466">The [retry policy][Storage.RetryPolicies] in the client SDK handles this automatically.</span></span>
2. <span data-ttu-id="96fac-467">저장소의 과부하를 방지하도록 회로 차단기 패턴을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-467">Implement the Circuit Breaker pattern to avoid overwhelming storage.</span></span>
3. <span data-ttu-id="96fac-468">N회의 다시 시도가 실패하면 정상적인 대체(fallback)를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-468">If N retry attempts fail, perform a graceful fallback.</span></span> <span data-ttu-id="96fac-469">예: </span><span class="sxs-lookup"><span data-stu-id="96fac-469">For example:</span></span>

   - <span data-ttu-id="96fac-470">로컬 캐시에 데이터를 저장하고, 나중에 서비스를 사용할 수 있게 되면 저장소에 쓰기를 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-470">Store the data in a local cache, and forward the writes to storage later, when the service becomes available.</span></span>
   - <span data-ttu-id="96fac-471">쓰기 작업이 트랜잭션 범위에 있는 경우 트랜잭션을 보정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-471">If the write action was in a transactional scope, compensate the transaction.</span></span>

<span data-ttu-id="96fac-472">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-472">**Diagnostics**.</span></span> <span data-ttu-id="96fac-473">[저장소 메트릭][storage-metrics]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-473">Use [storage metrics][storage-metrics].</span></span>

### <a name="reading-data-from-azure-storage-fails"></a><span data-ttu-id="96fac-474">Azure Storage에서 데이터를 읽는 작업이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-474">Reading data from Azure Storage fails.</span></span>

<span data-ttu-id="96fac-475">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-475">**Detection**.</span></span> <span data-ttu-id="96fac-476">읽기 작업 중에 클라이언트에서 오류를 받습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-476">The client receives errors when reading.</span></span>

<span data-ttu-id="96fac-477">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-477">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-478">작업을 다시 시도하여 일시적인 장애로부터 복구합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-478">Retry the operation, to recover from transient failures.</span></span> <span data-ttu-id="96fac-479">클라이언트 SDK의 [다시 시도 정책][Storage.RetryPolicies]에서 이 작업을 자동으로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-479">The [retry policy][Storage.RetryPolicies] in the client SDK handles this automatically.</span></span>
2. <span data-ttu-id="96fac-480">RA-GRS 저장소의 경우 기본 엔드포인트에서 읽기 작업이 실패하면 보조 엔드포인트에서 읽기 작업을 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-480">For RA-GRS storage, if reading from the primary endpoint fails, try reading from the secondary endpoint.</span></span> <span data-ttu-id="96fac-481">클라이언트 SDK에서 이 작업을 자동으로 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-481">The client SDK can handle this automatically.</span></span> <span data-ttu-id="96fac-482">[Azure Storage 복제][storage-replication]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-482">See [Azure Storage replication][storage-replication].</span></span>
3. <span data-ttu-id="96fac-483">*N*회의 다시 시도가 실패하면 대체(fallback)를 수행하여 성능을 정상적으로 낮춥니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-483">If *N* retry attempts fail, take a fallback action to degrade gracefully.</span></span> <span data-ttu-id="96fac-484">예를 들어 제품 이미지를 저장소에서 검색할 수 없는 경우 일반적인 자리 표시자 이미지가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-484">For example, if a product image can't be retrieved from storage, show a generic placeholder image.</span></span>

<span data-ttu-id="96fac-485">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-485">**Diagnostics**.</span></span> <span data-ttu-id="96fac-486">[저장소 메트릭][storage-metrics]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-486">Use [storage metrics][storage-metrics].</span></span>

## <a name="virtual-machine"></a><span data-ttu-id="96fac-487">가상 머신</span><span class="sxs-lookup"><span data-stu-id="96fac-487">Virtual machine</span></span>

### <a name="connection-to-a-backend-vm-fails"></a><span data-ttu-id="96fac-488">백 엔드 VM에 대한 연결이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-488">Connection to a backend VM fails.</span></span>

<span data-ttu-id="96fac-489">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-489">**Detection**.</span></span> <span data-ttu-id="96fac-490">네트워크 연결 오류입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-490">Network connection errors.</span></span>

<span data-ttu-id="96fac-491">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-491">**Recovery:**</span></span>

- <span data-ttu-id="96fac-492">부하 분산 장치 뒤에 있는 가용성 집합에 둘 이상의 백 엔드 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-492">Deploy at least two backend VMs in an availability set, behind a load balancer.</span></span>
- <span data-ttu-id="96fac-493">일시적인 연결 오류이면 TCP에서 메시지 전송을 성공적으로 다시 시도하는 경우가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-493">If the connection error is transient, sometimes TCP will successfully retry sending the message.</span></span>
- <span data-ttu-id="96fac-494">애플리케이션에서 다시 시도 정책을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-494">Implement a retry policy in the application.</span></span>
- <span data-ttu-id="96fac-495">영구 오류 또는 일시적이지 않은 오류의 경우 [회로 차단기][circuit-breaker] 패턴을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-495">For persistent or non-transient errors, implement the [Circuit Breaker][circuit-breaker] pattern.</span></span>
- <span data-ttu-id="96fac-496">호출하는 VM에서 네트워크 송신 제한을 초과하면 아웃바운드 큐가 채워집니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-496">If the calling VM exceeds its network egress limit, the outbound queue will fill up.</span></span> <span data-ttu-id="96fac-497">아웃바운드 큐가 일관되게 가득 차면 크기를 확장하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-497">If the outbound queue is consistently full, consider scaling out.</span></span>

<span data-ttu-id="96fac-498">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-498">**Diagnostics**.</span></span> <span data-ttu-id="96fac-499">서비스 경계에서 이벤트를 로깅합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-499">Log events at service boundaries.</span></span>

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a><span data-ttu-id="96fac-500">VM 인스턴스가 사용할 수 없거나 비정상적인 상태가 됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-500">VM instance becomes unavailable or unhealthy.</span></span>

<span data-ttu-id="96fac-501">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-501">**Detection**.</span></span> <span data-ttu-id="96fac-502">VM 인스턴스가 정상인지 여부를 나타내는 Load Balancer [상태 프로브][lb-probe]를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-502">Configure a Load Balancer [health probe][lb-probe] that signals whether the VM instance is healthy.</span></span> <span data-ttu-id="96fac-503">프로브를 통해 중요한 기능이 올바르게 응답하는지 여부를 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-503">The probe should check whether critical functions are responding correctly.</span></span>

<span data-ttu-id="96fac-504">**복구**.</span><span class="sxs-lookup"><span data-stu-id="96fac-504">**Recovery**.</span></span> <span data-ttu-id="96fac-505">각 애플리케이션 계층에 대해 여러 VM 인스턴스를 동일한 가용성 집합에 배치하고, VM 앞에 Load Balancer를 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-505">For each application tier, put multiple VM instances into the same availability set, and place a load balancer in front of the VMs.</span></span> <span data-ttu-id="96fac-506">상태 프로브가 실패하면 Load Balancer에서 비정상 인스턴스에 대한 새 연결 전송을 중지합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-506">If the health probe fails, the Load Balancer stops sending new connections to the unhealthy instance.</span></span>

<span data-ttu-id="96fac-507">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-507">**Diagnostics**.</span></span> <span data-ttu-id="96fac-508">Load Balancer [로그 분석][lb-monitor]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-508">- Use Load Balancer [log analytics][lb-monitor].</span></span>

- <span data-ttu-id="96fac-509">모든 상태 모니터링 엔드포인트를 모니터링하도록 모니터링 시스템을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-509">Configure your monitoring system to monitor all of the health monitoring endpoints.</span></span>

### <a name="operator-accidentally-shuts-down-a-vm"></a><span data-ttu-id="96fac-510">운영자가 실수로 VM을 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-510">Operator accidentally shuts down a VM.</span></span>

<span data-ttu-id="96fac-511">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-511">**Detection**.</span></span> <span data-ttu-id="96fac-512">해당 없음</span><span class="sxs-lookup"><span data-stu-id="96fac-512">N/A</span></span>

<span data-ttu-id="96fac-513">**복구**.</span><span class="sxs-lookup"><span data-stu-id="96fac-513">**Recovery**.</span></span> <span data-ttu-id="96fac-514">`ReadOnly` 수준의 리소스 잠금을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-514">Set a resource lock with `ReadOnly` level.</span></span> <span data-ttu-id="96fac-515">[Azure Resource Manager를 사용하여 리소스 잠그기][rm-locks]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-515">See [Lock resources with Azure Resource Manager][rm-locks].</span></span>

<span data-ttu-id="96fac-516">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-516">**Diagnostics**.</span></span> <span data-ttu-id="96fac-517">[Azure 활동 로그][azure-activity-logs]를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-517">Use [Azure Activity Logs][azure-activity-logs].</span></span>

## <a name="webjobs"></a><span data-ttu-id="96fac-518">웹 작업</span><span class="sxs-lookup"><span data-stu-id="96fac-518">WebJobs</span></span>

### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a><span data-ttu-id="96fac-519">SCM 호스트가 유휴 상태일 때 연속 작업의 실행이 중지됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-519">Continuous job stops running when the SCM host is idle.</span></span>

<span data-ttu-id="96fac-520">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-520">**Detection**.</span></span> <span data-ttu-id="96fac-521">취소 토큰을 WebJob 함수에 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-521">Pass a cancellation token to the WebJob function.</span></span> <span data-ttu-id="96fac-522">자세한 내용은 [정상 종료][web-jobs-shutdown]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-522">For more information, see [Graceful shutdown][web-jobs-shutdown].</span></span>

<span data-ttu-id="96fac-523">**복구**.</span><span class="sxs-lookup"><span data-stu-id="96fac-523">**Recovery**.</span></span> <span data-ttu-id="96fac-524">웹앱에서 `Always On` 설정을 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-524">Enable the `Always On` setting in the web app.</span></span> <span data-ttu-id="96fac-525">자세한 내용은 [WebJobs로 백그라운드 작업 실행][web-jobs]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-525">For more information, see [Run Background tasks with WebJobs][web-jobs].</span></span>

## <a name="application-design"></a><span data-ttu-id="96fac-526">애플리케이션 설계</span><span class="sxs-lookup"><span data-stu-id="96fac-526">Application design</span></span>

### <a name="application-cant-handle-a-spike-in-incoming-requests"></a><span data-ttu-id="96fac-527">애플리케이션에서 들어오는 요청의 스파이크를 처리할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-527">Application can't handle a spike in incoming requests.</span></span>

<span data-ttu-id="96fac-528">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-528">**Detection**.</span></span> <span data-ttu-id="96fac-529">애플리케이션에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-529">Depends on the application.</span></span> <span data-ttu-id="96fac-530">일반적인 증상은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-530">Typical symptoms:</span></span>

- <span data-ttu-id="96fac-531">웹 사이트에서 HTTP 5xx 오류 코드를 반환하기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-531">The website starts returning HTTP 5xx error codes.</span></span>
- <span data-ttu-id="96fac-532">데이터베이스 또는 저장소와 같은 종속 서비스에서 요청을 제한하기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-532">Dependent services, such as database or storage, start to throttle requests.</span></span> <span data-ttu-id="96fac-533">서비스에 따라 HTTP 429(너무 많은 요청)와 같은 HTTP 오류를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-533">Look for HTTP errors such as HTTP 429 (Too Many Requests), depending on the service.</span></span>
- <span data-ttu-id="96fac-534">HTTP 큐 길이가 늘어납니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-534">HTTP queue length grows.</span></span>

<span data-ttu-id="96fac-535">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-535">**Recovery:**</span></span>

- <span data-ttu-id="96fac-536">규모를 확장하여 증가된 부하를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-536">Scale out to handle increased load.</span></span>
- <span data-ttu-id="96fac-537">연속 장애로 인해 전체 애플리케이션이 중단되지 않도록 장애를 완화합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-537">Mitigate failures to avoid having cascading failures disrupt the entire application.</span></span> <span data-ttu-id="96fac-538">완화 전략은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-538">Mitigation strategies include:</span></span>

  - <span data-ttu-id="96fac-539">[제한 패턴][throttling-pattern]을 구현하여 백 엔드 시스템의 과부하를 방지합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-539">Implement the [Throttling pattern][throttling-pattern] to avoid overwhelming backend systems.</span></span>
  - <span data-ttu-id="96fac-540">[큐 기반 부하 평준화][queue-based-load-leveling]를 사용하여 요청을 버퍼링하고 적절한 속도로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-540">Use [queue-based load leveling][queue-based-load-leveling] to buffer requests and process them at an appropriate pace.</span></span>
  - <span data-ttu-id="96fac-541">특정 클라이언트의 우선 순위를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-541">Prioritize certain clients.</span></span> <span data-ttu-id="96fac-542">예를 들어 애플리케이션에 체험 계층과 유료 계층이 있는 경우 체험 계층의 고객은 제한하고 유료 계층의 고객은 제한하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-542">For example, if the application has free and paid tiers, throttle customers on the free tier, but not paid customers.</span></span> <span data-ttu-id="96fac-543">[우선 순위 큐 패턴][priority-queue-pattern]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-543">See [Priority queue pattern][priority-queue-pattern].</span></span>

<span data-ttu-id="96fac-544">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-544">**Diagnostics**.</span></span> <span data-ttu-id="96fac-545">[App Service 진단 로깅][app-service-logging]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-545">Use [App Service diagnostic logging][app-service-logging].</span></span> <span data-ttu-id="96fac-546">[Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights] 또는 [New Relic][new-relic]과 같은 서비스를 사용하면 진단 로그를 이해하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-546">Use a service such as [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights], or [New Relic][new-relic] to help understand the diagnostic logs.</span></span>

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a><span data-ttu-id="96fac-547">워크플로 또는 분산 트랜잭션의 작업 중 하나가 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-547">One of the operations in a workflow or distributed transaction fails.</span></span>

<span data-ttu-id="96fac-548">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-548">**Detection**.</span></span> <span data-ttu-id="96fac-549">*N*회의 다시 시도를 수행한 후에도 계속 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-549">After *N* retry attempts, it still fails.</span></span>

<span data-ttu-id="96fac-550">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-550">**Recovery:**</span></span>

- <span data-ttu-id="96fac-551">완화 계획으로, [스케줄러 에이전트 감독자][scheduler-agent-supervisor] 패턴을 구현하여 전체 워크플로를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-551">As a mitigation plan, implement the [Scheduler Agent Supervisor][scheduler-agent-supervisor] pattern to manage the entire workflow.</span></span>
- <span data-ttu-id="96fac-552">시간 제한 시 다시 시도를 수행하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-552">Don't retry on timeouts.</span></span> <span data-ttu-id="96fac-553">이 오류에 대한 성공률은 낮습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-553">There is a low success rate for this error.</span></span>
- <span data-ttu-id="96fac-554">나중에 다시 시도하기 위해 큐에 작업을 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-554">Queue work, in order to retry later.</span></span>

<span data-ttu-id="96fac-555">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-555">**Diagnostics**.</span></span> <span data-ttu-id="96fac-556">보정 작업을 포함하여 모든 작업(성공 및 실패)을 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-556">Log all operations (successful and failed), including compensating actions.</span></span> <span data-ttu-id="96fac-557">상관 관계 ID를 사용하여 동일한 트랜잭션 내의 모든 작업을 추적할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-557">Use correlation IDs, so that you can track all operations within the same transaction.</span></span>

### <a name="a-call-to-a-remote-service-fails"></a><span data-ttu-id="96fac-558">원격 서비스에 대한 호출이 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-558">A call to a remote service fails.</span></span>

<span data-ttu-id="96fac-559">**검색**.</span><span class="sxs-lookup"><span data-stu-id="96fac-559">**Detection**.</span></span> <span data-ttu-id="96fac-560">HTTP 오류 코드입니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-560">HTTP error code.</span></span>

<span data-ttu-id="96fac-561">**복구:**</span><span class="sxs-lookup"><span data-stu-id="96fac-561">**Recovery:**</span></span>

1. <span data-ttu-id="96fac-562">일시적 장애 시 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-562">Retry on transient failures.</span></span>
2. <span data-ttu-id="96fac-563">*N*회의 시도 후에 호출이 실패하면 대체(fallback) 작업을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-563">If the call fails after *N* attempts, take a fallback action.</span></span> <span data-ttu-id="96fac-564">(애플리케이션 특정입니다.)</span><span class="sxs-lookup"><span data-stu-id="96fac-564">(Application specific.)</span></span>
3. <span data-ttu-id="96fac-565">연속 장애를 방지하도록 [회로 차단기 패턴][circuit-breaker]을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-565">Implement the [Circuit Breaker pattern][circuit-breaker] to avoid cascading failures.</span></span>

<span data-ttu-id="96fac-566">**진단**.</span><span class="sxs-lookup"><span data-stu-id="96fac-566">**Diagnostics**.</span></span> <span data-ttu-id="96fac-567">모든 원격 호출 실패를 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="96fac-567">Log all remote call failures.</span></span>

## <a name="next-steps"></a><span data-ttu-id="96fac-568">다음 단계</span><span class="sxs-lookup"><span data-stu-id="96fac-568">Next steps</span></span>

<span data-ttu-id="96fac-569">FMA 프로세스에 대한 자세한 내용은 [클라우드 서비스 디자인에 의한 복원력][resilience-by-design-pdf](PDF 다운로드)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="96fac-569">For more information about the FMA process, see [Resilience by design for cloud services][resilience-by-design-pdf] (PDF download).</span></span>

<!-- markdownlint-enable MD026 -->

<!-- links -->

[api-management]: /azure/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: https://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache
[resilience-by-design-pdf]: https://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
