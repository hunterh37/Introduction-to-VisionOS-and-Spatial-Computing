# Introduction to visionOS (1.0) and Spatial Computing


# Getting Started With visionOS

When building your app, start with a window and add elements as appropriate to help immerse people in your content. Add a volume to showcase 3D content, or increase the level of immersion using a Full Space. The mixed style configures the space to display passthrough, but you can apply the progressive or full style to increase immersion and minimize distractions.

* Add depth to your windows. Apply depth-based offsets to views to emphasize parts of your window, or to indicate a change in modality. Incorporate 3D objects directly into your view layouts to place them side by side with your 2D views.
* Add hover effects to custom views. Highlight custom elements when someone looks at them using hover effects. Customize the behavior of your hover effects to achieve the look you want.
* Implement menus and toolbars using ornaments. Place frequently used tools and commands on the outside edge of your windows using ornaments.

 


     
## RealityKit

RealityKit plays an important role in visionOS apps, and you use it to manage the creation and animation of 3D objects in your apps. Create RealityKit content programmatically, or use Reality Composer Pro to build entire scenes that contain all the objects, animations, sounds, and visual effects you need. Include those scenes in your windows, volumes, or spaces using a RealityView.


## **RealityView**

- In RealityKit, objects are SwiftUI views with an Entity
- RealityView allows you to place ‘reality content’ into SwiftUI view hierarchy
    - 2 closures: Make, and update 
        - Make - loads the entity: 
            - If let scene = try? await Entity(named: “Scene”, in: …) { content.add(scene) }
        - Update - is not called every frame, only called when SwiftUI state changes

——————

- App icons on visionOS are special - 3 images overlayed on top of each other, that respond dynamically when viewed (Front, Middle & Back layer)

- Not all APIs that are available on iPadOS / iOS / MacOS are available on visionOS.  
    - APIs that were deprecated prior to iOS 14
    - API’s that do not work well on visionOS: UIDeviceOrientation, UIScreen, UITabBar, & more..


- Origin (0,0,0) point is below user in a Space (at the users feet)




## **visionOS Workflow**
- Use SwiftUI and UIKit for building UI
- RealityKit for presenting 3d content, animations, and VFX
- ARKit to understand the space around the user


## **Spaces**
- App can contain single or multiple Spaces
- Only one space can be open at a time


## **Model3D**

- Use Model3D to load simple scenes asynchronously 

```
struct CustomView: View {
    var body: some view {
        Model3D(named: “FileName”) { phase in   //file should be in proj
        }
    }
}

Model3d .overlay { … } // overlay on SwiftUI 3d objects
Model3d .rotation3dEffect( Rotation3D(angle: …., axis: …)) // apply rotation

```


## **Gestures**

- Since RealityView is a SwiftUI view containing multiple entities, a gesture will target each entity in the view.  Therefore if we want a gesture to only affect a specific object, we must use targetedToEntity
        - .targetedToEntity( …. ) // Allow this gesture on specific entity only
    - Entity must have Collision component and Input Target component to receive gestures (can be added in Reality Composer Pro or programmatically)

```
RealityView {
    // ….
}
.gesture(SpatialTapGesture()
.targetedToEntity( …. )
.onEnded { value in
        
})

```

```
if .userInterfaceIdiom == .reality {
	gesture.numberOfTouchesRequired = 2 
}
```



----------------------------------------------------------------------------------------------


## **Design / UX**

The visionOS user experience is basically an intuitive blend of iPadOS and VR.  Most default system UI controls and APIs are available on visionOS, along with many new features.  Apple has created an all new immersive way to interact and see UI in software applications.

- visionOS is missing several standard APIs found in iOS & iPad, but most apps are intended to work by default.  Apps will be presented on a 2d screen (Volume).
- System color shade slightly vary across platforms (iOS, visionOS, WatchOS).  
- Make use of system colors on labels for improved readability. (UIColor.label, UIColor.secondaryLabel)
- .borderStyle = .roundedRect for recessed background 
- Use Materials in UI. Materials adjust to blend into surroundings, materials adjust contrast and color balance based on light conditions and colors behind them
- There is no distinction between dark / light mode on visionOS.  All builtin controls use materials by default, they adjust to surroundings 




## **Eye Tracking / Hover**
- visionOS has eye tracking; (we do not receive exact eye position)
- It is important UX to indicate when user is looking at specific UI elements
- Builtin visionOS UI controls will have hover indicators
- New UIView property: UIHoverStyle: can be highlight or lift; or set to nil
    - self.hoverStyle = …. 



## **Interacting With UI**
- Look and pinch = tap 
- Pinch and move = pan 
- When close enough to screen, reach and touch UI elements 



## **Area Mapping**

visionOS Builds a 3d model of the user surroundings to enable realistic lighting, shadows, and spatial audio.  Apps get access to this data without needing access to user cameras, to ensure user privacy 




## **Spatial Audio**

