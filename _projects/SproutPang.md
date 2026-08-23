---
layout: post
title: "SproutPang"
display_area: projects
display_order: 30
thumbnail_type: img
thumbnail_img: "assets/images/projects/SproutPang/SproutPang.png"
thumbnail_vid: ""
tags:
  - Unity
  - Match3
  - 2D
  - 개인프로젝트
---

꼬꼬가 되어 냐옹의 눈을 피해 작물을 훔치는 Match3 게임입니다.
웹에서 플레이할 수 있습니다.

<!-- preview -->

<div class="print-hide">
<h2>Demo</h2>

<div class="youtube-embed">
  <iframe 
    src="https://www.youtube.com/embed/F2qwxJidN-w" 
    frameborder="0" 
    allowfullscreen="true">
  </iframe>
</div>
<details>
    <summary class="print-hide">Web Version을 1920*1080 환경에서 플레이할 수 있습니다. (소리 주의)</summary>
    <div style="position: relative; padding-bottom: 75%; height: 0; overflow: hidden; max-width: 100%;">
        <iframe 
            style="position: absolute; top: 0; left: 0; width: 140%; height: 140%; border: none; transform: scale(0.7); transform-origin: 0 0;" 
            src="https://taeahnk.github.io/SproutPang/" 
            scrolling="no"
            frameborder="0" 
            allowfullscreen="true">
        </iframe>
    </div>
</details>
</div>

<h2> About </h2>

