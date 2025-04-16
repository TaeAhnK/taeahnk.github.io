---
layout: post
title: "SproutPang"
date: 2000-12-31 12:00:00
thumbnail: "assets/thumbnails/SproutPang.png"
preview_file: "_posts/previews/2000-12-31-SproutPang-Preview.md"
main_post: true
---
## Demo

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 0 auto;">
  <iframe 
    style="position: absolute; top: 0; left: 50%; width: 90%; height: 90%; transform: translateX(-50%);" 
    src="https://www.youtube.com/embed/F2qwxJidN-w" 
    frameborder="0" 
    allowfullscreen="true">
  </iframe>
</div>
<details>
    <summary>Web Version을 1920*1080 환경에서 플레이할 수 있습니다. (소리 주의)</summary>
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

<h2> About </h2>

- 2024.07.17~2024.07.26 (2주, 이후 추가 리팩토링 진행)
- 1인 프로젝트
- Unity 2022.3.30f LTS, C#
- 간결하고 단순한 2D Match-3 게임에 "냐옹의 감시를 피해 작물을 훔친다"는<br>요소를 넣어 재미와 스릴을 추가했습니다.
- 멋쟁이사자처럼 유니티 게임스쿨 1기 자체 대회 출품 (53명 중 6등)
- <details><summary>유니티 게임스쿨 홍보 자료로 사용되었습니다.</summary><img src="/assets/img/SproutPang/SproutPang01.png" width="30%" height="auto"></details>
- [Github Link](https://github.com/TaeAhnK/SproutPang)

<br>

## Logics
### Observer Pattern으로 게임 페이즈 관리하기
<div align="center"><img src="/assets/img/SproutPang/SproutPang02.png" width="80%" height="auto"></div>

게임의 상태를 `Playing`, `Caught`, `GameOver`의 세 가지 페이즈로 구분하면 페이즈에 따라 게임 로직, UI, 사운드 등의 관리를 효율적으로 할 수 있다고 생각했습니다. 이를 바탕으로 GameManager가 페이즈를 변경하면, Observer Pattern을 활용해 하위 매니저에 알리고, 각 하위 매니저가 페이즈에 따른 동작을 수행하는 구조를 설계하였습니다. Observer Pattern을 활용한 구조로 하위 매니저를 확장하기에 용이하고, 페이즈 변화를 매번 확인하는 것이 아닌 알림으로 받아 불필요한 연산을 줄였습니다.

<details markdown="1">
<summary>자세히</summary>

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
<details markdown="1">
<summary>자세히</summary>

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
<div align="center"><img src="/assets/img/SproutPang/SproutPang03.png" width="50%" height="auto"></div>

세부 기획을 완성하기 전, 프로토타입을 제작하기 위해 Grid를 먼저 구현하였습니다. Grid의 내용물이 정해지지 않은 상태에서 쉽게 확장하기 위해 제네릭을 사용해 유연한 Grid를 제작했고, 덕분에 기획을 완성하고도 큰 수정 없이 Grid를 적용할 수 있었습니다.

<details markdown="1">
<summary>자세히</summary>

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

<details markdown="1">
<summary>자세히</summary>

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

### Object Pool을 활용한 Vegetable 최적화
Vegetable과 Vegetable이 터질 때 생성되는 파티클은 수많은 생성과 삭제를 반복합니다. 이는 비싼 `Instantiate` 연산과 GC 호출로 이어져 성능에 영향을 줍니다.
`ObjectPool`을 사용해 생성과 삭제가 아닌 대여와 반납 방식으로 성능을 향상시켰습니다.

<details markdown="1">
<summary>자세히</summary>

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

Profiler 확인 결과, 유의미한 성능의 차이는 일어나지는 않았습니다. 많은 오브젝트가 아니었기 때문이라고 생각합니다. 그러나 이러한 방식의 접근에 의의를 두었습니다.

<br>

### 간단한 사운드 최적화
SproutPang은 4개의 효과음과 2개의 BGM을 사용합니다. 유니티에서 제공하는 기본 기능을 사용해 불필요한 3D 효과를 해제하고 압축 해제 관련 설정을 사용해 오디오 메모리 사용량을 낮췄습니다.

<details markdown="1">
<summary>자세히</summary>
<div align="center"><img src="/assets/img/SproutPang/SproutPang04.png" width="60%" height="auto"></div>
효과음은 다음의 설정을 통해 최적화를 진행했습니다.

- Force To Mono : 입체 음향을 사용하는 것이 아니므로 단일 채널 사운드를 사용했습니다.
- Ambisonic : Sound Field 또한 사용하지 않아 해제했습니다.
- Decompress On Load : 메모리를 조금 더 사용하지만 미리 소리를 압축 해제해 놓습니다. 효과음은 용량이 작아 사용했습니다.
- 압축 포맷 : Vorbis

<div align="center"><img src="/assets/img/SproutPang/SproutPang05.png" width="60%" height="auto"></div>
BGM은 다음의 설정을 사용했습니다.
- Force To Mono
- Ambisonic
- Streaming : 메모리에 파일을 로드하지 않고 디스크에서 읽습니다.
- 압축 포맷 : Vorbis

</details>

<div align="center"><img src="/assets/img/SproutPang/SproutPang06.png" width="80%" height="auto"></div>
Profiler 확인 결과 오디오 메모리가 27.9MB에서 1.9MB로 약 93% 줄었습니다.

<br>

## Epilogue
이번 프로젝트를 통해 다양한 디자인 패턴을 실제로 적용하고, 효율적이고 확장성 있는 코드를 작성하는 연습을 할 수 있었습니다. 
공부한 이론들을 실제 프로젝트에 적용하며 원리와 사용법을 익힐 수 있었고, 재사용과 유지보수를 고려한 코드 작성에 집중했습니다.   

한편, 다양한 최적화 기법을 시도했지만, 실제 성능 향상은 기대만큼 크지 않았던 것은 아쉬웠습니다.
이미 가벼운 사양의 게임이기 때문에 체감할 만한 차이를 만들기 어려웠지만, 시도한 내용을 활용해 다음 프로젝트에서도 다양한 방식의 최적화를 도전해 볼 예정입니다.