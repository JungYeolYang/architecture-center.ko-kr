---
title: Azure Resource Manager 템플릿에서 개체를 매개 변수로 사용
description: Azure Resource Manager 템플릿의 기능을 확장하여 개체를 매개 변수로 사용하는 방법을 설명합니다.
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: f0826d8ed1ce446d295ebdacc66d8b0bef0b0dec
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111209"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="6d822-103">Azure Resource Manager 템플릿에서 개체를 매개 변수로 사용</span><span class="sxs-lookup"><span data-stu-id="6d822-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="6d822-104">[Azure Resource Manager 템플릿을 작성][azure-resource-manager-create-template]할 때 템플릿에서 직접 리소스 속성 값을 지정하거나, 배포 중에 매개 변수를 정의하고 값을 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="6d822-105">소규모 배포의 경우에는 각 속성 값에 대해 매개 변수를 사용하는 것이 좋지만, 매개 변수 수는 배포당 255개로 제한됩니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="6d822-106">보다 크고 복잡한 배포가 되면 매개 변수가 부족해질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="6d822-107">이 문제를 해결하는 한 가지 방법은 개체를 값이 아닌 매개 변수로 사용하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="6d822-108">이를 수행하려면 템플릿에서 매개 변수를 정의하고 배포 중에 단일 값 대신 JSON 개체를 지정하세요.</span><span class="sxs-lookup"><span data-stu-id="6d822-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="6d822-109">그런 다음 템플릿에서 [`parameter()` 함수][azure-resource-manager-functions] 및 점 연산자를 사용하여 매개 변수의 하위 속성을 참조합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="6d822-110">가상 네트워크 리소스를 배포하는 예를 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="6d822-111">먼저 템플릿에서 `VNetSettings` 매개 변수를 지정하고 `type`을 `object`로 설정해봅니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```

<span data-ttu-id="6d822-112">그런 다음 `VNetSettings` 개체의 값을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="6d822-113">배포 동안 매개 변수 값을 제공하는 방법을 알아보려면 [Azure Resource Manager 템플릿의 구조 및 구문 이해][azure-resource-manager-authoring-templates]의 **매개 변수** 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6d822-113">To learn how to provide parameter values during deployment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span>

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

