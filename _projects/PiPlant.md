---
layout: post
title: "PiPlant"
thumbnail: "assets/img/PiPlant/PiPlant.png"
main_post: true
order : 5
published: true
---


#Unity #모바일 #2D #캐주얼 #팀프로젝트<br>
멋쟁이사자처럼 로켓단 인턴십 중 타라게임즈와 협업하여 출시를 목표로 진행한 프로젝트입니다. 파이프를 회전해 퍼즐을 푸는 간단한 모바일 캐주얼 게임입니다.
<!--more-->
## Demo

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 0 auto;">
  <iframe 
    style="position: absolute; top: 0; left: 50%; width: 90%; height: 90%; transform: translateX(-50%);" 
    src="https://youtube.com/embed/9FFyFboAQqI" 
    frameborder="0" 
    allowfullscreen="true">
  </iframe>
</div>


<h2>About</h2>

- 2025.05.20 ~ 2025.06.18 (4주)
- 4인 프로젝트 (인게임 개발 담당)
- Unity 6000.0.47f1, C#
- 멋쟁이사자처럼 로켓단 인턴십 중 타라게임즈와 협업하여 진행한 프로젝트
- 파이프를 연결해 꽃을 피우는 간단한 모바일 캐주얼 게임
<!-- - [Github Link](https://github.com/) -->


## Logics

### 디자이너 편의성을 고려한 Pipe와 Level 팔레트
<br>
<div align="center"><img src="/assets/img/PiPlant/PiPlant02.png" width="70%" height="auto"></div>

디자이너와 기획자가 쉽게 파이프 이미지를 적용해보고 스테이지를 구성할 수 있도록 커스텀 에디터를 제작했습니다.
파이프 종류 추가, 테마 설정, 기믹 추가 등 기획이 심화될수록 에디터의 기능도 추가하여 디자이너가 문제없이 스테이지를 구성할 수 있도록 도왔습니다.

<details markdown="1">
<summary>자세히</summary>

**Level Editor 주요 기능**
- 그리드 사이즈 설정
- 시작, 도착 지점 설정
- 데이터 저장
- 새 레벨 생성
- 타일맵 팔레트처럼 클릭으로 파이프 배치
- 파이프 종류 선택
- 테마 설정
- 스테이지 정보 저장 및 게임 플레이 시 출력

</details>

<div align="center"><img src="/assets/img/PiPlant/PiPlant01.png" width="35%" height="auto"></div>

<br>

### 다양한 기믹을 위한 GimmickPipe
<br>
<div align="center"><img src="/assets/img/PiPlant/PiPlant03.png" width="40%" height="auto"></div>

PiPlant의 핵심 재미 요소는 다양한 기능을 하는 기믹 파이프입니다.
폭탄 파이프, Lock-Key 파이프, 얼음 파이프를 파훼해 퍼즐을 풀어야 합니다.
다양한 파이프를 더 효율적으로 유지보수하고, 새 기믹을 더 쉽게 추가하기 위해 `Interface`를 활용한 구조로 `GimmickPipe`를 설계했습니다.

<details markdown="1">
<summary>자세히</summary>

PipeData에서 파이프의 기믹을 설정합니다. 

```c#
public enum PipeType
{
    Normal,
    CountLocked,
    Key,
    Bomb,
    Flower,
    SequenceFlower,
    BlackBox,
    Frozen,
    FrozenFlower
}

[CreateAssetMenu(fileName = "New Pipe", menuName = "Pipe/Create New Pipe")]
public class PipeData : ScriptableObject
{
    public string identifier; // Pipe Name
    public PipeType pipeType; // Pipe Type

    ...

    // Gimmick Variables
    public bool rotatable = true;
    public bool rotatableAfterGimmick = true;
    
    //// Bomb Pipe
    public float bombTimeLimit = 5f;
    
    //// Key-Lock Pipe
    public int keyId;
    public int unlockCount;
}
```

PipeLoader가 PipeData를 읽어 파이프를 구성할 때, PipeType에 따라 파이프에 GimmickPipe Component를 부착합니다.

```c#
public class PipeLoader : MonoBehaviour
{
  ...
    private void AddGimmickComponent(GameObject obj, PipeData pipeData)
    {
        switch (pipeData.pipeType)
        {
            case PipeType.Normal:
                break;
            case PipeType.CountLocked:
                CountLockGimmickPipe gpclobj = obj.AddComponent<CountLockGimmickPipe>();
                gpclobj.Setup(pipeData);
                break;
            case PipeType.Key:
                KeyGimmickPipe gpkobj = obj.AddComponent<KeyGimmickPipe>();
                gpkobj.Setup(pipeData);
                break;
            ...
            case PipeType.Frozen:
                FrozenGimmickPipe gpfpobj = obj.AddComponent<FrozenGimmickPipe>();
                gpfpobj.Setup(pipeData);
                break;
            default:
                break;
        }
    }
}
```

GimmickPipe는 기믹 구현을 위한 Interface로 구현되어 있습니다.
각 Interface 함수는 연결, 첫 연결, 연결 해제, 게임 시작, 게임 끝 등의 상황에 실행됩니다.

```c#
public interface IGimmickPipe
{
    void Setup(PipeData pipeData);
    void OnConnected();
    void OnFirstConnected();
    void OnDisconnected();
    void SolveGimmick();
    void OnGameStart();
    void OnGameEnd();
}

public abstract class GimmickPipe : MonoBehaviour, IGimmickPipe
{
    protected Pipe pipe;
    protected bool wasConnected = false;
    protected bool rotatableAfterGimmick;
    protected bool isGimmickSolved = false;

    protected virtual void Awake()
    {
        pipe = GetComponent<Pipe>();
    }
    
    public virtual void Setup(PipeData pipeData) { }
    public virtual void OnConnected() { }
    public virtual void OnFirstConnected() { }
    public virtual void OnDisconnected() { }
    public virtual void SolveGimmick() { }
    public virtual void OnGameStart() { }
    public virtual void OnGameEnd() { }
}
```

GimmickPipe를 상속 받아 다양한 기믹을 구현하였습니다.
Interface를 활용해 쉽게 새 기믹을 추가할 수 있었습니다.

```c#
public class BombGimmickPipe : GimmickPipe
{
    private float bombTimeLimit;
    private Timer bombTimer;
    private TextMeshProUGUI bombTimeText;
    private Image bombImage;
    private int lastDisplayedSeconds = -1;

    protected override void Awake() { ... }    
    private void OnEnable() { ... }
    private void OnDisable() { ... }    
    public override void Setup(PipeData pipeData) { ... }
    ...
    public override void OnGameStart()
    {
        bombTimer.StartTimer(bombTimeLimit);
    }

    private void HandleTimeout()
    {
        bombTimeText.enabled = false;
        GameGimmickEventBus.RaiseBombExplode();
    }

    public override void OnFirstConnected()
    {
        if (wasConnected) return;
        wasConnected = true;
        
        if (bombTimer != null)
        {
            bombTimer.StopTimer();
            ...
        }
        ...
    }
    ...
}

```

</details>

<br>


### EventBus를 통한 디커플링
<br>
<div align="center"><img src="/assets/img/PiPlant/PiPlant04.png" width="90%" height="auto"></div>

협업 과정에서 특정 파일에서 충돌이 나는 경우가 종종 있었습니다. 특히 주요 로직을 관리하는 `GameManager`에서 UI 담당이 코드를 추가하며 충돌이 잦았습니다. 이를 의존성이 높은 상황으로 파악하고 의존도를 낮춰 충돌을 줄이기 위해 EventBus를 통해 디커플링을 구현했습니다.
주요 이벤트를 `EventBus`로 구현해 구독하고 알림을 보내는 구조로 수정해 직접 다른 클래스에 접근하지 않고도 상태 변화를 감지해 적절한 행동이 가능하도록 리팩토링했습니다.

<details markdown="1">
<summary>자세히</summary>

```c#
public static class GameStateEventBus // From GameManager to Pipes or Others
{
    public static event Action OnGameStart;
    public static void RaiseGameStart() => OnGameStart?.Invoke();
    
    public static event Action OnGameEnd;
    public static void RaiseGameEnd() => OnGameEnd?.Invoke();
    
    ...
    public static event Action OnStageClearCompleted;
    public static void RaiseStageClearCompleted() => OnStageClearCompleted?.Invoke();
}

public static class GameGimmickEventBus // From Pipe to Pipe or GameManager
{
    public static event Action<int> OnKeyConnected;
    public static void RaiseKeyConnected(int key) => OnKeyConnected?.Invoke(key);
    ...
}

public static class GameItemEventBus // From ItemManager to UI or Others
{
    public static event Action OnCrossPipeItemCompleted;
    public static void RaiseCrossPipeItemCompleted() => OnCrossPipeItemCompleted?.Invoke();
    ...
}
```
</details>

<br>


## Epilogue

이번 프로젝트에서는 회사 고유 라이브러리를 활용하여 모바일 캐주얼 게임을 개발했습니다. 그 중 주요 재미 요소인 인게임 퍼즐 개발을 담당했습니다. <br><br>
4인이서 한달 안에 기획부터 출시까지를 목표로 프로젝트를 진행했지만 출시까지는 진행하지 못한 것이 아쉬웠습니다. 초기 기획(1주)과 프로토타이핑 기간(1주)을 너무 여유롭게 잡은 탓에 개발 기간이 많이 짧아져 아웃 게임 개발이 늦어졌습니다. <br><br>
프로젝트를 진행하며 기획자/디자이너와의 소통에 대해서 많이 고민했습니다. 기획자가 의도한 바를 빠르고 정확하게 이해하고 이를 완성도 있게 구현하는 것에 초점을 맞췄습니다. 디자이너가 다양한 시도를 편하게 해볼 수 있도록 직관적인 에디터 툴을 제작한 것도 값진 경험이라고 생각합니다.
