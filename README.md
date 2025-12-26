# Isaac_sim_code
物理引擎（PhysX）是通过判断两个几何形状是否在空间中存在交集来计算碰撞的。
none (精确网格 - Triangle Mesh)
本质原理： 引擎直接读取 Mesh 的每一个三角面片（Triangle）。碰撞检测时，它会逐一计算对方物体是否穿过了这些三角形。
meshSimplification (网格简化)
本质原理： 在后台运行一个减面算法（类似 Blender 的 Decimate），生成一个点数更少的近似网格。
convexHull (凸包) —— 最常用  [包含所有顶点的最小凸多面体]
本质原理： 想象把物体塞进一个有弹性的气球里。无论物体内部有多少洞或凹陷，气球只会包裹住最外层的凸点。
convexDecomposition (凸分解)
本质原理： 针对凹物体。它利用 V-HACD 算法将一个凹物体自动切割成多个小的凸包。
物理表现： 比如一个“凹”字，会被拆解为三个矩形（凸包）。
应用： 它是解决“既要性能又要精确”的最佳方案，适合机械臂的夹爪、容器、椅子。
 boundingSphere (包围球)本质原理： 找到一个球体，刚好能把整个模型装进去。\
 计算逻辑： 只需要计算两个球心之间的距离 $d$ 是否小于两个半径之和 r_1 + r_2。这是计算机图形学中最简单的计算。
 缺点： 极度不精确。长条状物体会变成巨大的球，导致机器人还没碰到物体就被弹开了。
 6. boundingCube (包围盒)
 本质原理： 计算物体在 X, Y, Z 三个轴向上的最大和最小坐标，形成一个对齐坐标轴的盒子。
 计算逻辑： 只涉及简单的数值比较（x_{min1} > x_{max2} 等）。
 应用： 常用于初步筛选（Broadphase），即：如果两个包围盒都没撞上，那么里面的精细网格肯定没撞上。

<img width="1701" height="747" alt="image" src="https://github.com/user-attachments/assets/eb09b0a0-a9ea-43cc-be6c-e369dda1fa9f" />
全网格、简化网格
 

<img width="1701" height="490" alt="image" src="https://github.com/user-attachments/assets/46dd0d84-cc87-432b-a918-01f9fba4a2b9" />
凸包、凹包、包围球、包围盒


1. 为什么要搞“合并碰撞”？
想象你用 100 块小积木拼成了一个机器人的手掌。
普通做法： 给每块积木都套一个碰撞盒。物理引擎每秒要计算 100 次碰撞。如果积木之间有微小的缝隙，球滚过去会“卡一下”，甚至积木之间会互相打架导致手掌“炸开”。
合并做法： 物理引擎把这 100 块积木看成一个整体，只在最外面包一层“保鲜膜”。
操作的核心逻辑： 既然它们是粘在一起不动的，那就只算一次碰撞，省电、省 CPU、更丝滑。
2. 怎么操作？（三步走）
第一步：找个“桶”（Xform）把积木装起来
你需要一个父节点（比如 /World/rigidBody）。把所有属于同一个零件的 Mesh 全丢进去。
第二步：告诉引擎“我要包膜”
你在父节点上点一下（代码里就是 Apply 那几行），下达指令：
“我要合并”：PhysxMeshMergeCollisionAPI（开启合并功能）。
“怎么包膜”：MeshCollisionAPI。
选 convexHull：包一层紧绷的保鲜膜（最快）。
选 convexDecomposition：包一层贴合轮廓的保鲜膜（能进洞，稍微慢点）。
第三步：选哪些积木要包，哪些不要（关键）
这就是你代码最后那几行的逻辑：
Include（包含）：直接指定父节点。桶里所有的积木默认都要包进去。
Exclude（排除）：有些积木（比如传感器、装饰灯）不需要碰撞，或者它会干扰物理计算。你把它挑出来，保鲜膜就会绕过它。
优化以后（你的代码效果）： 虽然你有三个立方体（Mesh0, 1, 2），但因为你排除了 Mesh1，你会发现：
Mesh0 和 Mesh2 被一个巨大的红色罩子罩在一起了。
Mesh1 身上什么都没有，它就像个鬼魂，手摸不到它。

网格碰撞代码
 from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

halfSize = 0.5

points = [
    Gf.Vec3f(halfSize, -halfSize, -halfSize),
    Gf.Vec3f(halfSize, halfSize, -halfSize),
    Gf.Vec3f(halfSize, halfSize, halfSize),
    Gf.Vec3f(halfSize, -halfSize, halfSize),
    Gf.Vec3f(-halfSize, -halfSize, -halfSize),
    Gf.Vec3f(-halfSize, halfSize, -halfSize),
    Gf.Vec3f(-halfSize, halfSize, halfSize),
    Gf.Vec3f(-halfSize, -halfSize, halfSize),
]

normals = [
    Gf.Vec3f(1, 0, 0),
    Gf.Vec3f(1, 0, 0),
    Gf.Vec3f(1, 0, 0),
    Gf.Vec3f(1, 0, 0),
    Gf.Vec3f(-1, 0, 0),
    Gf.Vec3f(-1, 0, 0),
    Gf.Vec3f(-1, 0, 0),
    Gf.Vec3f(-1, 0, 0),
]

indices = [
    0, 1, 2, 3,
    1, 5, 6, 2,
    3, 2, 6, 7,
    0, 3, 7, 4,
    1, 0, 4, 5,
    5, 4, 7, 6
]

vertexCounts = [4, 4, 4, 4, 4, 4]

meshOptions = [
    UsdPhysics.Tokens.none,
    UsdPhysics.Tokens.meshSimplification,
    UsdPhysics.Tokens.convexHull,
    UsdPhysics.Tokens.convexDecomposition,
    UsdPhysics.Tokens.boundingSphere,
    UsdPhysics.Tokens.boundingCube
]

for i in range(6):

    meshPath = "/World/mesh" + str(i)
    mesh = UsdGeom.Mesh.Define(stage, meshPath)
    mesh.CreateFaceVertexCountsAttr(vertexCounts)
    mesh.CreateFaceVertexIndicesAttr(indices)
    mesh.CreatePointsAttr(points)
    mesh.CreateDoubleSidedAttr(False)
    mesh.CreateNormalsAttr(normals)
    meshPrim = mesh.GetPrim()

    collisionAPI = UsdPhysics.CollisionAPI.Apply(meshPrim)
    meshCollisionAPI = UsdPhysics.MeshCollisionAPI.Apply(meshPrim)
    meshCollisionAPI.GetApproximationAttr().Set(meshOptions[i])
