---
layout: post
title: "CoDefence"
date: 2000-01-01 12:00:00
thumbnail: "assets/thumbnails/CoDefence.png"
preview_file: "_posts/previews/2000-01-01-CoDefece.md"
main_post: true
order: 2
---

## Demo

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 0 auto;">
  <iframe 
    style="position: absolute; top: 0; left: 50%; width: 90%; height: 90%; transform: translateX(-50%);" 
    src="https://www.youtube.com/embed/d_d7eyaYCBs" 
    frameborder="0" 
    allowfullscreen="true">
  </iframe>
</div>

<details>
    <summary>Web Version을 1920*1080 환경에서 플레이할 수 있습니다. (소리 주의, 전체 화면 권장)</summary>
    <div style="position: relative; width: 100%; max-width: 1920px; margin: auto; aspect-ratio: 16/10;">
    <iframe 
        src="https://taeahnk.github.io/CoDefence/"
        allowfullscreen
        scrolling = no
        style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;">
    </iframe>
    </div>
    run viruskiller3를 입력하면 시작합니다.
</details>


<h2>About</h2>

- 2024.11.14 ~ 2024.11.15 (1일, 이후 추가 리팩토링 진행)
- 1인 프로젝트
- Unity 2022.3.30f LTS, C#
- 한컴 타자 연습의 '산성비'처럼 하늘에서 떨어지는 코드를 입력해 바이러스의 침투를 막는 디펜스 게임
- 의도치 않게 24시간 타임어택으로 진행된 프로젝트
- [Github Link](https://github.com/TaeAhnK/CoDefence)


## Refactoring

### 클래스 재정비와 로직 개선
빠르게 제작한 프로젝트다 보니 기능 구현에 최대한 집중했습니다. 많은 기능이 GameManager에 강하게 결합되어 있고, 서로의 데이터에 직접 접근하며, 하나의 클래스가 여러 책임을 맡는 구조였습니다.

<div align="center"><img src="/assets/img/CoDefence/CoDefence01.png" width="100%" height="auto"></div>


리펙토링을 통해 이벤트를 바탕으로 한 구조로 클래스 간의 결합을 낮추고, 각 클래스의 역할을 더욱 명확하게 변경하였습니다. 오브젝트들은 이벤트 버스 GameEvent를 구독하고 트리거하여, 서로의 구현과 변경에 영향을 받지 않도록 했습니다. Word의 관리와 소환을 모두 담당하던 WordSpawner의 역할을 WordPool과 WordSpawner로 분리해 각자의 책임을 명확히 했습니다. 이를 통해 새로운 기능 추가와 유지보수가 쉬워졌습니다.

<div align="center"><img src="/assets/img/CoDefence/CoDefence02.png" width="100%" height="auto"></div>

<details markdown="1">
<summary>자세히</summary>

**기존 코드 (일부)**
```c#
public class WordSpawner : MonoBehaviour
{
    ...
    private void Update()
    {
        if (GameManager.Instance.IsGameOver) return;

        timer += Time.deltaTime;

        if (timer >= duration)
        {
            timer = 0;
            SpawnWord();
        }
    }
    ...
}
```

**변경 코드 (일부)**
```c#
public static class GameEvent
{
    ...
    public static event Action OnGameOver;
    public static void RaiseOnGameOver()
    {
        OnGameOver?.Invoke();
    }
    ...
}


public class WordSpawner : MonoBehaviour
{
    ...
    private void OnEnable()
    {
        GameEvent.OnGameOver += OnGameOver;
    }

    private void OnDisable()
    {
        GameEvent.OnGameOver -= OnGameOver;
    }

    private void OnGameOver()
    {
        enabled = false;
    }
    ...
}

```
`GameManager`를 계속 확인하며 GameOver를 탐지하는 방식 대신, `GameEvent`의 `OnGameOver`를 구독해 `GameManager`로 부터 알림을 받는 형식으로 변경해 두 클래스의 결합도를 낮췄습니다. 이에 따라 `GameManager`을 수정하여도 `WordSpawner`는 수정할 필요 없이 잘 동작할 것입니다.

**수정된 SubManager**
```c#
public static class SoundEvent
{
    public static event Action<SoundType> OnSoundPlayEvent;
    public static event Action<SoundType> OnSoundStopEvent;
    
    public static void PlaySound(SoundType soundType)
    {
        OnSoundPlayEvent?.Invoke(soundType);
    }

    public static void StopSound(SoundType soundType)
    {
        OnSoundStopEvent?.Invoke(soundType);
    }
}

public class SoundManager : Singleton<SoundManager>
{
    ...
    private void OnEnable()
    {
        SoundEvent.OnSoundPlayEvent += PlaySound;
        SoundEvent.OnSoundStopEvent += StopSound;
    }

    private void OnDisable()
    {
        SoundEvent.OnSoundPlayEvent -= PlaySound;
        SoundEvent.OnSoundStopEvent -= StopSound;
    }
    ...
}
```
`SoundManager`나 `UIManager`도 이벤트 버스를 통해 소리를 재생하거나, UI를 업데이트하는 구조로 변경하였습니다.

</details>

<br>

### 상속할 수 있는 Singleton
모든 `Manager`와 `SubManager`가 가지는 `Singleton` 속성을 상위 클래스로 만들어 코드 중복을 줄이고 새 매니저를 추가할 때 간편하게 구현하도록 변경하였습니다.

<details markdown="1">
<summary>자세히</summary>

```c#
public abstract class Singleton<T> : MonoBehaviour where T : MonoBehaviour
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

    protected virtual void Awake()
    {
        if (instance != null && instance != this)
        {
            Destroy(gameObject);
            return;
        }
        else
        {
            instance = this as T;
        }
    }
}

public class GameManager : Singleton<GameManager> { ... }
public class BGMManager : Singleton<BGMManager> { ... }
public class SoundManager : Singleton<SoundManager> { ... }
public class MainMenuManager : Singleton<MainMenuManager> { ... }
public class GameStateUIManager : Singleton<GameStateUIManager> { ... }
```

</details>

<br>

### 간편한 패치를 위한 Scriptable Object
<div align="left"><img src="/assets/img/CoDefence/CoDefence03.png" width="50%" height="auto"></div>

밸런스 조절을 위한 변수와 단어 목록을 Scriptable Object로 저장하도록 변경하여 Inspector 창에서 간편히 변수를 변경하고 적용할 수 있도록 수정하였습니다.
<br>


## Optimization
### String 최적화
Unity의 `String`은 한 번 생성되면 수정이 불가능한 상태로 메모리에 저장됩니다. 따라서, 문자열을 수정할 때마다 새로운 문자열이 생성되고, 이는 메모리 증가와 GC 증가로 이어집니다. 이를 보완하기 위해 가변 버퍼를 사용하는 `StringBuilder`와 문자열을 하나만 저장하고 참조를 사용하는 `Interned String`을 사용해 메모리 낭비를 줄였습니다.

<details markdown="1">
<summary>자세히</summary>

**기존 String 수정 방식**
```c#
public class TypingEffect : MonoBehaviour
{
    ...
    private IEnumerator TypingEnumerator(string text)
    {
        this.text.text = string.Empty;

        for (int i = 0; i < text.Length; i++)
        {
            this.text.text += text[i];
            yield return new WaitForSeconds(typingSpeed);
        }
    }
    ...
}
```

**`StringBuilder` 사용**
```c#
public class TypingEffect : MonoBehaviour
{
    ...
    private IEnumerator TypingEnumerator(string text)
    {
        StringBuilder sb = new StringBuilder();
        this.text.text = string.Empty;

        for (int i = 0; i < text.Length; i++)
        {
            sb.Append(text[i]);
            this.text.text = sb.ToString();
            yield return new WaitForSeconds(typingSpeed);
        }
    }
    ...
}
```
한 글자씩 출력하여 타이핑 효과를 주는 스크립트입니다. += 연산으로 매번 새로운 문자열이 생성되던 문제를 StringBuilder로 해결해, 메모리 사용을 줄였습니다.

**Interned String 사용**
```c#
public class WordPool : MonoBehaviour
{
    ...
    private void LoadWordData()
    {
        ...
        string internKey = string.Intern(data.Key);
        word.Init(internKey, ...);
        ...
    }
    
    public Word GetWord()
    {
        ...
        spawned.Add(string.Intern(word.text), word);
        ...
    }

    public void ReturnWord(Word word)
    {
        ...
        spawned.Remove(string.Intern(word.text));
        ...
    }
}
```
`Word`의 `text`를 `Intern String`으로 저장해 `spawned` 딕셔너리에 추가하고 삭제할 때 `String`은 같은 참조를 사용하게 되고, 이를 통해 메모리를 절약할 수 있었습니다.

</details>



## Troubleshoot
### 정확한 Spawn 범위
`Word`의 소환 위치가 단어의 길이에 따라 화면을 벗어나는 현상이 있어 초기에는 소환 범위를 보수적으로 잡아 임시 조치를 했습니다. 단어가 많아지면 가장자리에는 단어가 소환되지 않는 것이 노골적으로 보여 어색해 보였고, 문제를 파악하기 위해 Unity UI 좌표계에 대해 조사하고, 오브젝트를 다양한 위치에 배치해 보며 원인을 파악했습니다.

<div align="center"><img src="/assets/img/CoDefence/CoDefence04.png" width="70%" height="auto"></div>


가장 큰 문제는 단어의 길이가 길어져도 UI Box를 넘어가서 출력되고, Box의 Width가 늘어나지는 않았습니다. 정확한 Width를 구하지 못해 Box는 범위 안에 있지만, 글자는 범위를 벗어나는 현상이 발생했습니다.

정확한 Width 계산 방법을 고민하던 중, 사용 중인 폰트가 고정폭 폰트(Consolas)임을 깨달았습니다. 이에 따라 Width는 글자 수에 정비례하며, 한 글자당 13.33...의 Width를 가진다는 점을 활용해 정확한 위치 계산을 위한 방법을 도출했습니다.

```c#
public class Word : MonoBehaviour
{
    public void Init(string text, ...)
    {
        ...
        rectT.sizeDelta = new Vector2(text.Length * 13.34f, rectT.sizeDelta.y);
        ...
    }
}

```
Word를 초기화할 때 Width도 변경하도록 수정하고, 좌표 계산의 편의성을 위해 화면과 Word의 Anchor를 좌측으로 고정했습니다.

<div align="center"><img src="/assets/img/CoDefence/CoDefence05.png" width="70%" height="auto"></div>


정확해진 계산으로 정상 위치에 Spawn 됩니다.


## Epilogue

이번 프로젝트는 24시간이라는 짧은 시간 안에 핵심 게임 플레이를 빠르게 구현하는 데 집중했습니다. 한정된 시간 동안 단순하고 직관적인 구조로 핵심 기능을 빠르게 완성할 수 있었습니다. 이 과정에서 완성도보다는 실행과 실험에 집중하며, 아이디어를 실제 플레이 가능한 형태로 빠르게 검증하는 연습을 할 수 있었습니다. <br><br>
프로토타입 완성 이후에는, 급하게 만든 구조를 리팩토링하고 최적화하는 과정을 통해 코드의 유지보수성과 확장성을 높이는 방법도 함께 익혔습니다.
프로토타입으로 짧은 시간 내에 반복적으로 시도하고, 빠르게 결과를 확인하는 경험은 앞으로도 새로운 아이디어를 실험하거나 게임의 재미 요소를 빠르게 테스트할 때 큰 도움이 되리라 생각합니다.

<details markdown="1">
<summary>뒷이야기</summary>

> _부트캠프 두번째 프로젝트로 디펜스 게임 제작이 있었지만, 강사님께서 프로젝트 동안 다른 공부를 하고 싶으면 그것도 괜찮다고 하셔서 따로 진행 중이던 언리얼 프로젝트를 진행했다.<br><br> 몇 달 후 부트캠프 수료 이틀 전에 매니저님께서 디펜스 게임을 포함해 총 3개의 프로젝트를 제출해야 한다고 하셨다. 하루 만에 디펜스 게임 하나를 제작하고, 영상과 결과물을 제출해야 했다. <br><br> 언젠가 코드 타자 연습을 만들어 보겠다는 계획이 있었고, 이를 디펜스 게임과 조합할 방법으로 산성비가 떠올라 빠르게 기획했다. 배경은 마침 보이던 Sci-Fi UI 에셋을 활용해 컴퓨터에 침투하는 바이러스 제거라는 컨셉을 만들었다. 최적화, 객체 지향 등등 생각할 것은 많았지만 신경쓸 겨를도 없었다. 다급함과 함께 밤을 꼬박 새워 다음날 오전 9시에 제출을 완료했다. 힘들긴 했지만 나름 재미있는 경험이었다._
</details>