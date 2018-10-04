---
title: Azure의 3D 비디오 렌더링
description: Azure Batch 서비스를 사용하여 Azure에서 원시 HPC 워크로드를 실행합니다.
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 67dc8496656a75eab8f5d0ce45d00f8b1f4ea03f
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428060"
---
# <a name="3d-video-rendering-on-azure"></a>Azure의 3D 비디오 렌더링

3D 렌더링은 공동으로 완료하는 데 상당한 양의 CPU 시간이 필요한 시간 소모적인 프로세스입니다.  단일 머신에서 정적 자산으로부터 비디오 파일을 생성하는 프로세스에는 생성하는 비디오의 길이와 복잡성에 따라 몇 시간 또는 며칠이 걸릴 수 있습니다.  대부분의 회사에서 이러한 작업을 수행하기 위해 비용이 많이 드는 고사양 데스크톱 컴퓨터를 구입하거나 작업을 제출할 수 있는 대형 렌더 팜에 투자합니다.  그러나 Azure Batch를 활용하면 필요할 때 이 기능을 사용할 수 있고, 그렇지 않은 경우 어떠한 자본 투자 없이도 기능 자체를 종료할 수 있습니다.

Batch는 Windows Server 또는 Linux 계산 노드 중 어느 것을 선택하든지 간에 일관된 관리 환경과 작업 예약을 제공합니다. Batch를 사용하면 AutoDesk Maya 및 Blender를 포함하여 기존 Windows 또는 Linux 응용 프로그램을 통해 Azure에서 대규모 렌더 작업을 실행할 수 있습니다.

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오는 다음과 비슷한 사용 사례에 사용하는 것이 좋습니다.

* 3D 모델링
* VFX(Visual FX) 렌더링
* 비디오 코드 변환
* 이미지 처리, 색 보정 및 크기 조정

## <a name="architecture"></a>아키텍처

![Azure Batch를 사용하는 클라우드 원시 HPC 솔루션과 관련된 구성 요소 아키텍처에 대한 개요][architecture]

이 샘플 시나리오에서는 Azure Batch를 사용하는 경우의 워크플로에 대해 설명하며, 데이터 흐름은 다음과 같습니다.

1. 입력 파일과 응용 프로그램을 업로드하여 이러한 파일을 Azure Storage 계정으로 처리합니다.
2. 배치 계정에 계산 노드의 Batch 풀을 만들고, 풀에 워크로드를 실행하는 작업을 만들고, 작업에 태스크를 만듭니다.
3. 입력 파일과 응용 프로그램을 Batch에 다운로드합니다.
4. 태스크 실행 모니터링
5. 태스크 출력을 업로드합니다.
6. 출력 파일을 다운로드합니다.

이 프로세스를 간소화하기 위해 [Maya 및 3ds Max용 Batch 플러그 인][batch-plugins]을 사용할 수도 있습니다.

### <a name="components"></a>구성 요소

Azure Batch는 다음 Azure 기술을 기반으로 합니다.

* [리소스 그룹][resource-groups]은 Azure 리소스에 대한 논리 컨테이너입니다.
* [가상 네트워크][vnet]는 헤드 노드 및 계산 리소스 모두에 사용됩니다.
* [저장소][storage] 계정은 동기화 및 데이터 보존에 사용됩니다.
* [Virtual Machine Scale Sets][vmss]는 CycleCloud에서 계산 리소스에 대해 사용됩니다.

## <a name="considerations"></a>고려 사항

### <a name="machine-sizes-available-for-azure-batch"></a>Azure Batch에 사용 가능한 머신 크기

대부분의 렌더링 고객은 CPU 능력이 높은 리소스를 선택하지만, 가상 머신 확장 집합을 사용하는 다른 워크로드는 VM을 다르게 선택할 수 있으며 다음과 같은 여러 요인에 따라 달라집니다.