- 2024.07.17~2024.07.26 (2주, 이후 추가 리팩토링 진행)
- 1인 프로젝트
- Unity 2022.3.30f LTS, C#
- 간결하고 단순한 2D Match-3 게임에 "냐옹의 감시를 피해 작물을 훔친다"는<br>요소를 넣어 재미와 스릴을 추가했습니다.
- 멋쟁이사자처럼 유니티 게임스쿨 1기 자체 대회 출품 (53명 중 6등)
- <details><summary>유니티 게임스쿨 홍보 자료로 사용되었습니다.</summary><img src="/assets/images/projects/SproutPang/SproutPang01.png" width="30%" height="auto" class="print-hide"></details>
- [Github Link](https://github.com/TaeAhnK/SproutPang)
- [관련 블로그 포스트](https://code-in-coffee.tistory.com/58)

## Logics
### Observer Pattern으로 게임 페이즈 관리하기

게임의 상태를 `Playing`, `Caught`, `GameOver`의 세 가지 페이즈로 구분하면 페이즈에 따라 게임 로직, UI, 사운드 등의 관리를 효율적으로 할 수 있다고 생각했습니다. 이를 바탕으로 GameManager가 페이즈를 변경하면, Observer Pattern을 활용해 하위 매니저에 알리고, 각 하위 매니저가 페이즈에 따른 동작을 수행하는 구조를 설계하였습니다. Observer Pattern을 활용한 구조로 하위 매니저를 확장하기에 용이하고, 페이즈 변화를 매번 확인하는 것이 아닌 알림으로 받아 불필요한 연산을 줄였습니다.

<div align="center"><img src="/assets/images/projects/SproutPang/SproutPang02.png" width="80%" height="auto"></div>

<details markdown="1">
<summary class="print-hide">자세히</summary>

```c#
public class GameManager : MonoBehaviour
{
    public void UpdateGameState(GameState state)
    {
        gameState = state;        
        OnGameStateChanged?.Invoke(gameState);
    }
}
```
`GameManager`는 특정 상황에서 `GameState`를 변경하고, `OnGameStateChanged`를 구독한 `SubManager`에 알림을 보냅니다.

```c#
public abstract class SubManager<T> : MonoBehaviour where T : MonoBehaviour
{
    protected virtual void Awake()
    {
        ... 
        GameManager.OnGameStateChanged += OnGameStateChanged;
    }
    
    protected void OnGameStateChanged(GameState state)
    {
        switch (state)
        {
            case GameState.Playing:
                OnPlaying();
                break;
            case GameState.Caught:
                OnCaught();
                break;
            case GameState.GameOver:
                OnGameOver();
                break;
            default:
                break;
        }
    }

    protected virtual void OnDestroy()
    {
        GameManager.OnGameStateChanged -= OnGameStateChanged;
    }
    protected virtual void OnPlaying() { }
    protected virtual void OnCaught() { }
    protected virtual void OnGameOver() { }
}
```
다른 하위 Manager들은 `SubManager`를 상속받으며, `SubManager`는 `OnGameStateChanged`의 구독과 구독 해제, 상태에 대한 Interface 역할을 합니다.
</details>
<br>

### Singleton을 활용한 GameManager
GameManager와 SubManager는 Singleton 패턴을 사용하였습니다. 이를 통해 다양한 곳에서 Global하게 접근할 수 있고, 유일해야 하는 매니저가 단 하나 존재함을 보장합니다.
또한, 기능 단위로 SubManager를 구성하여 추후 기능을 확장할 때 용이하게 하였습니다. (예: SoundManager를 통한 효과음의 볼륨 일괄 조정)
<details markdown="1"  class="print-hide">
<summary class="print-hide">자세히</summary>

```c#
public class GameManager : MonoBehaviour
{
    private static GameManager instance;
    public static GameManager Instance
    {
        get
        {
            if (!instance)
            {
                instance = FindAnyObjectByType<GameManager>();
                if (!instance)
                {
                    var go = new GameObject(typeof(GameManager).Name + " Auto-generated");
                    instance = go.AddComponent<GameManager>();
                }
            }
            return instance;
        }
    }
    ...
    private void Awake()
    {
        if (instance != null && instance != this)
        {
            Destroy(gameObject);
        }
        else
        {
            instance = this;
        }
        instance = this;
    }
}
```

SubManager는 제네릭을 사용해 상속이 가능하며, 쉽게 기능을 확장할 수 있습니다.

```c#
public abstract class SubManager<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;
    public static T Instance
    {
        get
        {
            if (!instance)
            {
                instance = FindAnyObjectByType<T>();
                if (!instance)
                {
                    var go = new GameObject(typeof(T).Name + " Auto-generated");
                    instance = go.AddComponent<T>();
                }
            }
            return instance;
        }
    }
}
```

</details>
<br>

### Generic을 활용한 Grid
<div align="center"><img src="/assets/images/projects/SproutPang/SproutPang03.png" width="50%" height="auto"></div>

세부 기획을 완성하기 전, 프로토타입을 제작하기 위해 Grid를 먼저 구현하였습니다. Grid의 내용물이 정해지지 않은 상태에서 쉽게 확장하기 위해 제네릭을 사용해 유연한 Grid를 제작했고, 덕분에 기획을 완성하고도 큰 수정 없이 Grid를 적용할 수 있었습니다.

<details markdown="1" class="print-hide">
<summary class="print-hide">자세히</summary>

```c#
public class Match3Grid<T> where T : MonoBehaviour
{
    public int width;
    public int height;
    public float cellSize;
    public Vector3 pivot;
    public T[,] gridArray;

    public Match3Grid(int width, int height, float cellSize, Vector3 pivot) { ... }
    public Vector3 GridToWorld(int x, int y, GridPoint gridPoint) { ... }
    public Vector2Int? WorldToGrid(Vector3 worldPosition) { ... }
    public void DrawDebugLines() { ... }
    public bool IsAdjacent(Vector2Int posA, Vector2Int posB) { ... }
    public bool IsValidPos(int x, int y) { ... }
    public bool IsValidPos(Vector2Int pos) { ... }
    public void Swap(Vector2Int targetA, Vector2Int targetB) { ... }
}
```

</details>
<br>

### DFS를 통한 Grid 탐색
Match3의 핵심 메커니즘인 게임 종료 조건 검사(더 이상 수확할 작물이 없음)와 작물 수확 조건 검사(3개 이상의 작물이 연결되어 있는지)는 DFS를 통한 Grid 탐색으로 구현했습니다. 최적의 해를 찾는 것이 아닌, 한 가지의 방법이라도 존재하는 지를 찾는 문제이므로 BFS보다는 DFS를 사용하였고, 이를 통해 BFS보다 적은 평균 탐색 횟수로 Grid에서 적절한 탐색을 수행합니다.

<details markdown="1" class="print-hide">
<summary class="print-hide">자세히</summary>

```c#
public class Match3 : MonoBehaviour
{
    ...
    // DFS
    private bool[,] visited;
    private Stack<Vector2Int> DFSStack = new Stack<Vector2Int>(10);
    private List<Vector2Int> PopList = new List<Vector2Int>(10);
    private Vector2Int[] adjVector = {Vector2Int.down, Vector2Int.right, Vector2Int.left, Vector2Int.up};

    private void HarvestTarget(Vector2Int targetA)
    {
        ResetDFS();
        VegetableType type = grid.gridArray[targetA.x, targetA.y].type;
        DFSStack.Push(targetA);
        while (DFSStack.Count > 0)
        {
            Vector2Int current = DFSStack.Pop();
            if (!visited[current.x, current.y])
            {
                visited[current.x, current.y] = true;
                PopList.Add(current);
                foreach (Vector2Int dir in adjVector)
                {
                    Vector2Int temp = current + dir;
                    if (grid.IsValidPos(temp)
                        && grid.gridArray[temp.x, temp.y] != null
                        && !visited[temp.x, temp.y]
                        && grid.gridArray[temp.x, temp.y].type == type
                        && grid.gridArray[temp.x, temp.y].state == VegetableState.Riped)
                    {
                        DFSStack.Push(temp);                        
                    }
                }
            }
        }

        // Not enough vegetables
        if (PopList.Count < Match3Config.MinPopNum)
        {
            return;
        }
        else
        {
            GameManager.Instance.AddScore(PopList.Count * PopList.Count * 10);
            foreach (Vector2Int popItem in PopList)
            {
                DestroyElement(popItem.x, popItem.y);
            }
            SoundManager.Instance.PlaySound(SoundType.harvest);
        }
    }

    private bool NoMatch3Check()
    {
        // Check Every Vegetable is riped
        for (int i = 0; i < grid.width; i++)
        {
            for (int j = 0; j < grid.height; j++)
            {
                Vector2Int temp = new Vector2Int(i, j);
                if (grid.gridArray[temp.x, temp.y] is null
                    || grid.gridArray[temp.x, temp.y].state != VegetableState.Riped)
                {
                    return false;
                }
            }
        }

        for (int i = 0; i < grid.width; i++)
        {
            for (int j = 0; j < grid.height; j++)
            {
                // DFS for Possible Match3
                ResetDFS();
                VegetableType type = grid.gridArray[i, j].type;
                DFSStack.Push(new Vector2Int(i, j));
                while (DFSStack.Count > 0)
                {
                    Vector2Int current = DFSStack.Pop();
                    if (!visited[current.x, current.y])
                    {
                        visited[current.x, current.y] = true;
                        PopList.Add(current);
                        foreach (Vector2Int dir in adjVector)
                        {
                            Vector2Int temp = current + dir;

                            if (grid.IsValidPos(temp)
                                && grid.gridArray[temp.x, temp.y] is not null
                                && !visited[temp.x, temp.y]
                                && grid.gridArray[temp.x, temp.y].type == type
                                && grid.gridArray[temp.x, temp.y].state == VegetableState.Riped)
                            {
                                DFSStack.Push(temp);
                            }
                        }
                    }
                    if (PopList.Count >= Match3Config.MinPopNum)
                    {
                        return false;
                    }
                }
            }
        }
        return true;
    }
}
```

</details>
<br>

## Optimization
픽셀아트와 단순한 로직을 사용해 성능은 차고 넘치지만, 그럼에도 최적화가 가능하다면 시도해 봐야 한다고 생각해 추가 최적화 작업을 시도했습니다.

<div class="print-hide">
<a href="https://code-in-coffee.tistory.com/58">관련 블로그 포스트 보기</a>
</div>

### Object Pool을 활용한 Vegetable 최적화
Vegetable과 Vegetable이 터질 때 생성되는 파티클은 수많은 생성과 삭제를 반복합니다. 이는 비싼 `Instantiate` 연산과 GC 호출로 이어져 성능에 영향을 줍니다.
`ObjectPool`을 사용해 생성과 삭제가 아닌 대여와 반납 방식으로 성능을 향상시켰습니다.

<details markdown="1" class="print-hide">
<summary class="print-hide">자세히</summary>

```c#
public class ObjectPool
{
    public GameObject prefab;
    private Queue<GameObject> pool = new Queue<GameObject>();

    public ObjectPool(GameObject prefab, int count)
    {
        this.prefab = prefab;
        for (int i = 0; i < count; i++)
        {
            pool.Enqueue(CreateNewObject());
        }
    }

    private GameObject CreateNewObject()
    {
        if (!prefab)
        {
            return null; // Error
        }
        var obj = Object.Instantiate(prefab);
        obj.SetActive(false);
        return obj;
    }

    public GameObject GetObject()
    {
        if (pool.Count > 0)
        {
            var obj = pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        else
        {
            var obj = CreateNewObject();
            obj.SetActive(true);
            return obj;
        }
    }

    public void ReturnObject(GameObject obj)
    {
        obj.SetActive(false);
        pool.Enqueue(obj);
    }

}
```
`Object Pool`을 생성할 때 미리 몇 개의 GameObject를 만들어 큐에 저장해두고, 요청이 있으면 저장된 GameObject를 반환합니다.
더 이상 반환할 GameObject가 없다면 추가로 생성합니다.

```c#
public class Match3 : MonoBehaviour
{
    ...
    private Dictionary<VegetableType, ObjectPool> VegetableObjPool 
                                                        = new Dictionary<VegetableType, ObjectPool>();
    ...
}


public class VegetableParticleManager : MonoBehaviour
{
    ...    
    private Dictionary<VegetableType, ObjectPool> ParticleObjectPools 
                                                        = new Dictionary<VegetableType, ObjectPool>();
    ...
}
```
Match3와 VegetableParticleManager는 `<VegetableType, ObjectPool>`의 쌍을 `Dictionary`로 저장하여 키를 통해 필요한 `Object Pool`에 접근합니다.

</details>

<br>
Profiler 확인 결과, 모바일 웹 60 프레임 환경에서 Object Pool 용량을 넘어가는 Particle이 생성될 때 60 프레임을 보장하지 못하는 경우가 발생했고, 12의 Pool Size로 이를 예방할 수 있었습니다.


### 오디오 로드 타이밍 관리로 프레임 유지하기
<div align="center"><img src="/assets/images/projects/SproutPang/SproutPang07.png" width="80%" height="auto"></div>
GameOver UI가 등장하고 냐옹이 우는 시점에서 프레임 타임이 20 ms를 넘어갔습니다.
이 시점에서는 UI 팝업, 냐옹 사운드 재생, GameOver BGM 재생이 겹쳐 목표한 60 프레임이 달성되지 않았습니다.

이를 해결하기 위해 몇 가지 내용을 조정했습니다.

1. 냐옹의 울음소리를 Preload로 변경해 해당 시점의 로드로 인한 부하를 게임 초기 로딩으로 옮겼습니다.
2. 게임 오버 직전, 나옹이 걸어오는 애니메이션이 재생되는데, 이때 GameOver BGM의 로드를 시작하도록 변경했습니다.


<div align="center"><img src="/assets/images/projects/SproutPang/SproutPang08.png" width="80%" height="auto"></div>

이로 인해 냐옹이 걷기 시작하는 Caught에서 프레임 타임이 13 ms, GameOver에서 14 ms로 분배되어 안정적인 60 프레임이 되었습니다.

<div align="center"><img src="/assets/images/projects/SproutPang/SproutPang09.png" width="80%" height="auto"></div>
이 밖에도 폰트 에셋 경량화, Sprite Atlas, 셰이더 최적화, 오디오, 텍스처 압축 방식 변경, BGM 로드 시점 변경, 프로젝트 설정 등의 최적화를 적용했고,
그 결과 아이폰 13 기준 모바일 웹 환경에서 60 프레임을 보장하며, 45%의 사용 메모리 감소와 20%의 빌드 사이즈 감소를 달성했습니다.


## Epilogue
이번 프로젝트를 통해 다양한 디자인 패턴을 실제로 적용하고, 효율적이고 확장성 있는 코드를 작성하는 연습을 할 수 있었습니다. 
공부한 이론들을 실제 프로젝트에 적용하며 원리와 사용법을 익힐 수 있었고, 재사용과 유지보수를 고려한 코드 작성에 집중했습니다.   

별도로 진행한 프로젝트 최적화도 재밌는 결과를 보였습니다. 프로파일러로 나아진 결과를 보는 재미가 있었습니다. PC 환경에서는 너무 가벼워 모바일 웹 환경을 기준으로 진행했습니다.
PC 환경에서는 체감되지 않던 성능 향상이 크게 작용했고, 다양한 기법을 통해 안정적인 프레임 뿐만 아니라 메모리와 빌드 사이즈도 경량화 시킬 수 있었습니다.
시도하지 못한 내용도 많은 것은 조금 아쉬웠습니다. 3D 게임에서는 스레드 병목이나 렌더링 최적화 등도 시도 해볼 수 있을 것 같습니다.
