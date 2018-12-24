---
title: 외부 구성 저장소
description: 구성 정보를 애플리케이션 배포 패키지에서 중앙 위치로 이동합니다.
keywords: 디자인 패턴
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
ms.locfileid: "26582787"
---
# <a name="external-configuration-store-pattern"></a>외부 구성 저장소 패턴

[!INCLUDE [header](../_includes/header.md)]

구성 정보를 애플리케이션 배포 패키지에서 중앙 위치로 이동합니다. 이렇게 하면 구성 데이터를 더 쉽게 관리하고 제어하며 구성 데이터를 응용 프로그램과 응용 프로그램 인스턴스에서 공유할 기회를 제공할 수 있습니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

대부분의 응용 프로그램 런타임 환경은 응용 프로그램과 함께 배포된 파일에 보관되는 구성 정보를 포함합니다. 그런데 배포 후 이런 파일을 편집해 응용 프로그램 동작을 변경할 수 없는 경우가 있습니다. 그런 경우 구성을 변경하려면 응용 프로그램을 재배포해야 하는데, 응용 프로그램의 재배포는 용납할 수 없는 가동 중지 시간 및 다른 관리 오버헤드를 초래하는 경우가 많습니다.

로컬 구성 파일 역시 구성을 단일 응용 프로그램으로 제한하지만, 때로 여러 응용 프로그램에서 구성 설정을 공유하는 데 로컬 구성 파일이 유용할 수 있습니다. 그와 같은 사례로는 데이터베이스 연결 문자열, UI 테마 정보, 큐의 URL 및 응용 프로그램의 관련 집합이 사용하는 저장소를 꼽을 수 있습니다.

문제는 특히 클라우드 호스팅 시나리오와 관련해 응용 프로그램에서 실행 중인 여러 인스턴스의 로컬 구성 변경 내용을 관리하기가 어렵다는 것입니다. 그 결과 업데이트를 배포하는 동안 여러 구성 설정을 사용하는 인스턴스가 발생할 수 있습니다.

또한 응용 프로그램과 구성 요소를 업데이트하려면 구성 스키마를 변경해야 합니다. 대부분의 구성 시스템은 구성 정보의 다양한 버전을 지원하지 않습니다.

## <a name="solution"></a>해결 방법

구성 정보를 외부 저장소에 저장하고 구성 설정을 빠르고 효율적으로 읽고 업데이트하는 데 사용할 수 있는 인터페이스를 제공합니다. 외부 저장소의 유형은 응용 프로그램의 호스팅과 런타임 환경에 따라 달라집니다. 클라우드 호스팅 시나리오에서는 보통 클라우드 기반 저장소 서비스이지만 호스티드 데이터베이스 또는 다른 시스템이 될 수도 있습니다.

구성 정보를 위해 선택하는 백업 저장소는 일관되고 사용하기 쉬운 액세스를 제공하는 인터페이스를 포함해야 합니다. 인터페이스는 올바르게 형식화된 정보를 구조화된 형식으로 표시해야 합니다. 구현도 구성 데이터를 보호하기 위해 사용자 액세스에 권한을 부여해야 하고 구성의 여러 버전(각각의 구성별로 여러 릴리스 버전을 포함하는 개발, 준비, 프로덕션 등)을 저장할 수 있을 정도로 충분히 유연해야 합니다.

