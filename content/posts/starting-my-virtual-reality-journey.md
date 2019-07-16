---
title: "Starting My Virtual Reality Journey"
date: 2019-06-12T07:47:30+01:00
draft: true
---

Unity, macOs, Google Cardboard (eBay).

Getting Unity:
Go to [unity website](https://unity.com/) and download the UnityHub. Install and activate license (will also need to create a Unity account). Use UnityHub to install the Unity Editor, as well as Visual Studio for macOS, Android, Mac and WebGL build support. Get coffee.

How does this thing work? Follow some beginner tutorials to build a game.

Camera types? Orthogonal vs Perspective?

Unity - what is a transform object? Every game object has one. Why?

Unity - player navigation, could use either Unity's navigation system or NavMesh (which is available from the Unity GitHub). NavMesh is more flexible and easier to use. Why?

Unity C# script lifecycle. When do `Awake()`, `Start()` and `Update()` get called? What is their purpose?

Game design document - what goes into it? Set up a template document that I can use for my future ideas (base it off of the one in Unity Fundamentals). Should be a living document, but keep ideas/details as small as possible.

### Prototyping

What aspects need to be prototyped?
- Mouse / keyboard interactions with world, items, NPC's, doors, etc?
- Player controller: movement (animations, transforms), health, damage, etc.
- NPC controller: movement, animation, patrolling, sight, etc.

Primitives are great for prototyping. A flattened cube can have its x and y axis extended to represent a floor. A capsule with a child cylinder to represent facing forward can represent a player. A cube could represent a treasure chest, and so forth.

To move around, use a MouseManager (class I define). Example shows lots of `if/else`, try and do it using polymorphic dispatch? Use layers and tags. Anything that should be clickable, put it in a `clickable` layer! items shout be tagged with their type.

When setting up navigation with NavMesh Surface, the player objects are often included in the NavMesh. Avoid this by assigning the Player objects to a layer named Player and exclude the layer from the the NavMesh Surface. The door object will also cut a hole in the NavMesh Surface, like the player. Because the door is an "obstacle", we can add a NavMesh Obstacle object as a component of the door. We also add a NavMesh Modifier component to the door and set it to `ignore from build`. The NavMesh Obstacle component should be set to `carve`. The carve setting basically carves out an area in the NavMesh Surface through which the player cannot navigate - this will be placed around the door. Finally, add a NavMesh Agent as a component of the Capsule (player) and leave it set to defaults.

Find out more about how NavMesh works and what the various options for the each of the components does.

### Camera

How do I set the distance, position, viewing angle, following, etc, that is used in Play mode?

### Assets

What is an .fbx file? An Autodesk file.
What is an .tga file? Raster graphics file.
What is an .mat file? Autodesk 3ds Max file.

### Software

Must provide Unity compatible formats!

FOSS alternative to Autodesk.
Raster graphics (TGA).
FOSS alternative to 3ds Max.

### Shaders

Learn about the different types of Unity shaders (easily found by going to the inspector with a material selected). How do different types of shader work?

How do textures relate to materials? What is a map (albedo, specular, normal, (ambient) occlusion, etc)? What is a secondary map? What is emmissive colour (constrains underlying colour to just the white area of the map)?

Texture is not marked as a normal map, do you want to fix it now? What does this mean?

### Animations

In Unity, Animator Controller component controlls the logic for animating game objects.

When talking about 3D models and animations, we use the term rig. What is a rig?

What are animation principles?
- Follow through?

### Rendering

It is recommended to choose colour space early on in the project. Options are either `linear` or `gamma`. What does this mean?

## Glossary

Albedo = colour without light (pur colour). May sometimes see it called base colour or diffuse.