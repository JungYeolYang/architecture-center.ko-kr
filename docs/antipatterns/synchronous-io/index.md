---
title: 동기 I/O 안티패턴
titleSuffix: Performance antipatterns for cloud apps
description: I/O가 완료하는 동안 호출 스레드를 차단하면 성능이 감소하고 수직 확장성에 영향을 미칠 수 있습니다.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 1b53806b2939a7c44a8b48c9146d5e86c84d9e2e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343563"
---
# <a name="synchronous-io-antipattern"></a>동기 I/O 안티패턴

I/O가 완료하는 동안 호출 스레드를 차단하면 성능이 감소하고 수직 확장성에 영향을 미칠 수 있습니다.

## <a name="problem-description"></a>문제 설명

동기 I/O 작업은 I/O가 완료하는 동안 호출 스레드를 차단합니다. 호출 스레드는 대기 상태로 전환하며 이 간격 동안 유용한 작업을 수행할 수 없어 처리 리소스를 낭비합니다.

I/O의 일반적인 예제는 다음을 포함합니다.

- 데이터 검색 또는 데이터베이스나 임의 유형의 영구 저장소에 데이터 유지.
- 웹 서비스에 요청 보내기.
- 메시지 게시 또는 큐에서 메시지 검색.
- 로컬 파일에 쓰기 또는 로컬 파일에서 읽기.

이런 안티패턴이 발생하는 일반적인 이유는 다음과 같습니다.

- 그러한 패턴이 작업을 수행하는 가장 직관적인 방법처럼 보입니다.
- 애플리케이션이 요청에서 응답을 요구합니다.
- 애플리케이션이 동기 I/O 방법만 제공하는 라이브러리를 사용합니다.
- 외부 라이브러리가 동기 I/O 작업을 내부적으로 사용합니다. 단일 동기 I/O 호출이 전체 호출 체인을 차단할 수 있습니다.

다음 코드는 Azure Blob Storage에 파일을 업로드합니다. 코드 블록이 동기 I/O를 위해 대기하는 두 장소, `CreateIfNotExists` 메서드 및 `UploadFromStream` 메서드가 있습니다.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

다음은 외부 서비스에서의 응답을 위해 대기하는 예제입니다. `GetUserProfile` 메서드는 `UserProfile`을 반환하는 원격 서비스를 호출합니다.

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

이 두 예제에 대한 전체 코드는 [여기][sample-app]서 찾아볼 수 있습니다.

## <a name="how-to-fix-the-problem"></a>문제를 해결하는 방법

동기 I/O 작업을 비동기 작업으로 바꿉니다. 그러면 현재 스레드가 해제되어 차단되지 않고 의미 있는 작업을 계속 수행할 수 있어 계산 리소스 사용률이 개선됩니다. I/O를 비동기로 수행하는 방법은 클라이언트 애플리케이션에서의 예기치 않은 요청 급증을 처리할 때 특히 효율적입니다.

많은 라이브러리는 메서드의 동기 버전과 비동기 버전을 모두 제공합니다. 가능하면 언제나 비동기 버전을 사용합니다. 다음은 Azure Blob Storage에 파일을 업로드하는 앞의 예의 비동기 버전입니다.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

`await` 연산자는 비동기 작업이 수행되는 동안 제어권을 호출 환경에 반환합니다. 이 명령문 뒤의 코드는 비동기 작업이 완료되었을 때 실행하는 연속의 역할을 합니다.

잘 설계된 서비스는 비동기 작업도 제공해야 합니다. 다음은 사용자 프로필을 반환하는 웹 서비스의 비동기 버전입니다. `GetUserProfileAsync` 메서드를 사용하려면 사용자 프로필 서비스의 비동기 버전이 있어야 합니다.

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

작업의 비동기 버전을 제공하지 않는 버전의 경우 선택한 동기 방법의 주위에 비동기 래퍼를 만들 수 있습니다. 이 접근법은 주의해서 따라야 합니다. 이 접근법은 비동기 래퍼를 호출하는 스레드 응답성을 개선할 수 있지만 실제로는 더 많은 리소스를 사용합니다. 추가 스레드가 만들어질 수 있으며 이 스레드가 수행한 작업의 동기화와 연결된 오버헤드가 있습니다. 일부 장단점은 [동기 메서드에 대해 비동기 래퍼를 공개해야 하나요?][async-wrappers] 블로그 게시물에서 설명하고 있습니다.

다음은 동기 메서드 주위의 비동기 래퍼의 예제입니다.

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

이제 호출 코드는 래퍼상에서 대기할 수 있습니다.

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>고려 사항

- 매우 수명이 짧아서 경합을 야기할 가능성이 낮은 I/O 작업은 비동기 작업으로서 더 효율적일 수 있습니다. SSD 드라이브에 있는 작은 파일을 읽는 경우가 그러한 예일 수 있습니다. 작업이 완료될 때 작업을 다른 스레드에 디스패치하고 해당 스레드와 동기화하는 오버헤드가 비동기 I/O의 이점보다 클 수 있습니다. 그러나 이러한 경우는 비교적 드물며 대부분의 I/O 작업은 비동기로 수행해야 합니다.

- I/O 성능이 개선되면 시스템의 다른 부분에 병목이 발생할 수 있습니다. 예를 들어 스레드를 차단 해제하면 동시 공유 리소스 요청이 더 많아지고 그에 따라 리소스 부족 또는 제한이 발생할 수 있습니다. 이러한 상황이 문제가 된다면 경합을 줄이기 위해 웹 서버 수의 규모를 확장하거나 데이터 저장소를 분할해야 할 수 있습니다.

