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
