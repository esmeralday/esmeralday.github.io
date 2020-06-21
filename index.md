## Introduction


## Modelling

Modelling soft robots is a problematic task as many simulation softwares are unable to realistically mimic the properties of soft, flexible materials. For this project, a soft robotic limb is modeled in simulation before learning algorithms can be tested on it. To achieve realistic results it is important that the software used has certain properties; for example, the ability to accurately model soft materials and the ability to use it to test the machine learning models.

A software called SOFA (Simulation Open Framework Architecture) which is primarily used in simulation in medical fields has a soft robotics plugin which can be used to achieve the task at hand. Several tutorials can be found online on Youtube and some documentation is also included with the software when it is downloaded.

{% include youtubePlayer.html id="91G-knmgnxM" %}

## First Steps

After downloading the SOFA software along with the soft robotics plugin I began by setting up some simple scenes that can be animated on SOFA. To begin we can start by experimenting with an object, such as a cuboid, and a floor. The object falls until it hits the floor. The object can be translated to the right or left, causing it to miss the floor; in this instance it will remain in free fall. Every scene starts with a root node, and other objects are added in relation to this. Gravity is also modelled in the code.



### Simple Scene

SOFA is able to animate .pyscn files. These are python files with an extra function named `createScene` that takes a root node as an input. The root node is the main parent node of the node tree. A scene consists of an ordered tree of nodes with parent/child relationships. Every node is built of components with names and features.

To set up this scene prefab objects are used. In this example the cube and floor prefabs are implemented and animated. The cube falls and stops upon collision with the floor. A contact distance is added as a buffer. Although this may seem appear unrealistic when the cube stops above the floor (i.e. before appearing to touch it), it is important to add this buffer to give the simulation enough time to compute the collision calculations.

```python

def createScene(rootNode):
    """This is my first scene"""
    MainHeader(rootNode, gravity=[0.0,-981.0,0.0])
    ContactHeader(rootNode, alarmDistance=15, contactDistance=10)

    Floor(rootNode,
          translation=[0.0,-160.0,0.0],
          isAStaticObject=True)

    Cube(rootNode,
          translation=[0.0,0.0,0.0],
          uniformScale=20.0)


    return rootNode

```

Next, the cube prefab is replaced with an implimentation of a cube model from scratch.

### Object Models 

There are three types of models for simulation in SOFA,

1. Mechanical model
2. Visual model
3. Collision model

<img src="http://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/cube.png" alt="Visual Model" width="300"/> <img src="http://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/mechanicalModel.png" alt="Mechanical Model" width="300"/>

<img src="_includes/images/cube.png" alt="Visual Model" width="300"/>

These models are able to correspond to work together and represent the properties of the object. 