## <a name="how-to-detect-the-problem"></a>문제를 감지하는 방법

사용자 입장에서는 애플리케이션이 응답하지 않거나 주기적으로 중지하는 것으로 보일 수 있습니다. 애플리케이션은 시간 초과 예외 때문에 실패할 수 있습니다. 이러한 장애는 HTTP 500(내부 서버) 오류를 반환할 수도 있습니다. 서버 쪽에서는 들어오는 클라이언트 요청이 스레드가 사용할 수 있게 될 때까지 차단되어 요청 큐 길이가 너무 커질 수 있으며 이는 HTTP 503(서비스를 사용할 수 없음) 오류로 명시됩니다.

다음 단계를 수행하면 문제를 식별하는 데 도움이 될 수 있습니다.

1. 프로덕션 시스템을 모니터링하여 차단된 작업자 스레드가 처리량을 제약하는지 여부를 결정합니다.

2. 요청이 스레드 부족 때문에 차단되는 경우 애플리케이션을 검토하여 I/O를 동기식으로 수행하는 작업이 어느 것인지 결정합니다.

3. 동기 I/O를 수행 중인 각 작업에 대해 제어된 부하 테스트를 수행하여 해당 작업이 시스템 성능에 영향을 미치는지 여부를 알아냅니다.

## <a name="example-diagnosis"></a>예제 진단

다음 섹션에서는 이러한 단계를 앞에서 설명한 애플리케이션 예제에 적용합니다.

### <a name="monitor-web-server-performance"></a>웹 서버 성능 모니터링

Azure 웹 애플리케이션 및 웹 역할의 경우 IIS 웹 서버의 성능을 모니터링하는 것이 좋습니다. 특히 요청 큐 길이에 주의를 기울여 작업량이 많은 기간 동안 요청이 차단되어 사용할 수 있는 스레드를 위해 대기하는지 여부를 확인해야 합니다. Azure 진단을 사용하도록 설정하면 이 정보를 수집할 수 있습니다. 자세한 내용은 다음을 참조하세요.

- [Azure App Service에서 앱 모니터링][web-sites-monitor]
- [Azure 애플리케이션에서 성능 카운터 만들기 및 사용][performance-counters]

애플리케이션을 계측하여 요청이 수용된 후 처리되는 방법을 알아봅니다. 요청 흐름을 추적하면 느리게 실행되는 호출을 수행하여 현재 스레드를 차단하는지 여부를 확인하는 데 도움이 될 수 있습니다. 또한 스레드 프로파일링을 수행하면 차단된 요청을 강조 표시할 수 있습니다.

### <a name="load-test-the-application"></a>애플리케이션 부하 테스트

다음 그래프는 동기 사용자 수가 최대 4000명인 변화하는 부하에서 앞에서 본 동기 `GetUserProfile` 메서드의 성능을 보여 줍니다. 애플리케이션은 Azure Cloud Service 웹 역할에서 실행 중인 ASP.NET 애플리케이션입니다.

![비동기 I/O 작업을 수행하는 샘플 애플리케이션에 대한 성능 차트][sync-performance]

동기 작업은 2초 동안 휴면하고 동기 I/O를 시뮬레이션하도록 하드 코딩되었으므로 최소 응답 시간은 2초를 조금 넘습니다. 부하가 대략 동시 사용자 2500명에 도달하면 평균 응답 시간은 안정기에 접어들지만 초당 요청 분량은 계속 증가합니다. 참고로 이 두 측정값에 대한 비율 크기 조정은 로그값입니다. 초당 요청 수는 이 점과 테스트 종료 시점 간에 두 배가 됩니다.

분리 상태에서는 이 테스트에서 동기 I/O가 문제인지 여부를 반드시 명확히 알 수 있는 것은 아닙니다. 더 큰 부하에서 애플리케이션은 웹 서버가 더 이상 요청을 적시에 처리할 수 없어 클라이언트 애플리케이션이 시간 초과 예외를 수신하게 하는 티핑 포인트에 도달할 수 있습니다.

들어오는 요청은 IIS 웹 서버에 의해 큐에 저장되며 ASP.NET 스레드 풀에서 실행 중인 스레드로 넘겨집니다. 각 작업이 I/O를 동기식으로 수행하므로 스레드는 작업이 완료될 때까지 차단됩니다. 워크로드가 증가함에 따라 결국 스레드 풀의 모든 ASP.NET 스레드가 할당되고 차단됩니다. 그 시점에서 추가로 들어오는 요청은 사용할 수 있는 스레드를 위해 큐에서 대기해야 합니다. 큐 길이가 커짐에 따라 요청이 시간 초과하기 시작합니다.

### <a name="implement-the-solution-and-verify-the-result"></a>솔루션 구현 및 결과 확인

다음 그래프는 코드의 비동기 버전에 대한 부하 테스트에서 나온 결과를 보여 줍니다.

![비동기 I/O 작업을 수행하는 샘플 애플리케이션에 대한 성능 차트][async-performance]

처리량이 훨씬 더 큽니다. 초당 요청 수로 측정했을 때, 이전 테스트와 같은 기간 동안 시스템은 거의 10배 증가한 처리량을 성공적으로 처리합니다. 또한 평균 응답 시간이 비교적 일정하며 이전 테스트보다 약 25배 더 작게 유지됩니다.

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO
[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg
