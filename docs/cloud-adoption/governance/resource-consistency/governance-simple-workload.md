---
title: 'CAF: 단일 워크로드를 위한 거버넌스 디자인'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: 사용자가 간단한 워크로드를 배포할 수 있도록 Azure 거버넌스 컨트롤을 구성하기 위한 지침
author: petertaylor9999
ms.date: 02/11/2019
ms.openlocfilehash: 6bf2f15f706140955df29f7f372068c80ce1ad21
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640586"
---
# <a name="governance-design-for-a-simple-workload"></a>단일 워크로드를 위한 거버넌스 디자인

이 지침의 목적은 단일 팀 및 간단한 워크로드를 지원하기 위해 Azure에서 리소스 거버넌스 모델을 디자인하는 프로세스를 알아보는 데 있습니다. 가상의 거버넌스 요구 사항 세트를 살펴본 다음 해당 요구 사항을 충족하는 몇 가지 예제 구현을 수행합니다.

기초 채택 단계에서는 간단한 워크로드를 Azure에 배포하는 것이 목표입니다. 그러면 다음과 같은 요구 사항이 발생합니다.

* 간단한 워크로드를 배포하고 유지 관리하는 작업을 담당하는 단일 **워크로드 소유자**의 ID 관리 워크로드 소유자에게는 ID 관리 시스템의 다른 사용자에게 이러한 권한을 위임하는 사용 권한뿐만 아니라 리소스를 만들고, 읽고, 업데이트하고, 삭제할 수 있는 사용 권한이 필요합니다.
* 간단한 워크로드의 모든 리소스를 단일 관리 단위로 관리합니다.

## <a name="licensing-azure"></a>Azure 라이선싱

거버넌스 모델 디자인을 시작하려면 Azure의 사용이 허가되는 방식을 파악해야 합니다. Azure 라이선스와 연결된 관리자 계정에 모든 Azure 리소스에 대한 가장 높은 수준의 액세스 권한이 있기 때문입니다. 이러한 관리 계정은 거버넌스 모델의 기본을 형성합니다.  