* 응용 프로그램이 메모리에 바인딩되어 실행되고 있나요?
* 응용 프로그램에서 GPU를 사용해야 하나요? 
* 당황스러울 정도의 병렬 작업 유형이거나 긴밀하게 결합된 작업에 대한 Infiniband 연결이 필요한가요?
* 계산 노드에서 저장소에 대한 빠른 I/O가 필요한가요?

Azure에는 위의 응용 프로그램 요구 사항을 모두 처리할 수 있는 광범위한 VM 크기가 있으며, 일부는 HPC에만 관련되지만 가장 작은 크기조차도 효과적인 그리드 구현을 제공하는 데 활용할 수 있습니다.

* [HPC VM 크기][compute-hpc] 렌더링의 CPU 바인딩 특성으로 인해 Microsoft는 일반적으로 Azure H 시리즈 VM을 제안합니다.  이러한 VM은 고급 계산 요구 사항에 맞게 특별히 만들어졌으며, 8개 및 16개 코어 vCPU 크기를 사용할 수 있고, DDR4 메모리, SSD 임시 저장소 및 Haswell E5 Intel 기술을 갖추고 있습니다.
* [GPU VM 크기][compute-gpu] GPU 최적화된 VM 크기는 단일 또는 여러 NVIDIA GPU에서 사용할 수 있는 특수한 가상 머신입니다. 이러한 크기는 계산 집약적이며 그래픽 집약적인 시각화 워크로드용으로 설계되었습니다.
* NC, NCv2, NCv3 및 ND 크기는 CUDA 기반 및 OpenCL 기반 응용 프로그램 및 시뮬레이션, AI, 딥 러닝을 포함하여 계산 집약적이고 네트워크 집약적인 응용 프로그램과 알고리즘에 최적화되어 있습니다. NV 크기는 OpenGL 및 DirectX와 같은 프레임워크를 활용하는 원격 시각화, 스트리밍, 게임, 인코딩 및 VDI 시나리오에 맞게 최적화되고 설계되었습니다.
* [메모리 최적화된 VM 크기][compute-memory] 더 많은 메모리가 필요한 경우 메모리 최적화된 VM 크기는 더 높은 메모리 대 CPU 비율을 제공합니다.
* [범용 VM 크기][compute-general] 범용 VM 크기도 사용할 수 있으며, 균형 잡힌 CPU 대 메모리 비율을 제공합니다.

### <a name="alternatives"></a>대안

Azure에서 렌더링 환경을 더 자세히 제어해야 하거나 하이브리드 구현이 필요한 경우, CycleCloud 컴퓨팅은 클라우드에서 IaaS 그리드를 오케스트레이션하는 데 도움을 줄 수 있습니다. Azure Batch와 동일한 기본 Azure 기술을 사용하여 IaaS 그리드를 효율적으로 작성하고 유지 관리합니다. 자세한 내용을 찾고 디자인 원칙을 알아보려면 다음 링크를 사용합니다.

Azure에서 사용할 수 있는 모든 HPC 솔루션에 대한 전체 개요는 [Azure VM을 사용하여 HPC, 일괄 처리 및 큰 계산 솔루션][hpc-alt-solutions] 문서를 참조하세요.

### <a name="availability"></a>가용성

Azure Batch 구성 요소에 대한 모니터링은 다양한 서비스, 도구 및 API를 통해 사용할 수 있습니다. 모니터링에 대해서는 [Batch 솔루션 모니터링][batch-monitor] 문서에서 자세히 설명합니다.

### <a name="scalability"></a>확장성

Azure Batch 계정 내의 풀은 수동 개입을 통해 크기 조정하거나 Azure Batch 메트릭에 기반한 수식을 사용하여 자동으로 크기 조정할 수 있습니다. 크기 조정에 대한 자세한 내용은 [Batch 풀에서 노드의 크기를 조정하는 자동 크기 조정 수식 만들기][batch-scaling] 문서를 참조하세요.