Sophisticated understanding of user surroundings. Spatial audio system engine in visionOS fuses accounting sensing with 3d scene understanding to create a detailed model of sonic characteristics of the space.  Since visionOS has a detailed understanding of the users surroundings, apps can simply direct where they want sounds to come from, and visionOS handles the audio mixing.  




## **RealityView Hierarchy**

- View
    - RealityView
        - Entity: An element of a RealityKit scene to which you attach components that provide appearance and behavior characteristics for the entity.



## **RealityKit Entity Base Class**

RealityKit defines a few concrete subclasses of ``Entity`` that provide commonly used functionality.  Components are added to entities as a representation of a geometry or a behavior that you apply to an entity.

- AnchorEntity
- ModelEntity
  

## Entity Components (protocol)

A representation of a geometry or a behavior that you apply to an entity. You can add at most one component of a given type to an entity. RealityKit has a variety of predefined component types that you can use to add commonly needed characteristics. For example, the ``ModelComponent`` specifies visual appearance with a mesh and materials. The `CollisionComponent`` contains a shape and other information used to decide if one entity collides with another.




## **Anchoring Component**

A description of how virtual content can be anchored to the real world.

- Target: The kinds of real world objects to which an anchor entity can be tethered

    - Alignment
        - Horizontal 
        - Vertical
        - Any
    - Classification:
        - Wall
        - Floor
        - Ceiling
        - Table
        - Seat
        - Any



     

## **Reduce draw calls**

Each model entity in your scene generates one draw call per material for noninstanced geometry. On the other hand, with instanced entities, RealityKit only generates a single draw call for each material used on the original instanced entity. As a result, using instanced entities can reduces the number of draw calls your app makes.
Sharing textures between entities by using texture atlases, which are textures that contain images for multiple materials, also reduces your app’s draw calls, as can combining multiple models into a single entity with shared materials. When combining model entities, be careful not to make the combined entities too large; if the entities are too large, they won’t be culled during frustum culling, when objects that are completely off-camera are removed from the rendering process — resulting in no draw calls at all. If any part of the combined entity is visible on screen, every material on that entity generates a draw call, even materials that are completely off screen.




----------------------------------------------------------------------------------------------



## **Useful visionOS Snippets**


Load and return a a specific entity from RealityComposerPro scene
```
// Load and return a a specific entity from RealityComposerPro scene

@MainActor
func loadFromRealityComposerPro(named entityName: String, fromSceneNamed sceneName: String) async -> Entity? {
    var entity: Entity? = nil
    do {
        let scene = try await Entity.load(named: sceneName, in: …bundle…)
        entity = scene.findEntity(named: entityName)
    } catch {
        print("Error loading \(entityName) from scene \(sceneName): \(error.localizedDescription)")
    }
    return entity
}
```


Start ARKitSession Example
```
// Start ARKitSession Example
let session = ARKitSession()
let planeData = PlaneDetectionProvider(alignments: [.horizontal, .vertical])


Task {
    try await session.run([planeData])
    
    for await update in planeData.anchorUpdates {
        // Skip planes that are windows.
        if update.anchor.classification == .window { continue }
        
        switch update.event {
        case .added, .updated:
            updatePlane(update.anchor)
        case .removed:
            removePlane(update.anchor)
        }
    }
}
```


Custom Shader (Underwater Apple example project, Octopus.swift)
```
// Custom Shader (Underwater Apple example project, Octopus.swift)
 do {
                let surfaceShader = CustomMaterial.SurfaceShader(
                    named: "octopusSurface",
                    in: MetalLibLoader.library
                )
                try octopusModel.modifyMaterials {
                    var mat = try CustomMaterial(from: $0, surfaceShader: surfaceShader)
                    mat.custom.texture = .init(mask)
                    mat.emissiveColor.texture = .init(bc2)
                    return mat
                }
            } catch {
                assertionFailure("Failed to set a custom shader on the octopus \(error)")
            }
```

```
// Generate a mesh for the plane (for occlusion).
var meshResource: MeshResource? = nil
do {
    let contents = MeshResource.Contents(planeGeometry: anchor.geometry)
    meshResource = try MeshResource.generate(from: contents)
} catch {
    print("Failed to create a mesh resource for a plane anchor: \(error).")
    return
}

if let meshResource {
    // Make this plane occlude virtual objects behind it.
    entity.components.set(ModelComponent(mesh: meshResource, materials: [OcclusionMaterial()]))
}

// Generate a collision shape for the plane (for object placement and physics).
var shape: ShapeResource? = nil
do {
    let vertices = anchor.geometry.meshVertices.asSIMD3(ofType: Float.self)
    shape = try await ShapeResource.generateStaticMesh(positions: vertices,
                                                       faceIndices: anchor.geometry.meshFaces.asUInt16Array())
} catch {
    print("Failed to create a static mesh for a plane anchor: \(error).")
    return
}