> [!NOTE]
> 조직에 Azure가 포함되지 않은 기존 [Microsoft 기업계약](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx)이 있는 경우 선불 현금 약정 금액을 만들어서 Azure를 추가할 수 있습니다. 자세한 내용은 [엔터프라이즈용 Azure 라이선스](https://azure.microsoft.com/pricing/enterprise-agreement/)를 참조하세요.

Azure를 조직의 기업계약에 추가하는 경우 조직에는 **Azure 계정**을 만들라는 메시지가 표시됩니다. 계정 생성 프로세스 중에 **Azure 계정 소유자**뿐만 아니라 **전역 관리자** 계정을 사용하는 Azure AD(Azure Active Directory) 테넌트가 생성되었습니다. Azure AD 테넌트는 Azure AD의 안전한 전용 인스턴스를 나타내는 논리적 구문입니다.

![Azure 계정 관리자 및 Azure AD 전역 관리자 권한이 있는 Azure 계정](../../_images/governance-3-0.png)
*그림 1. Azure 계정 관리자 및 Azure AD 전역 관리자가 있는 Azure 계정.*

## <a name="identity-management"></a>ID 관리

Azure에서는 [Azure AD](/azure/active-directory)를 신뢰하여 사용자를 인증하고 사용자에게 리소스에 대한 액세스 권한을 부여합니다. 따라서 Azure AD는 ID 관리 시스템입니다. Azure AD 전역 관리자에게는 가장 높은 수준의 사용 권한이 있으며 사용자 생성 및 사용 권한 할당을 비롯하여 ID와 관련된 모든 작업을 수행할 수 있습니다.

요구 사항은 간단한 워크로드를 배포하고 유지 관리하는 작업을 담당하는 단일 **워크로드 소유자**의 ID 관리입니다. 워크로드 소유자에게는 ID 관리 시스템의 다른 사용자에게 이러한 권한을 위임하는 사용 권한뿐만 아니라 리소스를 만들고, 읽고, 업데이트하고, 삭제할 수 있는 사용 권한이 필요합니다.

이 Azure AD 전역 관리자는 **워크로드 소유자**에 대해 **워크로드 소유자** 계정을 만듭니다.

![Azure AD 전역 관리자는 워크로드 소유자 계정을 만듭니다.](../../_images/governance-1-2.png)
*그림 2. Azure AD 전역 관리자는 워크로드 소유 사용자 계정을 만듭니다.*

이 사용자가 **구독**에 추가될 때까지 리소스 액세스 권한을 할당할 수 없습니다. 따라서 다음 두 섹션에서 해당 작업을 수행합니다.

## <a name="resource-management-scope"></a>리소스 관리 범위

조직에서 배포한 리소스 수가 증가함에 따라 해당 리소스를 관리하는 복잡성도 증가합니다. Azure는 논리 컨테이너 계층 구조를 구현하여 조직이 **범위**라고도 하는 다양한 수준의 세분성으로 그룹의 리소스를 관리할 수 있도록 합니다.

상위 수준의 리소스 관리 범위는 **구독** 수준입니다. 구독은 Azure **계정 소유자**에 의해 성성됩니다. 이 사용자는 재정 약정을 설정하고 구독과 연결된 모든 Azure 리소스에 대해 요금을 지불해야 합니다.

![Azure 계정 소유자가 구독을 만듭니다.](../../_images/governance-1-3.png)
*그림 3. Azure 계정 소유자가 구독을 만듭니다.*

구독을 만들면 Azure **계정 소유자**는 구독과 Azure AD 테넌트를 연결하고 이 Azure AD 테넌트는 사용자를 인증하고 권한을 부여하는 데 사용됩니다.

![Azure 계정 소유자는 구독과 Azure AD 테넌트를 연결합니다.](../../_images/governance-1-4.png)
*그림 4. Azure 계정 소유자는 구독과 Azure AD 테넌트를 연결합니다.*

현재 이 구독과 연결된 사용자가 없다는 것을 알 수 있습니다. 즉, 누구에게도 리소스를 관리할 사용 권한이 없습니다. 실제로 **계정 소유자**는 구독 소유자이며 구독의 리소스에 대해 조치를 취할 수 있는 사용 권한이 있습니다. 그러나 실용적인 용어로 **계정 소유자**는 조직의 재무 담당자 이상이며 이러한 리소스를 만들고, 읽고, 업데이트하고, 삭제하는 작업을 담당하지 않습니다. 해당 작업은 **워크로드 소유자**가 수행합니다. 따라서 **워크로드 소유자**를 구독에 추가하고 권한을 할당해야 합니다.

**계정 소유자**가 현재 구독에 **워크로드 소유자**를 추가할 수 있는 사용 권한을 가진 유일한 사용자이므로 **워크로드 소유자**를 구독에 추가합니다.

![Azure 계정 소유자는 **워크로드 소유자**를 구독에 추가합니다.](../../_images/governance-1-5.png)
*그림 5. Azure 계정 소유자는 워크로드 소유자를 구독에 추가합니다.*

Azure **계정 소유자**는 [RBAC(역할 기반 액세스 제어)](/azure/role-based-access-control/) 역할을 할당하여 **워크로드 소유자**에게 사용 권한을 부여합니다. RBAC 역할은 **워크로드 소유자**에게 개별 리소스 종류 또는 일련의 리소스 종류에 대한 사용 권한 집합을 지정합니다.

이 예제에서 **계정 소유자**에게는 [기본 제공 **소유자** 역할](/azure/role-based-access-control/built-in-roles#owner)을 부여했습니다.

![**워크로드 소유자**에게는 기본 제공 소유자 역할이 할당되었습니다.](../../_images/governance-1-6.png)
*그림 6. 워크로드 소유자에게는 기본 제공 소유자 역할이 할당되었습니다.*

기본 제공 **소유자** 역할은 구독 범위에서 **워크로드 소유자**에게 모든 사용 권한을 부여합니다.

> [!IMPORTANT]
> Azure **계정 소유자** 구독과 연결 된 약정을 담당 하지만 **워크 로드 소유자** 동일한 권한을 갖습니다. **계정 소유자**는 **워크로드 소유자**를 신뢰하여 구독 예산 내에 있는 리소스를 배포해야 합니다.

높은 수준의 관리 범위는 **리소스 그룹** 수준입니다. 리소스 그룹은 리소스의 논리 컨테이너입니다. 리소스 그룹 수준에서 적용된 작업은 그룹의 모든 리소스에 적용됩니다. 또한 각 사용자에 대한 사용 권한이 해당 범위에서 명시적으로 변경되지 않으면 높은 수준에서 상속됩니다.

예를 들어 **워크로드 소유자**가 리소스 그룹을 만드는 경우 어떤 상황이 발생하는지를 살펴보겠습니다.

![**워크로드 소유자**는 리소스 그룹을 만듭니다.](../../_images/governance-1-7.png)
*그림 7. 워크로드 소유자는 리소스 그룹을 만들고 리소스 그룹 범위에서 기본 제공 소유자 역할을 상속합니다.*

다시 기본 제공 **소유자** 역할은 리소스 그룹 범위에서 **워크로드 소유자**에게 모든 사용 권한을 부여합니다. 앞에서 설명한 대로 이 역할은 구독 수준에서 상속됩니다. 이 범위에서 다른 역할이 이 사용자에게 할당된 경우 이 범위에만 적용됩니다.

가장 낮은 수준의 관리 범위는 **리소스** 수준입니다. 리소스 수준에서 적용된 작업음 리소스 자체에만 적용됩니다. 또한 다시 한 번 리소스 수준의 사용 권한은 리소스 그룹 범위에서 상속됩니다. 예를 들어 **워크로드 소유자**가 [가상 네트워크](/azure/virtual-network/virtual-networks-overview)를 리소스 그룹에 배포한 경우 어떤 상황이 발생하는지를 살펴보겠습니다.

![**워크로드 소유자**는 리소스를 만듭니다.](../../_images/governance-1-8.png)
*그림 8. 워크로드 소유자는 리소스를 만들고 리소스 범위에서 기본 제공 소유자 역할을 상속합니다.*

**워크로드 소유자**는 리소스 범위에서 소유자 역할을 상속합니다. 즉, 워크로드 소유자에게는 가상 네트워크에 대한 모든 권한이 있습니다.

## <a name="implementing-the-basic-resource-access-management-model"></a>기본 리소스 액세스 관리 모델 구현

이전에 디자인된 거버넌스 모델을 구현하는 방법을 알아보겠습니다.

시작하기 위해 조직에는 Azure 계정이 필요합니다. 조직에 Azure가 포함되지 않은 기존 [Microsoft 기업계약](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx)이 있는 경우 선불 현금 약정 금액을 만들어서 Azure를 추가할 수 있습니다. 자세한 내용은 [엔터프라이즈용 Azure 라이선스](https://azure.microsoft.com/pricing/enterprise-agreement/)를 참조하세요.

Azure 계정의 만들어지면 조직의 사용자를 Azure **계정 소유자**로 지정합니다. 기본적으로 그런 다음, Azure AD(Azure Active Directory) 테넌트가 만들어집니다. Azure **계정 소유자**는 **워크로드 소유자**인 조직의 사용자에 대한 [사용자 계정을 만들](/azure/active-directory/add-users-azure-active-directory)어야 합니다.

다음으로, Azure **계정 소유자**는 [구독을 만들고](/partner-center/create-a-new-subscription) 여기에 [Azure AD 테넌트를 연결](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory)해야 합니다.

[이제 구독을 만들고 Azure AD 테넌트와 연결했으므로 마지막으로, 기본 제공 **소유자** 역할](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal)을 포함하는 구독에 **워크로드 소유자**를 추가할 수 있습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [Azure에 기본 워크로드 배포](../../infrastructure/basic-workload.md)
> [!div class="nextstepaction"]
> [여러 팀에 대한 리소스 액세스 알아보기](governance-multiple-teams.md)