### <a name="security"></a>보안

보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

Azure Batch에는 현재 장애 조치 기능이 없지만, 계획되지 않은 중단이 발생하는 경우 가용성을 보장하기 위해 다음 단계를 사용하는 것이 좋습니다.

* 대체 저장소 계정을 사용하여 Azure Batch 계정을 대체 Azure 위치에 만듭니다.
* 동일한 이름을 사용하여 0개 노드가 할당된 동일한 노드 풀을 만듭니다.
* 응용 프로그램이 만들어지고, 대체 저장소 계정으로 업데이트되었는지 확인합니다.
* 입력 파일을 업로드하고, 대체 Azure Batch 계정에 작업을 제출합니다.

## <a name="deploy-this-scenario"></a>시나리오 배포

### <a name="creating-an-azure-batch-account-and-pools-manually"></a>수동으로 Azure Batch 계정 및 풀 만들기

이 샘플 시나리오는 Azure Batch 작동 방법을 알아보는 데 도움이 되며, Azure Batch Labs를 자신의 고객을 위해 개발할 수 있는 예제 SaaS 솔루션으로 보여 줍니다.

[Azure Batch Masterclass][batch-labs-masterclass]

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-template"></a>Azure Resource Manager 템플릿을 사용하여 샘플 시나리오 배포

템플릿에서 다음과 같이 배포합니다.

* 새 Azure Batch 계정
* 저장소 계정
* 배치 계정과 연결된 노드 풀
* Canonical Ubuntu 이미지를 사용하여 노드 풀에서 A2 v2 VM을 사용하도록 구성됩니다.
* 처음에는 노드 풀에 0개의 VM이 포함되며, VM을 추가하려면 수동으로 크기 조정해야 합니다.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

[Resource Manager 템플릿에 대해 자세히 알아보기][azure-arm-templates]

## <a name="pricing"></a>가격

Azure Batch를 사용하는 데 드는 비용은 풀에 사용되는 VM 크기와 이러한 VM이 할당되고 실행되는 기간에 따라 다르며, Azure Batch 계정을 만드는 데 관련된 비용은 없습니다. 추가 비용이 적용되므로 저장소 및 데이터 송신도 고려해야 합니다.

서로 다른 수의 서버를 사용하여 8시간 내에 완료되는 작업에 청구될 수 있는 비용의 예는 다음과 같습니다.

* 100개 고성능 CPU VM: [예상 비용][hpc-est-high]

  100 x H16m(16개 코어, 225GB RAM, 512GB Premium Storage), 2TB Blob Storage, 1TB 송신

* 50개 고성능 CPU VM: [예상 비용][hpc-est-med]

  50 x H16m(16개 코어, 225GB RAM, 512GB Premium Storage), 2TB Blob Storage, 1TB 송신

* 10개 고성능 CPU VM: [예상 비용][hpc-est-low]
  
  10 x H16m(16개 코어, 225GB RAM, 512GB Premium Storage), 2TB Blob Storage, 1TB 송신

### <a name="low-priority-vm-pricing"></a>낮은 우선 순위 VM 가격 책정

Azure Batch는 노드 풀에서 낮은 우선 순위 VM*도 사용할 수 있도록 지원하므로 잠재적으로 상당한 비용을 절감할 수 있습니다. 표준 VM과 낮은 우선 순위 VM 간의 가격 비교 및 우선 순위가 낮은 VM에 대한 자세한 내용은 [Batch 가격 책정][batch-pricing]을 참조하세요.

\* 특정 응용 프로그램과 워크로드만 낮은 우선 순위 VM에서 실행하는 데 적합하다는 점에 유의하세요.

## <a name="related-resources"></a>관련 리소스

[Azure Batch 개요][batch-overview]

[Azure Batch 설명서][batch-doc]

[Azure Batch에서 컨테이너 사용][batch-containers]

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job