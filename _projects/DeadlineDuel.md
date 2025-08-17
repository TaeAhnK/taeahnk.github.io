---
layout: post
title: "DeadlineDuel"
thumbnail: "assets/img/DeadlineDuel/DeadlineDuel.png"
main_post: true
published: true
order : 3
---

#Unity #3D #액션 #레이드 #팀프로젝트 #보스<br>
3명이 진행한 팀 프로젝트입니다. 두 플레이어가 서로 다른 보스 몬스터를 잡으며 경쟁하는 경쟁형 액션 PvE 게임입니다. 

<!--more-->
## Demo
### 중간발표 자료

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 0 auto;">
  <iframe 
    style="position: absolute; top: 0; left: 50%; width: 90%; height: 90%; transform: translateX(-50%);" 
    src="https://www.youtube.com/embed/FdjHvWc55Yc?si=fGx_V1d_maLcXUkG&amp;start=42" 
    frameborder="0" 
    allowfullscreen="true">
  </iframe>
</div>

<h2> About </h2>

- 2025.04.01 ~ 2025.05.07 중간 발표
- 팀 프로젝트 (3인)
- 보스 몬스터 담당
- Unity 2022.3.58f LTS, C#, Unity NetCode
- 두 플레이어가 서로를 방해하며 보스 타임 어택을 진행하는 3D 액션 게임
- [Github Link](https://github.com/TaeAhnK/DeadlineDuel)
- <details>
    <summary>발표 자료 (<a href="https://docs.google.com/presentation/d/e/2PACX-1vSaZUwtZscl0152e5SGPeCluwjjVj191HcVxrvD0nOSuG_aQ4wEIiUHCpGiaZyZPZrOyfz5wm2cEmBQ/pub?start=false&loop=false&delayms=60000">Google Slide</a>)</summary>
    <iframe src="https://docs.google.com/presentation/d/e/2PACX-1vSaZUwtZscl0152e5SGPeCluwjjVj191HcVxrvD0nOSuG_aQ4wEIiUHCpGiaZyZPZrOyfz5wm2cEmBQ/pubembed?start=true&loop=false&delayms=60000" frameborder="0" width="800" height="500" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
    </details>


## Logics
### NetCode를 활용한 멀티 플레이
이번 프로젝트에서는 NetCode를 활용해 멀티 플레이를 구현하였습니다. <br>
보스 몬스터의 대부분의 로직은 `Server(Host)`에서 처리하고, 클라이언트는 시각적 효과만을 보여줍니다.

<div align="center"><img src="/assets/img/DeadlineDuel/DeadlineDuel01.png" width="70%" height="auto"></div>

`NavMesh`를 사용해 `Server`의 보스가 이동하면, `Network Transform`을 통해 `Client`에서 이를 동기화합니다. 
애니메이션은 `Network Animator`를 사용해 동기화합니다.<br>
`CurrentState`, `Stat`과 같은 데이터는 `Network Variable`로 설정해 추가적인 과정 없이 값이 변하면 자동으로 동기화되도록 설정했습니다.<br>

<details markdown="1">
<summary>자세히</summary>
`StateMachine`과 `State`, `Damage`등 주요 로직은 `IsServer`인 경우에만 동작합니다.

```c#
public void TakeDamage(float damage)
{
    if (!IsServer) return;
    ...
}
```

```c#
public class BossState : BossBaseState
{
    public BossState(BossStateMachine stateMachine) : base(stateMachine)
    {
        if (!StateMachine.IsServer) return;
        ...
    }

    public override void Enter()
    {
        if (!StateMachine.IsServer) return; // Only on Server
        ...
    }

    public override void Tick(float deltaTime)
    {
        if (!StateMachine.IsServer) return; // Only on Server
        ...
    }
    
    public override void Exit() { }
}
```

</details>

<br>

### FSM을 활용한 상태 전환
보스의 행동 양상을 기획하고, 이에 맞게 Finite State Machine을 구현하여 보스의 행동 전반을 관리했습니다.

<div align="center"><img src="/assets/img/DeadlineDuel/DeadlineDuel02.png" width="65%" height="auto"></div>

보스 몬스터는 Idle 상태에서 공격 대상을 탐색하고, 범위 안에 대상이 있으면 공격, 없으면 이동합니다. 어떤 상태이든 체력이 0이 되면 Death 상태로 돌입합니다.

<details markdown="1">
<summary>자세히</summary>
기본적인 StateMachine과 State를 구현한 뒤, 이를 상속 받는 BossStateMachine을 구현했습니다. 대부분의 로직은 Server에서 처리하도록 설정하고, State를 Byte로 저장해 네트워크 전송량을 줄였습니다.

```c#
public abstract class StateMachine : NetworkBehaviour
{
    [SerializeField] private NetworkVariable<byte> currentStateID; 
    private State currentState;
    protected Dictionary<byte, State> StateDict = new Dictionary<byte, State>();
    
    protected virtual void Update()
    {
        if (!IsServer) return; // Only Update by Sever
        
        currentState?.Tick(Time.deltaTime);
    }
    
    public void ChangeState(byte newStateID)
    {
        if (!IsServer) return; // Only Change State on Server

        if (StateDict.TryGetValue(newStateID, out State newState))
        {
            currentState?.Exit();
            currentState = newState;
            currentStateID.Value = newStateID;
            currentState?.Enter();
        }
    }

    public override void OnNetworkSpawn()
    {
        currentStateID.OnValueChanged += (prev, next) =>  // Sync Client State on currentStateID Update
        {
            if (!IsServer && StateDict.TryGetValue(next, out State newState))
            {
                currentState?.Exit();
                currentState = newState;
                currentState?.Enter();
            }
        };
    }
}
```

```c#
public abstract class State
{
    public abstract void Enter();
    public abstract void Tick(float deltaTime);
    public abstract void Exit();
}

```

`BossStateMachine`은 다양한 `State`를 생성하고, 특정 메시지를 받으면 해당 `State`로 전환할 수 있습니다.
```c#
public class BossStateMachine : StateMachine.StateMachine
{
    ...
    private BossIdleState IdleState { get; set; }
    private BossWakeState WakeState  { get; set; }
    private BossChaseState ChaseState { get; set; }
    private BossAttackState AttackState { get; set; }
    private BossDeathState DeathState { get; set; }
    private BossSleepState SleepState { get; set; }

    public override void OnNetworkSpawn() { ... }
    public override void OnNetworkDespawn() { ... }

    private void Init() { ... }

    private void OnDeathMessage() { ... }
    public void OnWakeMessage() { ... }
}
```

각 `State`는 `Enter`, `Tick`, `Exit`의 시점에 로직을 실행하고, 해당 로직은 `Server`에서만 실행됩니다.
```c#
public class BossIdleState : BossBaseState
{
    ...
    public BossIdleState(BossStateMachine stateMachine) : base(stateMachine) { }

    public override void Enter()
    {
        if (!StateMachine.IsServer) return; // Only on Server
        ...
    }

    public override void Tick(float deltaTime)
    {
        if (!StateMachine.IsServer) return; // Only on Server
        ...
    }

    public override void Exit() { }    
}
```

</details>

<br>

### 스킬 구현
<br>
<div align="center">
    <img src="/assets/img/DeadlineDuel/DeadlineDuel04.gif" width="40%" height="auto" style="border-top: 1px solid black;">
    <img src="/assets/img/DeadlineDuel/DeadlineDuel05.gif" width="40%" height="auto" style="border-top: 1px solid black;">
    <img src="/assets/img/DeadlineDuel/DeadlineDuel06.gif" width="40%" height="auto" style="border-top: 1px solid black;">
    <img src="/assets/img/DeadlineDuel/DeadlineDuel07.gif" width="40%" height="auto" style="border-top: 1px solid black;">
</div>


보스 몬스터의 스킬은 '로스트아크'의 레이드와 유사하게 제작하였습니다.<br>
보스가 스킬을 사용할 위치를 잠깐 보여준 뒤 이펙트를 재생하고 데미지를 주는 방식입니다.

<br>

<div align="center"><img src="/assets/img/DeadlineDuel/DeadlineDuel03.png" width="70%" height="auto"></div>


<details markdown="1">
<summary>자세히</summary>

```c#
public class BossSkill_Sample : BossSkill
{
    ...
    public override void ActivateSkill()
    {
        if (!IsServer) return;  // On Server
        
        bossPos = BossCore.transform.position;
        SyncBossPosClientRpc(bossPos);
        StartCoroutine(ExecuteSkillSequence());
    }

    private IEnumerator ExecuteSkillSequence()
    {
        BossCore.NetworkAnimator.SetTrigger(BossSkillHash);
        
        // Play Indicator
        yield return new WaitForSeconds(IndicatorTime);
        ActivateIndicatorClientRpc();
        
        // Play Effect and Damage Collider
        yield return new WaitForSeconds(EffectTime);
        ActivateSkillEffectClientRpc();
        ActivateDamageCollider(BossCore.BossStats.Atk.Value);
    }
    
    [ClientRpc] private void SyncBossPosClientRpc(Vector3 pos) { ... }
    [ClientRpc] protected override void ActivateIndicatorClientRpc() { ... }
    [ClientRpc] protected override void ActivateSkillEffectClientRpc() { ... }

    public override void ActivateDamageCollider(float bossAtk)
    {
        if (!IsServer) return;  // On Server
        ...
    }
}
```

</details>


## Troubleshoot

### Network Variable의 동기화 타이밍
`BossSkill_TargetShoot`은 플레이어의 현재 위치에 3번 레이저 공격을 하는 스킬입니다.
레이저가 떨어질 위치를 한 번 보여주고, 레이저 공격을 가합니다.
<img>
그러나 특정 상황에서 레이저의 인디케이터 위치가 보스 위치(`0,0,0`)로 설정되는 문제가 발생했습니다.

#### 문제 조건 분석
플레이를 반복한 결과 문제가 발생하는 조건은 다음과 같았습니다.
1. 호스트가 아닌 클라이언트에서 레이저 스킬을 발동할 때,
2. 3번의 레이저 중 첫 레이저만 (`0,0,0`),
3. 간헐적으로 발생하는 문제였습니다.

#### 로직 검토
기존의 스킬 로직은 다음과 같습니다.

1. 애니메이션 재생
2. 일정 시간 대기
3. Player 위칫값 받아옴
4. `Client`에서 `Indicator` 재생
5. 일정 시간 대기
6. `Client`에서 스킬 이팩트 재생
7. `Collider` 재생
8. 2~7을 3번 반복

<details markdown="1">
<summary>코드로 보기</summary>

```c#
private NetworkVariable<Vector3> targetPos;

private IEnumerator ExecuteSkillSequence()
{
    BossCore.NetworkAnimator.SetTrigger(BossSkillHash);

    for (int i = 0; i < 3; i++)
    {
        yield return new WaitForSeconds(IndicatorTime);
        targetPos = BossCore.BossCharacter.GetTargetPosition();

        ActivateIndicatorClientRpc();
        
        yield return new WaitForSeconds(EffectTime);
        ActivateSkillEffectClientRpc();
        ActivateDamageCollider(BossCore.BossStats.Atk.Value);
    }
}
```

</details>

높은 확률로 첫 `targetPos`의 초기화에 문제가 있을 것으로 생각해 값을 디버깅해 보았지만,
`NetworkVariable`임에도 불구하고 `Host`에서는 정상값, `Client`에서는 (`0,0,0`)이 출력되었습니다.

따라서, `NetworkVariable`의 동기화가 `ActivateIndicatorClientRpc()`보다 늦게 진행되어 발생하는 문제라는 결론을 내렸습니다.

#### 원인 분석
Unity NetCode의 `Network Variable`은 값이 변경되면, NetCode의 업데이트 주기마다 변경 사항이 전달됩니다. 따라서 업데이트가 진행되기 전에 해당 변수를 읽으면 변경되지 않은 값이 전달됩니다. <br>
이 경우 `ActivateIndicatorClientRpc()`가 Network Frame의 업데이트보다 빠르게 실행되어 `targetPos`의 값이 변경되기 전에 실행되었습니다.


#### 해결
NetworkVariable의 동기화 타이밍의 신뢰도가 떨어지므로 명시적으로 값을 설정하도록 변경하였습니다.

<details markdown="1">
<summary>코드로 보기</summary>

```c#
private Vector3 targetPos;
private IEnumerator ExecuteSkillSequence()
{
    BossCore.NetworkAnimator.SetTrigger(BossSkillHash);

    for (int i = 0; i < 3; i++)
    {
        yield return new WaitForSeconds(IndicatorTime);
        targetPos = BossCore.BossCharacter.GetTargetPosition();
        targetPos.y -= 0.2f;
        SyncTargetPosClientRpc(targetPos);
        
        ActivateIndicatorClientRpc();
        
        yield return new WaitForSeconds(EffectTime);
        ActivateSkillEffectClientRpc();
        ActivateDamageCollider(BossCore.BossStats.Atk.Value);
    }
}

[ClientRpc]
private void SyncTargetPosClientRpc(Vector3 pos)
{
    targetPos = pos;
}

```

</details>

#### 결과
<div align="center">
    <img src="/assets/img/DeadlineDuel/DeadlineDuel05.gif" width="60%" height="auto" style="border-top: 2px solid black;border-bottom: 1px solid black;">
</div>
`targetPos`의 동기화 타이밍이 보장되어 기존의 인디케이터 위치 에러가 해결되었습니다.

## Epilogue

이번 프로젝트에서는 NetCode를 활용하여 멀티 플레이 보스 시스템을 구현하였습니다. 처음으로 네트워크 게임 개발에 도전하며 시행착오도 많았지만,
그동안 배운 네트워크 프로그래밍 내용을 직접 적용하며 실습해 볼 수 있었습니다.
제가 좋아하는 장르를 직접 구현하는 과정이 즐거웠고, 결과물을 보며 큰 만족감을 느꼈습니다. <br><br>
앞으로는 아직 완성되지 못한 플레이어와 방해 로직의 개발을 마무리하고, 보스 몬스터에 더 다양한 스킬을 추가해 게임의 재미를 더할 예정입니다.