if let shape {
    var collisionGroup = PlaneAnchor.verticalCollisionGroup
    if anchor.alignment == .horizontal {
        collisionGroup = PlaneAnchor.horizontalCollisionGroup
    }
    
    entity.components.set(CollisionComponent(shapes: [shape], isStatic: true,
                                             filter: CollisionFilter(group: collisionGroup, mask: .all)))
    // The plane needs to be a static physics body so that objects come to rest on the plane.
    let physicsMaterial = PhysicsMaterialResource.generate()
    let physics = PhysicsBodyComponent(shapes: [shape], mass: 0.0, material: physicsMaterial, mode: .static)
    entity.components.set(physics)
}
```

-----------------------------------------------------------------


## **Useful Links / Documentation**

GeometryReader3d
- https://developer.apple.com/documentation/swiftui/geometryreader3d

Checking whether your existing app is compatible with visionOS
- https://developer.apple.com/documentation/visionos/checking-whether-your-app-is-compatible-with-visionos


Bringing your ARKit app to visionOS
- https://developer.apple.com/documentation/visionos/bringing-your-arkit-app-to-visionos


Explaining ECS / Entity Component System
- https://medium.com/macoclock/realitykit-911-entity-component-system-ecs-bfe0520e0e8e

Implementing ECS in RealityKit
- https://developer.apple.com/documentation/realitykit/implementing-systems-for-entities-in-a-scene


Augmented Reality - Apple webpage
- https://developer.apple.com/augmented-reality/


Understanding RealityKit’s modular architecture
- https://developer.apple.com/documentation/visionos/understanding-the-realitykit-modular-architecture


ARKit Scene Reconstruction / Scene Collision
- https://developer.apple.com/documentation/visionos/incorporating-surroundings-in-an-immersive-experience


AR Design / UI UX 

- https://developer.apple.com/design/human-interface-guidelines/augmented-reality


More Apple visionOS Documentation:
- https://developer.apple.com/documentation/visionos/incorporating-real-world-surroundings-in-an-immersive-experience
- https://developer.apple.com/documentation/visionos/tracking-points-in-world-space


-------------- 

## **WWDC 2023 visionOS Videos**

Meet ARKit for spatial computing
- https://developer.apple.com/videos/play/wwdc2023/10082/

Evolve your ARKit app for spatial experiences
- https://developer.apple.com/videos/play/wwdc2023/10091/

Get started with building apps for spatial computing
- https://developer.apple.com/videos/play/wwdc2023/10260/

Optimize app power and performance for spatial computing
- https://developer.apple.com/videos/play/wwdc2023/10100/

Meet UIKit for spatial computing
- https://developer.apple.com/videos/play/wwdc2023/111215/

Meet Core Location for spatial computing
- https://developer.apple.com/videos/play/wwdc2023/10146/

Elevate your windowed app for spatial computing 
- https://developer.apple.com/videos/play/wwdc2023/10110/

Enhance your iPad and iPhone apps for the Shared Space
- https://developer.apple.com/videos/play/wwdc2023/10094/

Create a great spatial playback experience
- https://developer.apple.com/videos/play/wwdc2023/10070/

Build spatial SharePlay experiences
- https://developer.apple.com/videos/play/wwdc2023/10087/

Deliver video content for spatial experiences
- https://developer.apple.com/videos/play/wwdc2023/10071/

Explore rendering for spatial computing
- https://developer.apple.com/videos/play/wwdc2023/10095/

Building Immersive Apps

Develop your first immersive app
- https://developer.apple.com/videos/play/wwdc2023/10203/

Bring your Unity VR app to a fully immersive space
- https://developer.apple.com/videos/play/wwdc2023/10093/

Create immersive Unity apps
- https://developer.apple.com/videos/play/wwdc2023/10088/


----------------------------------------------------------------------------------------------

Apple Sample Projects WWDC (ARKit, RealityKit) 2021 - 2023
- https://developer.apple.com/sample-code/wwdc/2021/
- https://developer.apple.com/sample-code/wwdc/2022/
- https://developer.apple.com/sample-code/wwdc/2023/


## **Apple visionOS Sample Projects 2023-24**
- https://developer.apple.com/documentation/realitykit/construct-an-immersive-environment-for-visionos
- https://developer.apple.com/documentation/realitykit/simulating-particles-in-your-visionos-app
- https://developer.apple.com/documentation/realitykit/simulating-physics-with-collisions-in-your-visionos-app
- https://developer.apple.com/documentation/visionos/happybeam
- https://developer.apple.com/documentation/visionos/incorporating-real-world-surroundings-in-an-immersive-experience
- https://developer.apple.com/documentation/visionos/placing-content-on-detected-planes


Other Sample Resources
- https://github.com/satoshi0212/visionOS_30Days
- https://github.com/hunterh37/VisionOS_AnimatedModelEntityDemo
- https://github.com/hunterh37/VisionOS_BouncyBalls
- https://github.com/hunterh37/VisionOS_SceneReconstructionDemo






## **Current Vision Pro Simulator Limitations**

- ARKitSession
    - PlaneDetectionProvider is not available on sim (Vision OS Beta 1.1 7/27/23)
- SceneReconstructionProvider is not available on sim (Vision OS Beta 1.1 7/27/23)


