---
title: Key Vault를 사용하여 응용 프로그램 암호 보호
description: Key Vault 서비스를 사용하여 응용 프로그램 암호를 저장하는 방법
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: 4cefea7e09cf11cbbc66cdb238c5dea8f700cdad
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902530"
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="04a15-103">Azure Key Vault를 사용하여 응용 프로그램 암호 보호</span><span class="sxs-lookup"><span data-stu-id="04a15-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="04a15-104">[![GitHub](../_images/github.png) 샘플 코드][sample application]</span><span class="sxs-lookup"><span data-stu-id="04a15-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="04a15-105">다음과 같이 민감하고 보호되어야 하는 응용 프로그램 설정을 갖는 것이 일반적입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="04a15-106">데이터베이스 연결 문자열</span><span class="sxs-lookup"><span data-stu-id="04a15-106">Database connection strings</span></span>
* <span data-ttu-id="04a15-107">암호</span><span class="sxs-lookup"><span data-stu-id="04a15-107">Passwords</span></span>
* <span data-ttu-id="04a15-108">암호화 키</span><span class="sxs-lookup"><span data-stu-id="04a15-108">Cryptographic keys</span></span>

<span data-ttu-id="04a15-109">보안 모범 사례로 이 암호를 소스 제어에 저장해서는 안됩니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="04a15-110">소스 코드 저장소가 비공개인 경우에도 쉽게 누출될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="04a15-111">공용으로부터 암호를 유지하는 것 뿐만이 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="04a15-112">대규모 프로젝트에서 프로덕션 암호에 액세스할 수 있는 개발자 및 운영자를 제한할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="04a15-113">(테스트 또는 개발 환경에 대한 설정은 서로 다릅니다.)</span><span class="sxs-lookup"><span data-stu-id="04a15-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="04a15-114">보다 안전한 옵션은 이러한 비밀을 [Azure Key Vault][KeyVault]에 저장하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="04a15-115">키 자격 증명 모음은 암호화 키 및 기타 암호를 관리하기 위한 클라우드 호스티드 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="04a15-116">이 문서는 키 자격 증명 모음을 사용하여 앱에 대한 구성 설정을 저장하는 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-116">This article shows how to use Key Vault to store configuration settings for your app.</span></span>

<span data-ttu-id="04a15-117">[Tailspin Surveys][Surveys] 응용 프로그램에서 다음 설정은 비밀입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="04a15-118">데이터베이스 연결 문자열.</span><span class="sxs-lookup"><span data-stu-id="04a15-118">The database connection string.</span></span>
* <span data-ttu-id="04a15-119">Redis 연결 문자열.</span><span class="sxs-lookup"><span data-stu-id="04a15-119">The Redis connection string.</span></span>
* <span data-ttu-id="04a15-120">웹 응용 프로그램에 대한 클라이언트 암호.</span><span class="sxs-lookup"><span data-stu-id="04a15-120">The client secret for the web application.</span></span>

<span data-ttu-id="04a15-121">설문 조사 응용 프로그램은 다음 위치에서 구성 설정을 로드합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="04a15-122">appsettings.json 파일</span><span class="sxs-lookup"><span data-stu-id="04a15-122">The appsettings.json file</span></span>
* <span data-ttu-id="04a15-123">[사용자 비밀 저장소][user-secrets](개발 환경에만 해당, 테스트용)</span><span class="sxs-lookup"><span data-stu-id="04a15-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="04a15-124">호스팅 환경(Azure 웹앱에서 앱 설정)</span><span class="sxs-lookup"><span data-stu-id="04a15-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="04a15-125">Key Vault(사용 가능한 경우)</span><span class="sxs-lookup"><span data-stu-id="04a15-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="04a15-126">각각은 이전 것을 재정의하므로 키 자격 증명 모음에 저장된 모든 설정이 우선적으로 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="04a15-127">기본적으로 키 자격 증명 모음 구성 공급자는 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="04a15-128">응용 프로그램을 로컬로 실행하는 데 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="04a15-129">프로덕션 배포에 사용하도록 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="04a15-130">시작 시 응용 프로그램은 모든 등록된 구성 공급자에서 설정을 읽고 이를 사용하여 강력한 형식의 옵션 개체를 채웁니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="04a15-131">자세한 내용은 [옵션 및 구성 개체 사용][options]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="04a15-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="04a15-132">설문 조사 앱에서 키 자격 증명 모음 설정</span><span class="sxs-lookup"><span data-stu-id="04a15-132">Setting up Key Vault in the Surveys app</span></span>
<span data-ttu-id="04a15-133">필수 조건:</span><span class="sxs-lookup"><span data-stu-id="04a15-133">Prerequisites:</span></span>