<span data-ttu-id="6d822-114">아시다시피 단일 매개 변수는 실제로 3개의 하위 속성, `name`, `addressPrefixes` 및 `subnets`를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="6d822-115">이러한 각 하위 속성은 값 또는 다른 하위 속성을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="6d822-116">그 결과, 단일 매개 변수는 가상 네트워크를 배포하는 데 필요한 모든 값을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="6d822-117">이제 템플릿의 나머지 부분에서 `VNetSettings` 개체가 사용되는 방식을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```

<span data-ttu-id="6d822-118">`VNetSettings` 개체 값은 `parameters()` 함수를 `[]` 배열 인덱서 및 점 연산자와 함께 사용하여 가상 네트워크 리소스에 필요한 속성에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="6d822-119">이 방법은 매개 변수 개체의 값을 리소스에 정적으로 적용하려는 경우에만 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="6d822-120">그러나 배포하는 동안 속성 값의 배열을 동적으로 할당하려는 경우에는 [복사 루프][azure-resource-manager-create-multiple-instances]를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="6d822-121">복사 루프를 사용 하려면 리소스 속성 값의 JSON 배열을 제공하면, 복사 루프가 해당 값을 리소스 속성에 동적으로 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span>

<span data-ttu-id="6d822-122">한 가지 문제점은 동적 방법을 사용하는지를 인식하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="6d822-123">이 문제를 이해하기 위해 속성 값의 일반 배열을 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="6d822-124">이 예제에서 속성 값은 변수에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="6d822-125">여기에는 이름이 각각 `firstProperty` 및 `secondProperty`인 두 배열이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span>

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

<span data-ttu-id="6d822-126">이제 복사 루프를 사용하여 변수에서 속성에 액세스하는 방법을 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

<span data-ttu-id="6d822-127">`copyIndex()` 함수가 복사 루프의 현재 반복을 반환하면, 해당 결과를 동시에 두 배열 각각에 대한 인덱스로 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="6d822-128">이러한 방식은 두 배열의 길이가 같을 때 잘 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="6d822-129">실수를 했거나 두 배열의 길이가 아르면 문제가 발생합니다. 이 경우에는 배포 동안 템플릿의 유효성 검사가 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="6d822-130">모든 속성을 단일 개체에 포함하면 값이 누락되었는지를 훨씬 쉽게 파악할 수 있으므로 이러한 방식으로 이 문제를 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="6d822-131">예를 들어, `propertyObject` 배열의 각 요소가 `firstProperty` 및 `secondProperty` 배열의 합집합인 다른 매개 변수 개체를 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

<span data-ttu-id="6d822-132">배열의 세 번째 요소를 확인할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="6d822-132">Notice the third element in the array?</span></span> <span data-ttu-id="6d822-133">`number` 속성이 없으나, 이러한 방식으로 매개 변수 값을 작성할 때는 누락되었는지를 훨씬 더 쉽게 알 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="6d822-134">복사 루프에 속성 개체 사용</span><span class="sxs-lookup"><span data-stu-id="6d822-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="6d822-135">이 방법은 [직렬 복사 루프][azure-resource-manager-create-multiple]와 함께 사용할 경우(특히 자식 리소스를 배포하는 경우) 훨씬 더 유용해집니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span>

<span data-ttu-id="6d822-136">이를 이해하기 위해 2개의 보안 규칙과 함께 [NSG(네트워크 보안 그룹)][nsg]를 배포하는 템플릿을 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span>

<span data-ttu-id="6d822-137">먼저, 매개 변수를 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="6d822-138">템플릿을 확인하면, `securityRules`라는 배열을 포함하는 `networkSecurityGroupsSettings` 매개 변수를 정의했다는 것을 알 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="6d822-139">이 배열에는 보안 규칙에 대한 다양한 설정을 지정하는 두 가지 JSON 개체가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

<span data-ttu-id="6d822-140">이제 템플릿을 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-140">Now let's take a look at our template.</span></span> <span data-ttu-id="6d822-141">`NSG1`라는 첫 번째 리소스는 NSG를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="6d822-142">`loop-0`이라는 두 번째 리소스는 2개의 함수를 수행합니다. 첫째, NSG에 `dependsOn`하므로 `NSG1`이 완료될 때까지 배포가 시작되지 않으며 순차 루프의 첫 번째 반복입니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="6d822-143">세 번째 리소스는 마지막 예제와 같이 매개 변수 값으로 개체를 사용해서 보안 규칙을 배포하는 중첩된 템플릿입니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],
  "outputs": {}
}
```

<span data-ttu-id="6d822-144">`securityRules` 자식 리소스에서 속성 값을 지정하는 방법을 좀 더 자세히 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="6d822-145">모든 속성이 `parameter()` 기능을 사용해서 참조됩니다. 그러면 점 연산자를 사용하여 반복의 현재 값으로 인덱싱된 `securityRules` 배열을 참조합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="6d822-146">마지막으로 다른 점 연산자를 사용하여 개체의 이름을 참조합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-146">Finally, we use another dot operator to reference the name of the object.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="6d822-147">템플릿 시도</span><span class="sxs-lookup"><span data-stu-id="6d822-147">Try the template</span></span>

<span data-ttu-id="6d822-148">[GitHub][github]에서 예제 템플릿을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-148">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="6d822-149">템플릿을 배포하려면 리포지토리를 복제하고, 다음 [Azure CLI][cli] 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-149">To deploy the template, clone the repo and run the following [Azure CLI][cli] commands:</span></span>

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a><span data-ttu-id="6d822-150">다음 단계</span><span class="sxs-lookup"><span data-stu-id="6d822-150">Next steps</span></span>

- <span data-ttu-id="6d822-151">개체 배열을 반복하고 JSON 스키마로 변환하는 템플릿을 만드는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="6d822-151">Learn how to create a template that iterates through an object array and transforms it into a JSON schema.</span></span> <span data-ttu-id="6d822-152">[Azure Resource Manager 템플릿에서 속성 변환기 및 수집기 구현](./collector.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6d822-152">See [Implement a property transformer and collector in an Azure Resource Manager template](./collector.md)</span></span>

<!-- links -->

[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