The full code for this scene can be found on the [Defrost robotics github](https://github.com/SofaDefrost/SoftRobots/blob/master/docs/tutorials/FirstSteps/firststeps-tuto.pyscn).

### Manipulating Objects

An object can be moved in the scene by either modigying the translation vector feature in the code, or by using the SOFA gui. On the left hand panel, expand the menu for the object and double-click on the `MechanicalObject mstate` variable. Then go to the `Transofrmation` tab and modify the translation values. This will only apply to the current session and is does not affect the code.

### Modelling Deformable Objects

Due to the material properties of soft robots the simulation is only able to approxiamte the true behaviour of a deformable object. The most essential material properties to take into account are the stiffness and elasticity. The material modeled here will be silicone.

The behaviour of silicone is simulated using FEM along with a tetrahedral mesh provided with SOFA.

There are two important parameters to note which will be used in the code: Poisson's ratio and Young's modulus.

Poisson's ratio is a measure of the ratio of expansion perpendicular to contraction of a material.

ADD PHOTO!!!!!!!!!!!

Young's modulus indicates a material's ability to remain elastic under tension or compression.

ADD PHOTO!!!!!!!!!!!

## Soft Actuator

To model and simulate the behaviour of a soft actuator a volumetric mesh is used. This is computed using a surface mesh or image and the `CGALPlugin`.
The actuator mesh is loaded into SOFA and the force field can be turned on in the view panel to show the triangles making up the object.  The volumetric mesh is the .vtk file while a secondary surface mesh .stl file is used to model the surface. When the scene is animated using the code above the soft actuator will begin to free fall under gravity so it can be modified to have a fixed end on one side holding it in place. This is done by adding a `FixedBox`.

<video src="http://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/fixedBox.mov" width="320" height="200" controls preload></video>


```python
from stlib.physics.constraint import FixedBox

def Finger(parentNode=None, name="Finger",
           rotation=[0.0, 0.0, 0.0], translation=[0.0, 0.0, 0.0],
           fixingBox=[0.0,0.0,0.0], pullPointLocation=[0.0,0.0,0.0]):

    finger = Node(parentNode, name)
    eobject = ElasticMaterialObject(finger,
                               volumeMeshFileName="data/mesh/finger.vtk",
                               poissonRatio=0.3,
                               youngModulus=18000,
                               totalMass=0.5,
                               surfaceColor=[0.0, 0.8, 0.7],
                               surfaceMeshFileName="data/mesh/finger.stl",
                               rotation=rotation,
                               translation=translation)

    FixedBox(eobject, atPositions=[-10,-10,-10,10,10,15],
                      doVisualization=True)

    return finger
```

### Control

The finger will be actuated using cables that span from the base to the top of the finger. The cable is modeled using a template provided by the SoftRobot plugin called `PullingCable`. This is added to the finger function shown above.
A controller can be made using the PythonScriptController class and then added to the script.

```python
from softrobots.actuators import PullingCable
    ...
import Sofa

class FingerController(Sofa.PythonScriptController):
    def __init__(self, node, cable):
        self.cableconstraintvalue = cable.getObject("CableConstraint").findData('value')
        self.name = "FingerController"

    def onKeyPressed(self,c):
        if (c == "+"):
            self.cableconstraintvalue.value =  self.cableconstraintvalue.value[0][0] + 1.
        if (c == "-"):
            self.cableconstraintvalue.value =  self.cableconstraintvalue.value[0][0] - 1.
    
def Finger(parentNode=None, name="Finger",
           rotation=[0.0, 0.0, 0.0], translation=[0.0, 0.0, 0.0],
           fixingBox=[0.0,0.0,0.0], pullPointLocation=[0.0,0.0,0.0]):
           ...
           
    cable = PullingCable(eobject, cableGeometry=loadPointListFromFile("data/mesh/cable.json"))
    
    FingerController(eobject, cable) # This causes a segmentation fault but the controller still works without it?
```
    
<video src="https://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/control.mov" width="320" height="200" controls preload></video>    
    
    
#### Note: Some issues with visualising the cables, although they are working as they should in terms of manipulating the actuator.

### Collisions

Collisions are computationally expensive in SOFA so these need to be defined in the code based on the object's geometric properties. This is done by adding a collision mesh. Self-collisions are handled in this way as well. 
Two more finger actuators are added to make a three pronged flexible and compliant grip. 

<img src="http://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/fingerWithMeshCG.png" alt="Visual Model" width="300"/> <img src="http://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/gripperWithMeshCG.png" alt="Visual Model" width="300"/>

```python
def Finger(parentNode):
    ## ... the finger
    ## ... the cable..
    ## ... the controller..

    CollisionMesh(eobject,
         surfaceMeshFileName="data/mesh/finger.stl", name="part0", collisionGroup=1)


    CollisionMesh(eobject,
             surfaceMeshFileName="data/mesh/fingerCollision_part1.stl", name="part1", collisionGroup=1)

    CollisionMesh(eobject,
              surfaceMeshFileName="data/mesh/fingerCollision_part2.stl", name="part2", collisionGroup=2)

```

The CollisionMesh is imported from a template which is part of the STLIB plugin. This makes it easier to handle collision and self-collision events as well as contact handling.
The mesh is then added to the `finger.py` file which describes the soft actuator. In total three seperate meshes are needed to cover the finger. Each mesh is responsible for a specified `collisionGroup`. The `collisionMesh` is made of a set of triangles; triangles from different collision groups can interact and overlap.
The collision groups handle any contact between objects. In the video below, the finger is shown to comply and deform when it hits the rigid cube. There is also a `contactDistance` parameter which was also used when testing the cube falling onto the floor. The contact distance ensures there is enough time to carry out calculations when a collision occurs. If the contact distance is set to zero the cube falling into the floor might have already gone into the floor by the time the computation is complete.

<video src="https://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/deformations.mov" width="320" height="200" controls preload></video>

### Final Scene

This can be used to pick up objects. The full simualtion is composed of two python files and a .pyscn file.

- gripper.py where the finger objects are positioned and instantiated. 
- finger.py where the soft actuator is modelled using the mesh and material properties.
- cablegripper.pyscn where the scene is put together and loaded into SOFA.

<video src="https://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/pickingUpCube.mov" width="320" height="200" controls preload></video>

## TO DO!!!!!!!!





    










# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/esmeralday/esmeralday.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