> 많은 기본 제공 구성 시스템은 응용 프로그램이 시작될 때 데이터를 읽고 데이터를 메모리에 캐시해 빠른 액세스를 제공하고 응용 프로그램 성능에 미치는 영향을 최소화합니다. 사용하는 백업 저장소의 유형과 백업 저장소의 대기 시간에 따라 외부 구성 저장소 내에 캐싱 메커니즘을 구현하는 것이 유용할 수 있습니다. 자세한 내용은 [캐싱 지침](https://msdn.microsoft.com/library/dn589802.aspx)을 참조하세요. 다음 그림은 선택적 로컬 캐시가 있는 외부 구성 저장소 패턴의 개요를 보여 줍니다.

![선택적 로컬 캐시가 있는 외부 구성 저장소 패턴의 개요](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

수락할 수 있는 성능, 고가용성, 견고성을 제공하고 응용 프로그램 유지와 관리 프로세스의 일부로 백업할 수 있는 백업 저장소를 선택합니다. 클라우드 호스티드 응용 프로그램에서 클라우드 저장소 메커니즘의 사용은 이런 요구 사항을 충족하는 일반적인 선택입니다.

백업 저장소의 스키마를 보관할 수 있는 정보 유형에 유연성을 제공할 수 있는 방식으로 설계합니다. 형식화된 데이터, 설정 모음, 설정의 여러 버전 및 사용 중인 응용 프로그램에 필요한 다른 모든 기능과 같은 구성 요구 사항을 모두 제공하는지 확인합니다. 요구 사항이 변경될 때 추가 설정을 지원하도록 스키마는 확장하기 쉬워야 합니다.

백업 저장소의 물리적 기능, 이런 기능을 구성 정보가 저장되는 방식과 연관시키는 방법 및 성능에 미치는 영향을 고려합니다. 예를 들어 구성 정보를 포함하는 XML 문서를 저장하려면 문서를 구문 분석하여 개별 설정을 읽는 구성 인터페이스 또는 응용 프로그램이 필요합니다. 그러나 이런 사례에서 설정의 캐싱은 느려지는 읽기 성능을 보완하는 데 도움을 줄 수 있지만 설정의 업데이트를 더 복잡하게 만든다는 단점이 있습니다.

구성 인터페이스가 구성 설정의 범위와 상속을 제어하도록 허용하는 방법을 고려합니다. 예를 들면 조직, 응용 프로그램 및 컴퓨터 수준에서 구성 설정의 범위를 지정하는 요구 사항이 여기에 해당할 수 있습니다. 한편 다양한 범위의 액세스에 대한 제어의 위임을 지원하고 개별 응용 프로그램에서 설정의 재정의를 방지 또는 허용할 필요가 있습니다.

구성 인터페이스가 구성 데이터를 형식화된 값, 모음, 키/값 쌍 또는 속성 모음과 같은 필요한 형식으로 표시할 수 있는지 확인합니다.

설정이 오류를 포함하거나 백업 저장소에 설정이 없을 때 구성 저장소 인터페이스의 동작 방법을 고려합니다. 기본 설정으로 되돌리고 오류를 로그하는 것이 적절할 수 있습니다. 또한 구성 설정 키 또는 이름의 대/소문자 구분, 이진 데이터의 저장과 처리 및 null 값 또는 빈 값을 처리하는 방식과 같은 측면도 고려합니다.

적절한 사용자와 응용 프로그램에만 액세스를 허용하도록 구성 데이터를 보호하는 방법을 고려합니다. 이런 방법에는 구성 저장소 인터페이스의 기능이 해당할 수 있지만, 적절한 권한 없이 백업 저장소의 데이터에 직접 액세스할 수 없게 하는 방식도 필요할 수 있습니다. 구성 데이터를 읽고 쓰는 데 필요한 권한을 엄격하게 분리합니다. 또한 구성 설정의 일부 또는 전부에 대한 암호화가 필요한지 여부 및 암호화를 구성 저장소 인터페이스에 구현하는 방법도 고려합니다.

런타임 중 응용 프로그램 동작을 변경하는 중앙 집중식으로 저장하는 구성은 굉장히 중요하며 응용 프로그램 코드의 배포와 동일한 방식을 사용해 배포, 업데이트, 관리해야 합니다. 예를 들어 둘 이상의 응용 프로그램에 영향을 줄 수 있는 변경은 완벽하게 테스트하고 준비한 배포 접근 방식을 사용하여 수행함으로써, 변경이 이런 구성을 사용하는 모든 응용 프로그램에 적절하다는 것을 보장해야 합니다. 관리자가 하나의 응용 프로그램을 업데이트하기 위해 설정을 편집하는 경우, 동일한 설정을 사용하는 다른 응용 프로그램에 악영향을 미칠 수 있습니다.

응용 프로그램이 구성 정보를 캐시하는 경우, 구성 정보가 변경되면 응용 프로그램도 변경해야 합니다. 캐시된 구성 데이터에 만료 정책을 구현하여 캐시된 정보를 주기적으로 자동 새로 고침하고 모든 변경 내용을 선택하여 반영하도록 조치할 수 있습니다.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

이 패턴은 다음에 유용합니다.

- 구성 설정이 여러 응용 프로그램과 응용 프로그램 인스턴스에 공유되는 경우 또는 여러 응용 프로그램과 응용 프로그램 인스턴스에 표준 구성을 사용해야 하는 경우

- 이미지 저장 또는 복잡한 데이터 유형과 같은 필요한 구성 설정을 모두 지원하지는 않는 표준 구성 시스템

- 응용 프로그램의 일부 설정에 대한 보조 저장소로, 응용 프로그램이 중앙 집중식으로 저장하는 설정의 일부 또는 전부를 재정의할 수 있는 경우

- 여러 응용 프로그램의 관리를 단순화하는 방식으로 구성 저장소에 대한 액세스의 일부 또는 모든 유형을 로그하여 구성 설정의 사용을 선택적으로 모니터링하는 경우

## <a name="example"></a>예

Microsoft Azure 호스티드 응용 프로그램에서 구성 정보를 외부적으로 저장하기 위한 대표적인 선택은 Azure Storage를 사용하는 것입니다. Azure Storage는 복원력이 있고, 고성능을 제공하며, 자동 장애 조치(Failover)로 3번 복제되어 고가용성을 제공합니다. Azure Table Storage는 값에 유연한 스키마를 사용할 수 있는 키/값 저장소를 제공합니다. Azure Blob Storage는 데이터의 유형을 개별적으로 명명된 blob에 보관할 수 있는 계층적 컨테이너 기반 저장소를 제공합니다.

다음 예제는 구성 저장소를 구성 정보를 저장하고 표시하는 Blob 저장소로 구현하는 방법을 보여 줍니다. `BlobSettingsStore` 클래스는 구성 정보를 보관하는 Blob 저장소를 추상화하고, 다음 코드에 제시되는 `ISettingsStore` 인터페이스를 구현합니다.

> 이 코드는 _ExternalConfigurationStore_ 솔루션의 _ExternalConfigurationStore.Cloud_ 프로젝트에 포함되어 있고 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store)에서 다운로드할 수 있습니다.

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

이 인터페이스는 구성 저장소에 보관된 구성 설정을 검색하고 업데이트하는 메서드를 정의하며 어떤 구성 설정이 최근에 수정되었는지를 검색하는 데 사용할 수 있는 버전 번호를 포함합니다. `BlobSettingsStore` 클래스는 blob의 `ETag` 속성을 사용해 버전 관리를 구현합니다. `ETag` 속성은 blob에 쓸 때마다 자동으로 업데이트됩니다.

> 보통 이런 간단한 솔루션은 모든 구성 설정을 형식화된 값이 아닌 문자열 값으로 표시합니다.

`ExternalConfigurationManager` 클래스는 `BlobSettingsStore` 개체를 감싸는 래퍼를 제공합니다. 응용 프로그램은 이 클래스를 사용해 구성 정보를 저장하고 검색할 수 있습니다. 이 클래스는 Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) 라이브러리를 사용하여 `IObservable` 인터페이스의 구현을 통해 구성에 이루어진 모든 변경 내용을 표시합니다. `SetAppSetting` 메서드를 호출해 설정을 수정하면 `Changed` 이벤트가 발생하고 이 이벤트의 모든 구독자에게 알려집니다.

