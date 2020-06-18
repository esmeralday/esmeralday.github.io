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

<img src="https://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/cube.png" alt="Visual Model" width="300"/> <img src="https://github.com/esmeralday/esmeralday.github.io/blob/master/_includes/images/mechanicalModel.png" alt="Mechanical Model" width="300"/>

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
