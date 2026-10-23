# Unity Utilities

Unity `6000.3` 이상에서 사용하는 UPM 패키지예요. 이 폴더의 **내용 전체(숨김 파일과 `.meta` 포함)**를 새 `unity-utilities` 저장소의 루트에 복사하면 돼요. `package.json`이 저장소 루트에 있어야 해요.

## 포함된 기능

| 타입 | 네임스페이스 | 기능 |
| --- | --- | --- |
| `MathUtil` | `Utility` | Vector2 방향의 각도를 도 단위로 반환해요. |
| `SingletonMonoBehaviour<T>` | 전역 | Awake에서 인스턴스를 등록하고 플레이 중 씬 전환에도 유지해요. |
| `ErrorHandler` | `Utility` | UniTask를 반환하는 작업을 재시도해요. |

기존 호출부와의 호환성을 위해 원본 스크립트의 내용, 네임스페이스, `.meta` GUID를 유지해요.

## 구조

```text
package.json
Runtime/
  Song.Unity.Utilities.asmdef
  MathUtil.cs
  SingletonMonoBehaviour.cs
  ErrorHandler.cs
```

패키지 ID는 `com.song.unity-utilities`, 어셈블리 이름은 `Song.Unity.Utilities`, 초기 버전은 `0.1.0`이에요.

## 설치

Unity의 Package Manager에서 **Install package from Git URL**로 다음 순서대로 설치해요. `YOUR_ACCOUNT`는 생성한 저장소의 소유자로 바꾸고, 저장소에 `v0.1.0` 태그를 먼저 생성해요.

1. UniTask `2.5.10`

   ```text
   https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask#2.5.10
   ```

2. Unity Utilities `0.1.0`

   ```text
   https://github.com/YOUR_ACCOUNT/unity-utilities.git#v0.1.0
   ```

`package.json`에는 UniTask의 패키지 ID와 버전을 선언해요. UPM은 패키지 사이의 Git URL 의존성을 지원하지 않으므로, UniTask를 Git으로 사용하는 프로젝트에서는 위 URL로 먼저 직접 설치해야 해요.

로컬 개발 중에는 UniTask를 설치한 뒤 **Install package from disk**에서 이 폴더의 `package.json`을 선택하면 돼요.

소비 코드가 자체 `.asmdef`를 사용하면 `Song.Unity.Utilities` 참조를 추가해요. 소비 코드에서 UniTask 타입도 직접 사용하면 `UniTask` 어셈블리도 참조해요. 기본 `Assembly-CSharp`에서는 자동 참조해요.

## 사용 예시

```csharp
using UnityEngine;
using Utility;

float angle = MathUtil.GetAngle(Vector2.up); // 90도
```

```csharp
public sealed class AppManager : SingletonMonoBehaviour<AppManager> {
}
```

싱글턴 컴포넌트를 씬의 루트 GameObject에 추가해야 해요. `Instance`는 Awake에서 등록되며 인스턴스를 자동 생성하지 않아요. Awake를 재정의하면 `base.Awake()`를 호출해요.

```csharp
using Cysharp.Threading.Tasks;
using Utility;

bool succeeded = await ErrorHandler.DoWithReTryAsync(
    () => UniTask.FromResult(true),
    "Initialize",
    3);
```

현재 `retryCount`는 최초 실행을 포함한 총 시도 횟수예요. false를 반환하면 500ms 대기하고, 예외가 발생하면 로그를 남긴 뒤 즉시 다음 시도로 넘어가요. 취소 토큰이나 재시도 정책은 이번 복사에서 변경하지 않았어요.

## 기존 프로젝트에 적용할 때

원본 `.cs`와 같은 GUID의 `.meta`가 이 패키지에 들어 있어요. 소비 프로젝트의 Assets에 원본을 둔 채 패키지까지 설치하면 타입과 GUID가 중복되므로, 실제 패키지 전환 시에는 원본과 패키지 중 하나만 임포트되도록 해야 해요. 현재 CrawlStars의 원본 파일은 그대로 유지했어요.

## 릴리스

`package.json` 버전과 CHANGELOG를 갱신하고, 해당 커밋에 같은 버전의 태그(`v0.1.0` 등)를 생성해요. 소비 프로젝트에서 새 태그 URL로 설치한 뒤 `Packages/manifest.json`과 `Packages/packages-lock.json`을 함께 커밋해요.
