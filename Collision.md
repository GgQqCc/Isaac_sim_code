碰撞组黑白名单：
from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# The goal is to create three colliders (box0, box1, box2) with collision
# only between box0 and box1.
# This is achieved by creating three collision groups such that each group
# contains a different box.
# Filtering is used to filter out (or in) collision between collision groups.
# The process is repeated without inverted collision filtering (opt-out) and with
# inverted collision group filtering (opt-in)

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

useInvertedCollisionGroups = [False, True]

for k in range(2):

    # Three collision groups. Each group has a list of members and
    # a list of other collision groups used to filter collision.
    collisionGroupPaths = [
        "/World/collisionGroup0",
        "/World/collisionGroup1",
        "/World/collisionGroup2"]
    collisionGroupIncludesRel = [None] * 3
    collisionGroupFilteredRels = [None] * 3

    # Three boxes. Each box will be added to its corresponding collision group.
    boxPaths = ["/World/boxA", "/World/boxB", "/World/boxC"]

    # Set the flag that configures inverted collision group filtering as required.
    if useInvertedCollisionGroups[k]:
        defaultPrimPath = str(stage.GetDefaultPrim().GetPath())
        physicsScenePath = defaultPrimPath + "/physicsScene"
        physicsScene = UsdPhysics.Scene.Define(stage, physicsScenePath)
        physicsScenePrim = physicsScene.GetPrim()
        physxSceneAPI = PhysxSchema.PhysxSceneAPI.Apply(physicsScenePrim)
        physxSceneAPI.CreateInvertCollisionGroupFilterAttr(True)

    # Create three colliders
    for i in range(3):

        boxGeom = UsdGeom.Cube.Define(stage, boxPaths[i])
        boxPrim = boxGeom.GetPrim()
        UsdPhysics.RigidBodyAPI.Apply(boxPrim)
        UsdPhysics.CollisionAPI.Apply(boxPrim)

    # Create three collision groups.
    # For each collision group create a list that will store the colliders in the group
    # For each collision group create a list of other collision groups
    for i in range(3):
        collisionGroup = UsdPhysics.CollisionGroup.Define(stage, collisionGroupPaths[i])
        collisionGroupPrim = collisionGroup.GetPrim()
        collectionAPI = Usd.CollectionAPI.Apply(
            collisionGroupPrim,
            UsdPhysics.Tokens.colliders
        )
        collisionGroupIncludesRel[i] = collectionAPI.CreateIncludesRel()
        collisionGroupFilteredRels[i] = collisionGroup.CreateFilteredGroupsRel()

    # Add box0 to collisionGroup0, box1 to collisonGroup1, box2 to collisionGroup2
    collisionGroupIncludesRel[0].AddTarget(boxPaths[0])
    collisionGroupIncludesRel[1].AddTarget(boxPaths[1])
    collisionGroupIncludesRel[2].AddTarget(boxPaths[2])

    if useInvertedCollisionGroups[k]:
        # Collision only between box0 and box1 is achieved by fil tering in
        # collision between collisionGroup0 and collisionGroup2
        collisionGroupFilteredRels[0].AddTarget(collisionGroupPaths[1])
    else:
        # Collision only between box0 and box1 is achieved by filtering out
        # collision between collisionGroup0 and collisionGroup2 and between
        # collisionGroup1 and collisionGroup2
        collisionGroupFilteredRels[0].AddTarget(collisionGroupPaths[2])
        collisionGroupFilteredRels[1].AddTarget(collisionGroupPaths[2])




点对点精准不碰撞 （避免手工安上物体的自碰撞）
from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

# The goal is to create three colliders (box0, box1, box2) with
# collision only between box0 and box1.
# This is achieved by disabling collision between box0 and box2 and
# between box1 and box2

# Create three collider prims
boxPaths = ["/World/boxA", "/World/boxB", "/World/boxC"]
boxPrims = [None] * 3
for i in range(3):
    boxGeom = UsdGeom.Cube.Define(stage, boxPaths[i])
    boxPrim = boxGeom.GetPrim()
    UsdPhysics.RigidBodyAPI.Apply(boxPrim)
    UsdPhysics.CollisionAPI.Apply(boxPrim)
    boxPrims[i] = boxPrim