* <span data-ttu-id="04a15-134">[Azure Resource Manager cmdlet][azure-rm-cmdlets]을 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="04a15-135">[Surveys 응용 프로그램 실행][readme]에 설명된 대로 Surveys 응용 프로그램을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="04a15-136">대략적인 단계:</span><span class="sxs-lookup"><span data-stu-id="04a15-136">High-level steps:</span></span>

1. <span data-ttu-id="04a15-137">테넌트에서 관리 사용자를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="04a15-138">클라이언트 인증서를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="04a15-139">키 자격 증명 모음을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-139">Create a key vault.</span></span>
4. <span data-ttu-id="04a15-140">키 자격 증명 모음에 구성 설정을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="04a15-141">키 자격 증명 모음을 사용하는 코드의 주석 처리를 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="04a15-142">응용 프로그램의 사용자 암호를 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="04a15-143">관리 사용자 설정</span><span class="sxs-lookup"><span data-stu-id="04a15-143">Set up an admin user</span></span>
> [!NOTE]
> <span data-ttu-id="04a15-144">키 자격 증명 모음을 만들려면 Azure 구독을 관리할 수 있는 계정을 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="04a15-145">또한 키 자격 증명 모음을 읽을 수 있도록 허용하는 모든 응용 프로그램을 해당 계정과 동일한 테넌트에 등록해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>
> 
> 

<span data-ttu-id="04a15-146">이 단계에서 설문 조사 앱이 등록된 테넌트에서 사용자로 로그인한 상태에서 키 자격 증명 모음을 만들 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="04a15-147">Surveys 응용 프로그램이 등록된 Azure AD 테넌트 내에서 관리자 사용자를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="04a15-148">[Azure Portal][azure-portal]에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="04a15-149">응용 프로그램이 등록된 Azure AD 테넌트를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="04a15-150">**추가 서비스** > **보안 + ID** > **Azure Active Directory** > **사용자 및 그룹** > **모든 사용자**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="04a15-151">포털의 맨 위에서 **새 사용자**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="04a15-152">필드를 채우고 해당 사용자를 **전역 관리자** 디렉터리 역할에 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="04a15-153">**만들기**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-153">Click **Create**.</span></span>

