# Introduction-to-VisionOS-and-Spatial-Computing


# Getting Started With VisionOS

When building your app, start with a window and add elements as appropriate to help immerse people in your content. Add a volume to showcase 3D content, or increase the level of immersion using a Full Space. The mixed style configures the space to display passthrough, but you can apply the progressive or full style to increase immersion and minimize distractions.

* Add depth to your windows. Apply depth-based offsets to views to emphasize parts of your window, or to indicate a change in modality. Incorporate 3D objects directly into your view layouts to place them side by side with your 2D views.
* Add hover effects to custom views. Highlight custom elements when someone looks at them using hover effects. Customize the behavior of your hover effects to achieve the look you want.
* Implement menus and toolbars using ornaments. Place frequently used tools and commands on the outside edge of your windows using ornaments.

 


     
## RealityKit

RealityKit plays an important role in visionOS apps, and you use it to manage the creation and animation of 3D objects in your apps. Create RealityKit content programmatically, or use Reality Composer Pro to build entire scenes that contain all the objects, animations, sounds, and visual effects you need. Include those scenes in your windows, volumes, or spaces using a RealityView.

——————

- App icons on VisionOS are special - 3 images overlayed on top of each other, that respond dynamically when viewed (Front, Middle & Back layer)

- Not all APIs that are available on iPadOS / iOS / MacOS are available on VisionOS.  
    - APIs that were deprecated prior to iOS 14
    - API’s that do not work well on visionOS: UIDeviceOrientation, UIScreen, UITabBar, & more..


- Origin (0,0,0) point is below user in a Space (at the users feet)




## **VisionOS Workflow**
- Use SwiftUI and UIKit for building UI
- RealityKit for presenting 3d content, animations, and VFX
- ARKit to understand the space around the user


## **Spaces**
- App can contain single or multiple Spaces
- Only one space can be open at a time

```
// boilerplate VisionOS Application in SwiftUI
@main
struct WorldApp: App {
	var body: some Scene {
		ImmersiveSpace  {
			CustomView()
		}
	}
}
```



## **Model3D**

- Use Model3D to load simple scenes asynchronously 

```
struct CustomView: View {
	var body: some view {
		Model3D(named: “FileName”) { phase in   //file should be in proj
		
		}
	}
}
```


Model3d .overlay { … } // overlay on SwiftUI 3d objects
Model3d .rotation3dEffect( Rotation3D(angle: …., axis: …)) // apply rotation
WindowGroup { … } // SwiftUI window




## **Model 3d vs RealityView**

- RealityView uses ECS (lighting, orbit paths)



## **RealityView Attachments****

- Attach SwiftUI views to an RealityKit Entity 

```
var body: some View {
	RealityView { content, attachments in 
		// Create world content
	
		// ….
	
	} update: { content, attachments in
		
		// …
	} attachments: {
		// Add attachments here
	}
}
```



## **Gestures**

- Since RealityView is a SwiftUI view containing multiple entities, a gesture will target each entity in the view.  Therefore if we want a gesture to only affect a specific object, we must use targetedToEntity
        - .targetedToEntity( …. ) // Allow this gesture on specific entity only
    - Entity must have Collision component and Input Target component to receive gestures (can be added in Reality Composer Pro or programmatically)

```
RealityView {
	// ….
}.gesture(SpacialTapGesture()
	.targetedToEntity( …. )
	.onEnded { value in 
	
	})
```


## **RealityView**

- In RealityKit, objects are SwiftUI views with an Entity
- RealityView allows you to place ‘reality content’ into SwiftUI view hierarchy
    - 2 closures: Make, and update 
        - Make - loads the entity: 
            - If let scene = try? await Entity(named: “Scene”, in: …) { content.add(scene) }
        - Update - is not called every frame, only called when SwiftUI state changes

```
struct Earth: View { 
	@State private var thisEntity: EarthEntity?

	var body: some View {
		RealityView { content in
			// …
			content.add( … )
			self.thisEntity = ( … )
		}
	}
}
```


## **Design / UX**

The VisionOS user experience is basically an intuitive blend of iPadOS and VR.  Most default system UI controls and APIs are available on VisionOS, along with many new features.  Apple has created an all new immersive way to interact and see UI in software applications.

- VisionOS is missing several standard APIs found in iOS & iPad, but most apps are intended to work by default.  Apps will be presented on a 2d screen (Volume).
- System color shade slightly vary across platforms (iOS, VisionOS, WatchOS).  
- Make use of system colors on labels for improved readability. (UIColor.label, UIColor.secondaryLabel)
- .borderStyle = .roundedRect for recessed background 
- Use Materials in UI. Materials adjust to blend into surroundings, materials adjust contrast and color balance based on light conditions and colors behind them
- There is no distinction between dark / light mode on VisionOS.  All builtin controls use materials by default, they adjust to surroundings 




## **Eye Tracking / Hover**
- VisionOS has eye tracking; (we do not receive exact eye position)
- It is important UX to indicate when user is looking at specific UI elements
- Builtin VisionOS UI controls will have hover indicators
- New UIView property: UIHoverStyle: can be highlight or lift; or set to nil
    - self.hoverStyle = …. 



## **Interacting With UI**
- Look and pinch = tap 
- Pinch and move = pan 
- When close enough to screen, reach and touch UI elements 


```
if .userInterfaceIdiom == .reality {
	gesture.numberOfTouchesRequired = 2 
}
```



## **Area Mapping**

VisionOS Builds a 3d model of the user surroundings to enable realistic lighting, shadows, and spatial audio.  Apps get access to this data without needing access to user cameras, to ensure user privacy 




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