# Disable collision between box0 and box2
filteredPairsAPIBox0 = UsdPhysics.FilteredPairsAPI.Apply(boxPrims[0])
filteredPairsAPIBox0.CreateFilteredPairsRel().AddTarget(boxPaths[2])

# Disable collision between box1 and box2
filteredPairsAPIBox1 = UsdPhysics.FilteredPairsAPI.Apply(boxPrims[1])
filteredPairsAPIBox1.CreateFilteredPairsRel().AddTarget(boxPaths[2])

<img width="1522" height="1169" alt="image" src="https://github.com/user-attachments/assets/e7b98640-7276-47fb-b859-e510c42d17e4" />

1. 接触偏移量 (Contact Offset)
本质原理：设置“预警力场”
接触偏移量定义了在两个物体表面真正接触之前，物理引擎多远就开始提前生成“接触点”进行计算。
解决的问题：
高速穿透（隧道效应）： 当物体移动极快或非常薄时，在第一个时间步长物体在左边，第二个时间步长物体就在右边了（穿过了），引擎没来得及发现碰撞。增加此值相当于扩大了检测范围，让引擎提前发现碰撞趋势。
模拟稳定性： 提前准备好接触数据，可以让物理求解器（Solver）有更充裕的时间平滑地处理排斥力，而不是在重合瞬间猛烈弹开。
注意： 该值增加会生成更多潜在接触点，显著增加计算压力，导致性能下降。

2. 休息偏移量 (Rest Offset)
本质原理：定义“虚拟表面”
休息偏移量决定了当物体处于“静止”状态时，它们之间到底留多大的缝隙。
解决的问题：
视觉与物理的脱节： * 浮空感： 如果你的碰撞几何体（Collision Mesh）比渲染网格（Visual Mesh）大，物体看起来会悬浮在地面上。
嵌合感：如果碰撞几何体比渲染网格小，物体看起来会陷入地面。
修正方法：
通过设置负值的休息偏移量，可以让视觉上的物体看起来刚好紧贴地面，即使物理上它们的碰撞体之间还有微小距离。

from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

cubePath = "/World/cube"

# rest and collision offset
restOffset = 0.0
contactOffset = 0.02

# Properties of the geometry
cubePosition = Gf.Vec3f(2.0, 0.0, 0.0)
cubeOrientation = Gf.Quatf(1.0)
cubeScale = Gf.Vec3f(1.0, 1.0, 1.0)
cubeSize = 2.0
cubeHalfExtent = cubeSize/2.0

# Create a cube geometry prim
cubeGeom = UsdGeom.Cube.Define(stage, cubePath)
cubeGeom.CreateSizeAttr(cubeSize)
cubeGeom.CreateExtentAttr([Gf.Vec3f(-cubeHalfExtent), Gf.Vec3f(cubeHalfExtent)])
cubeGeom.AddTranslateOp().Set(cubePosition)
cubeGeom.AddOrientOp().Set(cubeOrientation)
cubeGeom.AddScaleOp().Set(cubeScale)
cubePrim = cubeGeom.GetPrim()
UsdPhysics.CollisionAPI.Apply(cubePrim)

# Apply PhysxSchema.PhysxCollisionAPI to the cube prim
physxCollisionAPI = PhysxSchema.PhysxCollisionAPI.Apply(cubePrim)
physxCollisionAPI.CreateRestOffsetAttr(restOffset)
physxCollisionAPI.CreateContactOffsetAttr(contactOffset)


尺寸不能差异过大 
仿真需求,推荐碰撞体类型,备注
机器人移动底盘轮子,Cylinder,保证平滑滚动，不颠簸
机械臂连杆/外壳,Convex Hull,性能与精度的平衡点
工厂地面/大型建筑,Triangle Mesh,节省内存，支持复杂静态地形
精密螺栓、复杂装配零件,SDF Mesh,避免穿透，支持高精度动态交互
简单的箱子/球类,Box / Sphere,能用基础形状就绝不用复杂的
