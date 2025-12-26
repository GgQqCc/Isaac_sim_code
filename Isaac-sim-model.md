# demonstrates the creation and destruction of a cube collision prim centered at the origin
# 演示了以原点为中心的立方体碰撞单元的创建与销毁

from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

cubePath = "/World/cube"

# Properties of the geometry
cubePosition = Gf.Vec3f(0.0, 0.0, 0.0)
cubeOrientation = Gf.Quatf(1.0)
cubeScale = Gf.Vec3f(1.0, 1.0, 1.0)
cubeSize = 2.0
cubeHalfExtent = cubeSize/2.0

cubeGeom = UsdGeom.Cube.Define(stage, cubePath)
cubeGeom.CreateSizeAttr(cubeSize)
cubeGeom.CreateExtentAttr([Gf.Vec3f(-cubeHalfExtent), Gf.Vec3f(cubeHalfExtent)])
cubeGeom.AddTranslateOp().Set(cubePosition)
cubeGeom.AddOrientOp().Set(cubeOrientation)
cubeGeom.AddScaleOp().Set(cubeScale)
cubePrim = cubeGeom.GetPrim()
UsdPhysics.CollisionAPI.Apply(cubePrim)

stage.RemovePrim(cubePath)



# Create a Sphere Collider

from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

spherePath = "/World/sphere"
sphereRadius = 2.0

sphereGeom = UsdGeom.Sphere.Define(stage, spherePath)
sphereGeom.CreateRadiusAttr(sphereRadius)
sphereGeom.AddTranslateOp().Set(Gf.Vec3f(0.0))
sphereGeom.AddOrientOp().Set(Gf.Quatf(1.0))
sphereGeom.AddScaleOp().Set(Gf.Vec3f(1.0))
spherePrim = sphereGeom.GetPrim()
UsdPhysics.CollisionAPI.Apply(spherePrim)


# Create a Box Collider
from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

boxPath = "/World/box"
boxDimensions = Gf.Vec3f(2.0, 3.0, 4.0)

boxGeom = UsdGeom.Cube.Define(stage, boxPath)
boxGeom.AddTranslateOp().Set(Gf.Vec3f(0.0))
boxGeom.AddOrientOp().Set(Gf.Quatf(1.0))
boxGeom.AddScaleOp().Set(boxDimensions)
boxPrim = boxGeom.GetPrim()
UsdPhysics.CollisionAPI.Apply(boxPrim)

# Create a Capsule Collider
from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

capsulePath = "/World/capsule"
capsuleRadius = 1.0
capsuleHeight = 10.0
capsuleAxis = UsdGeom.Tokens.y

capsuleGeom = UsdGeom.Capsule.Define(stage, capsulePath)
capsuleGeom.AddTranslateOp().Set(Gf.Vec3f(0.0))
capsuleGeom.AddOrientOp().Set(Gf.Quatf(1.0))
capsuleGeom.CreateRadiusAttr(capsuleRadius)
capsuleGeom.CreateHeightAttr(capsuleHeight)
capsuleGeom.CreateAxisAttr(capsuleAxis)
capsulePrim = capsuleGeom.GetPrim()
UsdPhysics.CollisionAPI.Apply(capsulePrim)



# Create a Cylinder Collider
from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd

# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

cylinderPath = "/World/cylinder"
cylinderHeight = 10.0
cylinderRadius = 0.5
cylinderAxis = UsdGeom.Tokens.y

cylinderGeom = UsdGeom.Cylinder.Define(stage, cylinderPath)
cylinderGeom.AddTranslateOp().Set(Gf.Vec3f(0.0))
cylinderGeom.AddOrientOp().Set(Gf.Quatf(1.0))
cylinderGeom.CreateHeightAttr(cylinderHeight)
cylinderGeom.CreateRadiusAttr(cylinderRadius)
cylinderGeom.CreateAxisAttr(cylinderAxis)
cylinderPrim = cylinderGeom.GetPrim()
UsdPhysics.CollisionAPI.Apply(cylinderPrim)



# 碰撞边距 (Margin) 0.1 防止穿模
cylinderPrim.CreateAttribute("physxConvexGeometry:margin", Sdf.ValueTypeNames.Float).Set(0.1)



# SETTING_COLLISION_APPROXIMATE_CYLINDERS = True，多面体来近似球面，提升性能，会忽视margin属性
from carb.settings import get_settings
from omni.physx.bindings._physx import SETTING_COLLISION_APPROXIMATE_CYLINDERS
# Use convex hull approximation
get_settings().set_bool(SETTING_COLLISION_APPROXIMATE_CYLINDERS, True)






# Create a Cone Collider

from pxr import Usd, UsdGeom, UsdPhysics, Gf
import omni.usd
# Create a stage
omni.usd.get_context().new_stage()
stage = omni.usd.get_context().get_stage()

# Define the root Xform (transformable object)
rootxform = UsdGeom.Xform.Define(stage, "/World")

conePath = "/World/cone"
coneRadius = 1.0
coneHeight = 10.0
coneAxis = UsdGeom.Tokens.y

coneGeom = UsdGeom.Cone.Define(stage, conePath)
coneGeom.AddTranslateOp().Set(Gf.Vec3f(0.0))
coneGeom.AddOrientOp().Set(Gf.Quatf(1.0))
coneGeom.CreateRadiusAttr(coneRadius)
coneGeom.CreateHeightAttr(coneHeight)
coneGeom.CreateAxisAttr(coneAxis)
conePrim = coneGeom.GetPrim()
UsdPhysics.CollisionAPI.Apply(conePrim)



