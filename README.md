# QTM_Vive_Peform
QTM Integration for Unity in Perform

 Combine Qualisys with HTC VIVE

Run Yu   runyu@vt.edu
## Introduction
This is a tutorial about how to combine the Qualisys motion tracking system with the VIVE VR system. The goal is to let you use the VIVE headset, controllers and trackers just like usual, while also having extra items tracked by the Qualisys system co-exist in the same virtual environment (VE). In this way, you take advantage of the high-quality display of the VIVE system without losing the flexibility of tracking custom-made objects (e.g. you can even track full human bodies with the Qualisys system). We think this kind of setup would be ideal for the CUBE. 
To bring the two systems together, the general idea is to have a physical prop co-tracked by the two systems simultaneously, and we use this prop as a reference to bring the two separate coordinate systems to the same virtual environment in Unity. Section 2 is a quick start guide, which is what I could build and test in the Perform studio during summer 2019. Using this guide, you can quickly create a scenario that you use the VIVE headset to walk in a VE while holding a rigid body tracked by the Qualisys system. The biggest problem of this guide is the lack of accuracy. Bringing the two systems together is a calibration process, and all the steps in Section 2 is based on me eyeballing everything. The result of this inaccuracy is that you may see the rigid body tracked by Qualisys being offset by a certain distance (or angle) in the VE compared to reality. Hence, I’ve also written a Section 3 to explain the ideas behind Section 2 and how one could potentially follow up to improve upon the current procedure. 

## Quick Start Guide

### 2.1 Connect QTM with Unity

1) Calibrate the Qualisys system as usual, remember to make the Y axis pointing up in the world coordinate system. If you are not sure, there is a QTM project called “RunTest” on the computer that’s connected to the QTM system in Perform studio. Directly use that project or follow the settings in that project.

2) Make sure you have this physical prop by your hand (shown in the picture below). This prop combines a QTM rigid body and a VIVE tracker together, and it’s specifically built to synchronize between QTM and VIVE. 

![image 1](photo1.jpg)

3) If you are not directly using the “RunTest” project in QTM, you need to define the rigid body on this physical prop in the QTM software. Make sure you define it in the way shown below (notice how QTM uses a right-handed system). Give the rigid body a name and remember it (you’ll need to put it in Unity later).

![image2](picture2.png)
![image3](picture3.png)


4) Initialize a blank Unity project, download the QTM Unity package from the repo of Qualisys https://github.com/qualisys/Qualisys-Unity-SDK

5) Import that Unity package into your project. This should create a folder called “Qualisys”.

6) Create a small 3D object (e.g. a cube) in the Unity scene, which would represent the QTM rigid body on the physical prop. 

7) Drag the “RTObject.cs” script (in Qualisys/QTM-Unity-Realtime-Streaming/Streaming) onto this gameobject. Put the rigid body’s name you defined earlier in QTM in the field of “Object Name”. This would make this Unity project receive real-time motion tracking data from QTM and use that to move this 3D virtual object in Unity. Also, give the “Rotation Offset” field a -90 at the Y field and a 90 at the Z field, as shown in the picture below (in this picture the 3D virtual object representing the QTM rigid body is a “slab” called “QTMOrig”). 

![image4](picture4.png)

*Note: QTM uses a right-handed system while Unity is left-handed. When using this Unity package from QTM, my experience is that it’s helpful to think that the QTM world coordinate system and the Unity world share the same x axis, as shown below.*

(8) Now test if QTM works with Unity. On the top menu of Unity, press “Window” -> “Qualisys” -> RTClient, which should bring up a QTM panel. Press “Play” in Unity Editor to start the application, and then click “connect” on the QTM panel. The computer that runs this application needs to be in the same LAN environment with the QTM server computer to make this work. If it works correctly, you should be able to move the physical prop around in reality while seeing the 3D virtual object (with the script “RTObject.cs” attached) follow your movement. 
Note: you’ll need to click “connect” every time you run the application when using this Unity package provided by Qualisys. 

2.2 Connect VIVE with Unity

(1) Next we need to connect this Unity project with the VIVE system. Just do the regular things that would make VIVE work with Unity. What I was using was the “VIVE Input Utility” from Unity Asset Store. https://assetstore.unity.com/packages/tools/integration/vive-input-utility-64219. It has a prefab that directly includes the VIVE trackers. Download this package and put a “ViveCameraRig” in the Unity scene and make sure its position is at world origin (0,0,0). Also, don’t forget to enable “Virtual Reality Supported” in the “XR setting” of Unity. 

(2) Test if VIVE works. Press “Play” and see if the HMD and tracking works. 