모든 설정은 빠른 액세스를 위해 `ExternalConfigurationManager` 클래스 내의 `Dictionary` 개체에 캐시되기도 합니다. 구성 설정을 검색하는 데 사용되는 `GetSetting` 메서드는 캐시에서 데이터를 읽습니다. 설정이 캐시에 없으면 그 대신 `BlobSettingsStore` 개체에서 가져옵니다.

`GetSettings` 메서드는 `CheckForConfigurationChanges` 메서드를 호출해 blob 저장소의 구성 정보가 변경되었는지 여부를 검색합니다. 이런 검색은 버전 번호를 검사하고 검사한 버전 번호와 `ExternalConfigurationManager` 개체에 보관된 현재 버전 번호의 비교를 통해 이루어집니다. 하나 이상의 변경 내용이 발생하면 `Changed` 이벤트가 발생하고 `Dictionary` 개체에 캐시된 구성 설정이 새로 고쳐집니다. 이런 과정은 [캐시 배제 패턴](cache-aside.md)의 적용에 해당합니다.

다음 코드 샘플은 `Changed` 이벤트, `GetSettings` 메서드 및 `CheckForConfigurationChanges` 메서드의 구현 방법을 보여줍니다.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache. 
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> `ExternalConfigurationManager` 클래스도 `Environment`로 명명된 속성을 제공합니다. 이 속성은 준비 및 프로덕션과 같은 다양한 환경에서 실행하는 응용 프로그램을 위한 다양한 구성을 지원합니다.

`ExternalConfigurationManager` 개체도 변경 내용을 확인하기 위해 `BlobSettingsStore` 개체를 주기적으로 쿼리할 수 있습니다. 다음 코드에서 앞서 설명한 것처럼 `StartMonitor` 메서드는 주기적으로 `CheckForConfigurationChanges`를 호출하여 변경 내용을 검색하고 `Changed` 이벤트를 발생시킵니다.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

`ExternalConfigurationManager` 클래스는 `ExternalConfiguration` 클래스를 통해 단일 인스턴스로 인스턴스화됩니다.

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

다음 코드는 _ExternalConfigurationStore.Cloud_ 프로젝트의 `WorkerRole` 클래스에서 가져온 것입니다. 여기서는 응용 프로그램이 `ExternalConfiguration` 클래스를 사용해 설정을 읽는 방법을 보여 줍니다.

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

역시 `WorkerRole` 클래스에서 가져온 다음 코드도 응용 프로그램이 구성 이벤트를 구독하는 방법을 보여 줍니다.

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>관련 패턴 및 지침

- 이 패턴을 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store)에서 사용할 수 있음을 보여주는 샘플.
