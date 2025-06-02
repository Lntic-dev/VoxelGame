# 开放世界体素游戏

==gif加载可能需要时间==

![Image](https://github.com/user-attachments/assets/e1d64434-7b33-4751-9e47-36c0b75f55d1)

![Image](https://github.com/user-attachments/assets/daff7d90-c149-4de0-b4bc-a9084aec105f)

![Image](https://github.com/user-attachments/assets/883a3a8d-4466-4ea2-beed-ea82ba323641)

## 资源来源

3d资源使用MagicaVoxel进行体素模型建模

Blender进行模型的顶点合并和材质烘焙，以便于启用动态合批，同时进行进行骨骼动画的设计

部分2d资源Icon为AI生成（Google Image-FX、豆包）

音效来源为SFXR（8-bit和16-bit音效生成器）和符合CC0协议的免费资源

## 技术

### 世界

========================随机世界生成gif========================

![随机世界](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E9%9A%8F%E6%9C%BA%E4%B8%96%E7%95%8C.gif)

===========================================================================
#### 随机世界生成

- 基于Sunny Valley Studio的体素世界生成框架实现

  https://github.com/SunnyValleyStudio/Unity-2020-Voxel-World-Tutorial-Section-1-starter-project

- 计算顶点、三角形、方块网格绘制出方块，利用多次不同频率的泊松噪声生成地形，为了避免地形过于平滑叠加了域翘楚（domain warping）

- 使用了**责任链**设计模式对相应坐标点方块进行计算

```c#
public abstract class BlockLayerHandler : MonoBehaviour
{
    [SerializeField]
    private BlockLayerHandler Next;

    public bool Handle(ChunkData chunkData, int x, int y, int z, int surfaceHeightNoise, Vector2Int mapSeedOffset)
    {
        if (TryHandling(chunkData, x, y, z, surfaceHeightNoise, mapSeedOffset))
            return true;
        if (Next != null)
            return Next.Handle(chunkData, x, y, z, surfaceHeightNoise, mapSeedOffset);
        return false;
    }

    protected abstract bool TryHandling(ChunkData chunkData, int x, int y, int z, int surfaceHeightNoise, Vector2Int mapSeedOffset);
}
```

#### 世界加载

- 利用携程不断检查玩家的位置，利用异步任务计算区块包含的方块数据，在携程中进行区块的渲染
- 设计了一格区块的缓存，玩家移动进行区块加载时，优先激活缓存中的区块
- 利用对象池管理区块的消失和创建

#### 世界种子

通过对泊松噪声添加offset实现种子的效果

## 环境

===========================================================================

![昼夜更替](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E6%98%BC%E5%A4%9C%E6%9B%B4%E6%9B%BF.gif)

===========================================================================

### 昼夜更替

通过将Fog、Ambient、定向光、天空盒的颜色随时间设置为渐变色中相应的颜色实现昼夜更替的效果

### 环境特效

使用粒子系统的拖尾，设定双曲线间随机速度实现风的线条吹过的效果

## 配置
===========================================================================

![设置功能](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E8%AE%BE%E7%BD%AE%E5%8A%9F%E8%83%BD.gif)

===========================================================================
- 通过修改单例对象World的可见范围参数，并重新执行世界绘制函数进行渲染区块的改变
- 通过修改自定义ShaderGraph中Water材质的Boolean改变水方块材质的输出类型
- 通过修改单例对象AudioManager中的AudioSource中相应volume实现音量的调节

## 保存

===========================================================================

![保存功能](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E4%BF%9D%E5%AD%98%E5%8A%9F%E8%83%BD.gif)

===========================================================================
### 序列化与反序列化

- 使用Newtonsoft进行数据的序列化与反序列化
- 自定义Vector3和Quaternion序列化与反序列化逻辑，用于存储角色的Transform的信息

## 对话系统

### XNode

将对话节点分为起始、内容、分支、事件、结束等节点，利用XNode可视化节点配置过程

设计全局执行器，对话时将对话资产注入全局执行器逐个节点执行对话内容

===========================================================================

![Image](https://github.com/user-attachments/assets/b47bf409-c82b-4857-8b3d-ac7c84bd4452)

===========================================================================

```c#
public class DialogueNodeBase : Node {

    [Input(ShowBackingValue.Never)] public ConnectionType input;
    [Output] public ConnectionType output;
    protected override void Init() {
		base.Init();
	}

	public override object GetValue(NodePort port) {
		return null;
	}

    public virtual void Execute() { }

    public void SingleMoveNext()
    {
        NodePort port = GetOutputPort("output");

        if (port.IsConnected)
        {
            (port.Connection.node as DialogueNodeBase)?.Execute();
        }
        else
        {
            Debug.Log("没有下一个节点，或者连接错误");
        }
    }

    public virtual void BranchMoveNext(int index)
    {
        NodePort[] ports = DynamicOutputs.ToArray();
        if (ports[index].IsConnected)
        {
            DialogueNodeBase nextNode = (DialogueNodeBase)ports[index].Connection.node;
            nextNode.Execute();
        }
        else
        {
            Debug.LogError("No connected node");
        }
    }
}
```



## 背包系统

===========================================================================

![商店交易](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E5%95%86%E5%BA%97%E4%BA%A4%E6%98%93.gif)

===========================================================================

- 使用ScriptableObject存储角色背包信息
- 使用IPointerClickHandler, IBeginDragHandler, IDragHandler, IEndDragHandler等接口实现鼠标在背包内的交互动作
- 通过对物品的配置可以实现**是否可售卖、是否可堆叠、是否可使用**

### 宝箱

宝箱数据相互隔离

===========================================================================

![宝箱数据隔离](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E5%AE%9D%E7%AE%B1%E6%95%B0%E6%8D%AE%E9%9A%94%E7%A6%BB.gif)

===========================================================================

## 角色架构

===========================================================================

![Image](https://github.com/user-attachments/assets/7adb0da8-1f2d-4d88-bce1-889c095047fa)

===========================================================================

基于组件模式，将角色的功能进行拆分

Controller管理所有组件，一切逻辑通过Controller进行

Controller继承自泛型类BaseCharacterController，在继承时指定泛型类型

```c#
public abstract class BaseCharacterController<TConfig, TMovement, TCombat, TAnim, TMaterals, TStateMachine> : MonoBehaviour, IBaseCharacterController
    where TConfig : BaseCharacterConfig
    where TMovement : BaseCharacterMovement
    where TCombat : BaseCharacterCombat
    where TAnim : BaseCharacterAnimation
    where TMaterals : BaseCharacterMaterals
    where TStateMachine : BaseCharacterStateMachine, new()
{
    public TConfig Config;
    [HideInInspector] public CharacterStats Stats { get; set; }
    [HideInInspector] public CharacterEmoji Emoji { get; set; }
    [HideInInspector] public CharacterParticle Particle { get; set; }
    [HideInInspector] public TMovement Movement { get; set; }
    [HideInInspector] public TCombat Combat { get; set; }
    [HideInInspector] public TAnim Anim { get; set; }
    [HideInInspector] public TMaterals Materals { get; set; }
    [HideInInspector] public TStateMachine StateMachine { get; set; }

    BaseCharacterConfig IBaseCharacterController.Config => Config;
    BaseCharacterMovement IBaseCharacterController.Movement => Movement;
    BaseCharacterCombat IBaseCharacterController.Combat => Combat;
    BaseCharacterAnimation IBaseCharacterController.Anim => Anim;
    BaseCharacterMaterals IBaseCharacterController.Materals => Materals;
    BaseCharacterStateMachine IBaseCharacterController.StateMachine => StateMachine;
    protected virtual void Awake()
    {
        CacheComponents();
        Initialize();
    }
    protected virtual void Initialize()
    {
        InitializeStateMachine();
        SetupDependencies();
        InitState();
    }
    protected virtual void Update()
    {
        StateMachine.Update();
    }
    protected abstract void CacheComponents();
    protected virtual void InitializeStateMachine()
    {
        StateMachine = new TStateMachine();

        var states = new Dictionary<CharacterStates, ICharecterState>
        {
            { CharacterStates.DEAD, new DeadState(this) },
            { CharacterStates.GETHITUP, new GetHitUpState(this) },
            { CharacterStates.FALL, new FallState(this) },
            { CharacterStates.STANDUP, new StandUpState(this) },
            { CharacterStates.SKILL, new SkillState(this) }
        };
        StateMachine.SetStates(states);
    }
    protected virtual void SetupDependencies()
    {
        Movement.Initialize(this);
        Combat.Initialize(this);
        Anim.Initialize(this);
        StateMachine.Initialize();
    }
    void InitState()
    {
        StateMachine.SwitchToInit();
    }
}
```

## 有限状态机

基于状态模式

设计ICharecterState

```c#
public interface ICharecterState
{
    void OnEnter();
    void OnUpdate();
    void OnExit();
}
```

- 定义通用状态GenetalStates管理跳跃、坠落、受击等共用状态
- 定义玩家状态PlayerStates管理移动、跑步、技能状态
- 定义敌人状态EnemyStates管理追击、静置、技能状态

在状态的生命周期内执行相应状态的期望逻辑

设计**状态机**管理状态转换规则CheckCondition和状态转换的方法SwitchTo

```c#
public class BaseCharacterStateMachine
{
    protected Dictionary<CharacterStates, ICharecterState> States = new Dictionary<CharacterStates, ICharecterState>();
    public ICharecterState CurrentState { get; private set; }
    public virtual void SwitchTo(CharacterStates newStateType)
    {
        if (CheckCondition(newStateType))
        {
            ICharecterState newState;
            if (States.TryGetValue(newStateType, out newState))
            {
                CurrentState?.OnExit();
                CurrentState = newState;
                CurrentState.OnEnter();
            }
        }
    }
    public void Update()
    {
        CurrentState?.OnUpdate();
    }
    protected virtual bool CheckCondition(CharacterStates targetState)
    {
        return true;
    }
}

```



## 换装

===========================================================================

![换装](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E6%8D%A2%E8%A3%85.gif)

===========================================================================

各个装备的模型绑定同一套骨骼，导入时使用同一个Avatar，通过修改角色Skinned Mesh Renderer的网格和材质球实现角色换装

物品只能拖拽到对应的装备栏

## 战斗系统

===========================================================================

![Boss战斗](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E6%8D%A2%E8%A3%85.gif)

![技能战斗](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E6%8A%80%E8%83%BD%E6%88%98%E6%96%97.gif)

===========================================================================

利用TrailRender进行武器尾迹的绘制

通过动画事件开启TrailRender和武器的碰撞体实现攻击检测的效果（敌人的攻击检测仅根据朝向和与玩家的距离计算）

===========================================================================

![被击败](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E8%A2%AB%E5%87%BB%E8%B4%A5.gif)

===========================================================================

死亡时，通过**观察者模式**呼叫事件中心的死亡时间，事件中心执行所有注册的方法组，即敌人注册到事件中心的胜利时的方法；

击败玩家时，所有敌人进入胜利状态

玩家重生时，敌人的状态将被重置

### 技能系统

基于AETimeline技能编辑器，添加了攻击范围检测的轨道和Clip，能够实现在特定位置创建特定持续时间的碰撞体，用于实现伤害的检测

AETimeline利用状态模式设计各个轨道的进入、执行、退出状态

设计技能执行器逐帧执行技能资产，根据帧数据检测判断轨道中Clip的执行

===========================================================================

![Image](https://github.com/user-attachments/assets/33f4bab4-a455-4c88-8ab3-46dbb778258e)

===========================================================================

## 技能树

===========================================================================

![技能树](https://github.com/Lntic-dev/VoxelGame/blob/master/gif/%E6%8A%80%E8%83%BD%E6%A0%91.gif)

===========================================================================

目前只实现了简单的技能树系统，只设计了一个技能

- 技能加点时，优先检查上一个节点的等级，满足条件才可加点
- 技能学习后，存储在Model中的List中
- 技能配置时，通过Controller访问Model中的List，将对应的技能信息注册给主角

## 物品交互



## 对象池

针对频繁需要创建销毁的对象创建相应的对象池，创建了**区块池、树木池、敌人池、特效池**

优先从缓存中取出对象，避免大量新能消耗

## ShaderGraph

### 角色

===========================================================================

![Image](https://github.com/user-attachments/assets/c33e0fe6-02bc-4d5d-91f7-4267c228e882)

===========================================================================

通过移动的噪声图加Fresnel实现霸体时角色的身体闪烁黄色光芒的效果

通过脚本设置Boolean，设置受击、霸体状态，实现角色的**受击闪白**和**霸体闪烁**效果

### 物品可交互

===========================================================================

![Image](https://github.com/user-attachments/assets/16bdc9f6-1773-48dd-b61e-f82cefb1da66)

===========================================================================

### 风格化水

基于https://ameye.dev/notes/stylized-water-shader/添加了是否开启高级特效的功能



## UI设计

UI基于MVC架构。

- View管理UI组件的获取和UI的更新
- Model管理UI相关的数据，包括初始化和存取
- Controller依赖View和Model进行逻辑处理

通过单例对象UIManager管理全部UI的Controller，UI显示时添加到UIManager的Stack中，关闭UI时，默认从栈顶取出UI进行关闭

## 摄像机

使用Cinemachine设置多个镜头，包括默认第三人称跟随镜头、对话镜头、战斗锁定镜头、Boss出现镜头

通过脚本指定LookAt和Follow，并挂载Cinemachine Collider组件避免镜头穿模

通过对摄像机优先级的调节实现镜头的切换