![전역 관리자 사용자](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="04a15-155">이제 이 사용자를 구독 소유자로 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="04a15-156">허브 메뉴에서 **구독**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="04a15-157">관리자가 액세스할 구독을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-157">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="04a15-158">구독 블레이드에서 **액세스 제어(IAM)** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-158">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="04a15-159">**추가**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-159">Click **Add**.</span></span>
4. <span data-ttu-id="04a15-160">**역할** 아래에서 **소유자**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-160">Under **Role**, select **Owner**.</span></span>
5. <span data-ttu-id="04a15-161">소유자로 추가할 사용자의 메일 주소를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-161">Type the email address of the user you want to add as owner.</span></span>
6. <span data-ttu-id="04a15-162">사용자를 선택하고 **저장**을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-162">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="04a15-163">클라이언트 인증서 설정</span><span class="sxs-lookup"><span data-stu-id="04a15-163">Set up a client certificate</span></span>
1. <span data-ttu-id="04a15-164">다음과 같이 PowerShell 스크립트 [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault]을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-164">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="04a15-165">`Subject` 매개 변수의 경우 "surveysapp"과 같은 이름을 임의로 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-165">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="04a15-166">스크립트는 자체 서명된 인증서를 생성하고 "현재 사용자/개인" 인증서 저장소에 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-166">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="04a15-167">스크립트의 출력은 JSON 조각입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-167">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="04a15-168">이 값을 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-168">Copy this value.</span></span>

2. <span data-ttu-id="04a15-169">[Azure Portal][azure-portal]에서 포털의 오른쪽 위에 있는 사용자 계정을 선택하여 Surveys 응용 프로그램이 등록된 디렉터리로 전환합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-169">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="04a15-170">**Azure Active Directory** > **앱 등록** > 설문 조사를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-170">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4.  <span data-ttu-id="04a15-171">**매니페스트**, **편집**을 차례로 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-171">Click **Manifest** and then **Edit**.</span></span>

5.  <span data-ttu-id="04a15-172">스크립트의 출력을 `keyCredentials` 속성에 붙여 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-172">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="04a15-173">다음과 유사하게 나타납니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-173">It should look similar to the following:</span></span>
        
    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```          

6. <span data-ttu-id="04a15-174">**저장**을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-174">Click **Save**.</span></span>  

7. <span data-ttu-id="04a15-175">3-6단계를 반복하여 동일한 JSON 조각을 Web API(Surveys.WebAPI)의 응용 프로그램 매니페스트에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-175">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="04a15-176">PowerShell 창에서 다음 명령을 실행하여 인증서의 지문을 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-176">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>
   
    ```
    certutil -store -user my [subject]
    ```
    
    <span data-ttu-id="04a15-177">`[subject]`에는 PowerShell 스크립트에서 Subject에 대해 지정한 값을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-177">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="04a15-178">지문은 "Cert Hash(sha1)" 아래에 나열됩니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-178">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="04a15-179">이 값을 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-179">Copy this value.</span></span> <span data-ttu-id="04a15-180">나중에 지문을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-180">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="04a15-181">키 자격 증명 모음 만들기</span><span class="sxs-lookup"><span data-stu-id="04a15-181">Create a key vault</span></span>
1. <span data-ttu-id="04a15-182">다음과 같이 PowerShell 스크립트 [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault]을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-182">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    <span data-ttu-id="04a15-183">자격 증명을 묻는 메시지가 나타나면 이전에 만든 Azure AD 사용자로 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-183">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="04a15-184">스크립트는 해당 리소스 그룹 내에 새 리소스 그룹 및 새 키 자격 증명 모음을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-184">The script creates a new resource group, and a new key vault within that resource group.</span></span> 
   
2. <span data-ttu-id="04a15-185">다음과 같이 SetupKeyVault.ps를 다시 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-185">Run SetupKeyVault.ps again as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    <span data-ttu-id="04a15-186">다음 매개 변수 값을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-186">Set the following parameter values:</span></span>
   
       * <span data-ttu-id="04a15-187">키 자격 증명 모음 이름 = 이전 단계에서 키 자격 증명 모음에 제공한 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-187">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="04a15-188">Surveys 앱 ID = Surveys 웹 응용 프로그램의 응용 프로그램 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-188">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="04a15-189">Surveys.WebApi 앱 ID = Surveys.WebAPI 응용 프로그램의 응용 프로그램 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-189">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>
         
    <span data-ttu-id="04a15-190">예제:</span><span class="sxs-lookup"><span data-stu-id="04a15-190">Example:</span></span>
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    <span data-ttu-id="04a15-191">이 스크립트는 웹앱 및 웹 API에 사용자 키 자격 증명 모음에서 암호를 검색할 수 있는 권한을 부여합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-191">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="04a15-192">이에 대한 설명은 [Azure Key Vault 시작](/azure/key-vault/key-vault-get-started/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="04a15-192">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="04a15-193">키 자격 증명 모음에 구성 설정 추가</span><span class="sxs-lookup"><span data-stu-id="04a15-193">Add configuration settings to your key vault</span></span>
1. <span data-ttu-id="04a15-194">다음과 같이 SetupKeyVault.ps를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-194">Run SetupKeyVault.ps as follows::</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    <span data-ttu-id="04a15-195">여기서,</span><span class="sxs-lookup"><span data-stu-id="04a15-195">where</span></span>
   
   * <span data-ttu-id="04a15-196">키 자격 증명 모음 이름 = 이전 단계에서 키 자격 증명 모음에 제공한 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-196">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="04a15-197">Redis DNS 이름 = Redis 캐시 인스턴스의 DNS 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-197">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="04a15-198">Redis 액세스 키 = Redis 캐시 인스턴스에 대한 액세스 키입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-198">Redis access key = The access key for your Redis cache instance.</span></span>
     
2. <span data-ttu-id="04a15-199">이 시점에서 키 자격 증명 모음에 암호를 성공적으로 저장했는지 여부를 테스트하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-199">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="04a15-200">다음 PowerShell 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-200">Run the following PowerShell command:</span></span>
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="04a15-201">SetupKeyVault.ps를 다시 실행하여 데이터베이스 연결 문자열을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-201">Run SetupKeyVault.ps again to add the database connection string:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    <span data-ttu-id="04a15-202">여기에서 `<<DB connection string>>` 은(는) 데이터베이스 연결 문자열의 값입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-202">where `<<DB connection string>>` is the value of the database connection string.</span></span>
   
    <span data-ttu-id="04a15-203">로컬 데이터베이스로 테스트하기 위해 Tailspin.Surveys.Web/appsettings.json 파일에서 연결 문자열을 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-203">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="04a15-204">이 경우 이중 백슬래시('\\\\')를 단일 백슬래시로 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-204">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="04a15-205">이중 백슬래시는 JSON 파일에서 이스케이프 문자입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-205">The double backslash is an escape character in the JSON file.</span></span>
   
    <span data-ttu-id="04a15-206">예제:</span><span class="sxs-lookup"><span data-stu-id="04a15-206">Example:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="04a15-207">키 자격 증명 모음을 사용하는 코드의 주석 처리 제거</span><span class="sxs-lookup"><span data-stu-id="04a15-207">Uncomment the code that enables Key Vault</span></span>
1. <span data-ttu-id="04a15-208">Tailspin.Surveys 솔루션을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-208">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="04a15-209">Tailspin.Surveys.Web/Startup.cs에서 다음 코드 블록을 찾아 주석 처리를 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-209">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="04a15-210">Tailspin.Surveys.Web/Startup.cs에서 `ICredentialService`를 등록하는 코드를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-210">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="04a15-211">`CertificateCredentialService`를 사용하는 줄의 주석 처리를 제거하고 `ClientCredentialService`를 사용하는 줄을 주석으로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-211">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    <span data-ttu-id="04a15-212">이렇게 변경하면 웹앱이 [클라이언트 어설션][client-assertion]을 사용하여 OAuth 액세스 토큰을 가져올 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-212">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="04a15-213">클라이언트 어설션을 사용하는 경우 OAuth 클라이언트 암호가 필요 없습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-213">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="04a15-214">또는 클라이언트 암호를 키 자격 증명 모음에 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-214">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="04a15-215">그러나 키 자격 증명 모음 및 클라이언트 어설션 모두에서 클라이언트 인증서를 사용하므로 키 자격 증명 모음을 활성화하는 경우 클라이언트 어설션도 활성화하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-215">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="04a15-216">사용자 암호 업데이트</span><span class="sxs-lookup"><span data-stu-id="04a15-216">Update the user secrets</span></span>
<span data-ttu-id="04a15-217">솔루션 탐색기에서 Tailspin.Surveys.Web 프로젝트를 마우스 오른쪽 단추로 클릭하고 **사용자 암호 관리**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-217">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="04a15-218">secrets.json 파일에서 기존 JSON을 삭제하고 다음에 붙여 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-218">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

```json
{
  "AzureAd": {
    "ClientId": "[Surveys web app client ID]",
    "ClientSecret": "[Surveys web app client secret]",
    "PostLogoutRedirectUri": "https://localhost:44300/",
    "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="04a15-219">[대괄호] 안에 있는 항목을 올바른 값으로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-219">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="04a15-220">`AzureAd:ClientId`: 설문 조사 앱의 클라이언트 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-220">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="04a15-221">`AzureAd:ClientSecret`: Azure AD에 Surveys 응용 프로그램을 등록할 때 생성된 키입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-221">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="04a15-222">`AzureAd:WebApiResourceId`: Azure AD에서 Surveys.WebAPI 응용 프로그램을 만들 때 지정한 앱 ID URI입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-222">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="04a15-223">`Asymmetric:CertificateThumbprint`: 클라이언트 인증서를 만들 때 이전에 가져온 인증서 지문입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-223">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="04a15-224">`KeyVault:Name`: 키 자격 증명 모음의 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-224">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="04a15-225">`Asymmetric:ValidationRequired`는 이전에 만든 인증서가 루트 CA(인증 기관)에서 서명되지 않았기 때문에 false입니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-225">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="04a15-226">프로덕션에서는 루트 CA에서 서명된 인증서를 사용하고 `ValidationRequired`를 true로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-226">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>
> 
> 

<span data-ttu-id="04a15-227">업데이트된 secrets.json 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-227">Save the updated secrets.json file.</span></span>

<span data-ttu-id="04a15-228">다음으로 솔루션 탐색기에서 Tailspin.Surveys.WebApi 프로젝트를 마우스 오른쪽 단추로 클릭하고 **사용자 암호 관리**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-228">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="04a15-229">기존 JSON을 삭제하고 다음에 붙여 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-229">Delete the existing JSON and paste in the following:</span></span>

```json
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="04a15-230">[대괄호] 안에 있는 항목의 이름을 바꾸고 secrets.json 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-230">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="04a15-231">웹 API의 경우 설문 조사 응용 프로그램이 아닌 Surveys.WebAPI 응용 프로그램에 대한 클라이언트 ID를 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="04a15-231">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>
> 
> 

<span data-ttu-id="04a15-232">[**다음**][adfs]</span><span class="sxs-lookup"><span data-stu-id="04a15-232">[**Next**][adfs]</span></span>

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: /aspnet/core/security/app-secrets
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