(3) More specific to our setup is that we need to make sure the VIVE tracker on the physical prop is properly tracked and displayed. Depends on how you calibrated the VIVE system, you may need to rotate the “ViveCameraRig” to align it with the Unity world and with the Qualisys world. Test this by moving the physical prop to see if the tracker and the 3D item representing the QTM rigid body move in the same direction in the Unity scene (note that they are not likely to be co-located since it’s likely that the Qualisys and VIVE systems don’t share the same world origin based on the separate calibration processes). If not, you’ll need to rotate ViveCameraRig accordingly. In my testing in Perform, I had to rotate the ViveCameraRig by -90 degrees, as shown below. 

2.3 Synchronize QTM with VIVE

Create an empty GameObject in the Unity scene. Attach the script “Cali_QTM_VIVE.cs” to it. You should be able to find the script in a folder called “code” in the directory of this tutorial. The rest of this sub-section is illustrated in the picture below.

Drag the object representing the VIVE tracker to the field of “Vive Tracker”

Drag the object representing the QTM rigid body to the field of “Qtm R Body”

Drag the object representing the VIVE system (the parent of all VIVE objects in the scene, the ViveCameraRig if you are using it) to the field of “Vive System”

The value of “Real World Offset Vive Minus QTM” should be determined by the distance between the VIVE tracker and the QTM rigid body on the physical prop, as shown below. You can give it a (0, 0, 0.03) approximately. 



(6) Run the project and at any time press space bar, which should instantly align the QTM system with the VIVE system (you should see your viewpoint “jump” to a new location at the moment you press the space bar).  
3. The Long Story
In this section I’ll explain the idea behind this calibration process, so you could create a better and more accurate one. Again, the basic idea is having a physical object co-tracked by the two systems together, so we can use this object as a reference to synchronize (position and orientation wise) them in a Unity project. 
    Because you calibrate the VIVE and the Qualisys systems separately, they have different origins and orientations relative to the real world. For example, a 2D version of such difference is illustrated in the picture below. This discrepancy between the two coordinate systems needs to be accounted for in the Unity project, so objects tracked by the two systems separately would have the same position and orientation data in the Unity scene when they are placed at the same location and orientation in reality (even though the two systems report different numbers). This is done in Unity by placing the parent of all VIVE objects at a location and orientation relative to the parent of all QTM objects so that the offset between these two parents matches the offset between the two coordinate systems in the real world. In the example shown in the picture below, the parent of the VIVE objects (e.g. the “ViveCameraRig” if you are using the Unity package provided by VIVE) should be placed on the top right of the parent of the QTM objects, with a clockwise rotation. 


Now the question is how can we know this offset? This is where the “co-tracked” physical prop comes in. When you have such an item tracked by both systems, you can calculate this offset using the different numbers reported by each system. Take the same 2D example, let’s say you have the “smiley face” object tracked by both systems (shown in the picture below). QTM is reporting that it is at location (3,2) with an orientation that’s Quaternion.identity, while VIVE is reporting that it is at location (1,1) with an orientation that’s with 30 degrees of counter-clock rotation relative to Quaternion.identity (Note this is looking at the object from the perspective of the VIVE system). From these numbers, you can easily figure out that the VIVE coordinate system is placed at the upper right of Qualisys, with a positional offset of (2, 1), and it’s rotated clockwise by 30 degrees relative to Qualisys. Then you can use these numbers to place the parent of the VIVE objects relative to the parent of the QTM objects in the Unity scene. This is what we mean by “using the co-tracked object as a reference to bring the two coordinate systems together”. This is also the idea behind the entire Section 2. If you check the short code of “Cali_QTM_VIVE.cs” used in Section 2, it does exactly what’s just explained. 

Now the unsolved problem is: how to do this accurately? In reality, we don’t have an item that could be co-tracked by both systems, since VIVE only tracks the several devices built by HTC while Qualisys only tracks rigid bodies that are explicitly built with the reflective balls. This is why I built that physical prop that stick a VIVE tracker with a QTM rigid body together. Doing so introduces a lot of potential errors. For example, the two items (i.e., the VIVE tracker and the QTM rigid body) may not be aligned perfectly, which introduces an offset for orientation between the two. Such orientation errors are particularly bad, since they can turn in to huge discrepancies in translation (Imagine you are using a laser pointer to point to a location on a wall that’s 10 meters away, rotating your hand by 1 degree could result in the red dot moved by 1 meter or more on the wall). The two items also have a positional offset between them, and this is why there is a field called “Real World Offset Vive Minus QTM” (Section 2.3) in the calibration code. Ideally, you should have the accurate measurement of this offset in real world and put it in the code, but I did all these by eyeballing and approximation in Section 2. To make a millimeter-accuracy calibration, all these potential errors need to be accounted for. 
    My hope of writing this is that:
By following Section 2, you can have a rough calibration that brings the two systems together. You can wear a HTC VIVE headset and hold a Qualisys-tracked item in your hand, and you’ll see the virtual representation of that item roughly resides in your hand. This should be enough for a lot of demos that don’t require high accuracy. 
By understanding the ideas explained in Section 3, you can improve upon the process and create a more accurate version of this. 
Worst case scenario, shoot me an e-mail and I’ll try to help :-)
