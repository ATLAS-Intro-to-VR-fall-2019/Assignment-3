# Assignment-3: VR Controllers
In this assigment we will walk through how to get input from a connected bluetooth controller. It will take some detective work on our part to get everything going. First we need to set up a console window on the phone so we can use the console. Before we get into that though, make sure you set up a new scene or project to be ready to deploy to the phone, but disable VR for now, we will get back to it later.

## In App Debug Logs
One of the challenges of doing mobile development in Unity is that you can't see the console from your phone. In class we went over how to create a text element that can capture the logs when the app is running, this section is a recap of that process.

First step is to create a text element and attach it to the main camera so that we know it will always be in view. In the Hierarchy right click on the *Main Camera* and select `3D Object -> 3D Text`, this will create a 2D text element that we can move around in 3D space. What you might notice is that it's pretty gross looking, the text is way to big and very blurry. Let's fix that so we can actually read what's going on when we log stuff.

Let's start with the position, you won't be able to see the text unless it's actually in front of the camera, to do this give the transform a positive z value. Now let's address the blur. The issue here is that our font size is actually getting scaled up from a tiny number. In the *Text Mesh* component, change the font size to be 100, then the character size to be 0.1 and in the transform set the scale to be `0.03, 0.03, 0.03`. The text should look nice and crisp.

### Capturing Debug Logs
Create a new script named *LogMonitor* and attach it to the text object. Edit that script and put the following code in the body of the class.
```
// Will be used to store a reference to the text mesh component
private TextMesh textMesh;

// Use this for initialization
void Start () {
    // Get the instance of the TextMesh component on this game object and store it
    textMesh = gameObject.GetComponentInChildren<TextMesh>();
}

// Called by Unity when this object is enabled in the scene
void OnEnable() {
    // Attach the LogMessage function as a callback for the logMessageReceived event in Unity
    Application.logMessageReceived += LogMessage;
}

// Called by Unity when this object is disabled in the scene
void OnDisable() {
    Application.logMessageReceived -= LogMessage;
}

public void LogMessage(string message, string stackTrace, LogType type) {
    // Set the text 
    textMesh.text = message;
}
```
Now whenever you call `Debug.Log()` you will see the message appear as text in this text object.

## Controller Detective Work
The biggest challenge we face to getting our controllers working is poor documentation. In the case of the controller suggested for this class we will need to probe Unity's input system to get a correct mapping of the buttons. Make sure your controller is connected to the phone and that Unity is set up to deploy to your phone. Next, in any script (I used the LogMonitor), create an Update function that we can log messages from.

### Unity's Input System
Unity handles input through a global input manager that allows us to get the state of pre-defined input fields. Previously we used this to get the state of the mouse button with `Input.GetMouseButtonDown(0)`, which returned true if the left mouse button was pressed only on the frame it was pressed. This tool also allows us to get float values, each are referred to as an *Axis* and given a name by the editor. We can determine the what Axes are available to us by opening the project settings `Edit -> Project Settings -> Input`. Click on the `Fire1` axis and note it's properties, this is where the Input is connected into the Unity input system and given an alias. You can have multiple input's map to the same axis, which is what we will be taking advantage of today.

### Finding Your Controller
If your controller is already connected to your phone we should be able to get it's values in Unity. First if there is a "game" setting on the controller make sure that mode is set, on my controller it's a little switch on the left side. Now let's probe around for some values. In the Update function add the following code
```
Debug.Log(Input.GetAxis("Horizontal"));
```
This will take whatever value the Horizontal axis has at the current frame and print it to the logs. Now upload to your phone and see if it works when you move the joystick. When I move mine, I get a value between -0.8 and 1. Try some other axis names and make note of how they behave. If they only return values 0 and 1 then they are a button, otherwise they are an axis that returns float values. Some important ones you should look for and find on your controller are: Horizontal, Vertical, Fire1, and Jump.


## Joystick Movement and Look Teleportation
At this point let's make VR enabled again by editing the player settings like you did for assignment 2. Also add a couple of elements to the scene to use as landmarks so you know you are moving, and a Plane object named floor that creates a ground plane along the bottom of the scene.

### Joystick
Your controller should have a joystick or a d-pad we can use for movement. Create a new script to capture the joystick input named `CameraMovement` and attach it to *MainCamera*. We will be using some of the Axis names we discovered before ( "Horizontal" and "Vertical") to add and subtract from the camera's x and z position based on the state of the joystick. We will also need to use relative vectors for movement instead of just adding and subtracting from the x and z position of the camera. Luckily, Unity has us coverd with  `transform.right` and `transform.forward` to get a vectors that point in the directions we want.

In VR you will encounter an issue where you cannot directly set the transform of the camera, this issue can be avoided by a little trick with object nesting. Create an empty game object in the Hierarchy and name it *CameraParent* and drag your *MainCamera* on to it. Then in the `CameraMovement` script add a new class variable `public GameObject parent;` and set it in the editor by dragging the CameraParent from the Hierarchy onto that field in the inspector when it appears. Now try this as your joystick movement code for left and right.
```
// When we have a horizontal value
if (Input.GetAxis("Horizontal") != 0) {
    // Move the attached parent based on the right vector of this object multiplied by the horizontal axis value
    parent.transform.position += (transform.right) * Input.GetAxis("Horizontal");
}
```
This may move a little quick so try scaling it down by multiplying the axis value by 0.5 or whatever feels right to you. Once you have it moving nicely, do the same thing for moving forward and backward using `transform.forward` and the "Vertical" axis.

### Teleport
Most controllers have a trigger, let's map that to a teleport function. First, take the plane we had before and create a layer for it called *Environment* in the 8th layer slot, this is similar to how we had a targets layer in assignment two. Modify the camera movement script to use same raycasting code we used in assignment 2 to delete targets, then remove the line that destroies the object hit by the raycast. Replace `Input.GetMouseButtonDown(0)` in the if statement with `Input.GetButtonDown("Fire1")`, "fire1" was the name of the input for my trigger, use whichever Axis mapped to yours in your code.

Then where the destroy function used to be add the following.
```
parent.transform.position = new Vector3(hit.point.x, 0, hit.point.z);
```
This takes the current transform of the the camera and set's it's position to have the raycast hit point's x and z while maintaining the same y value when you click the trigger while looking at some spot on the ground.

## Assignment Turn In
For full credit you must complete at least one of the bonus goals below. Once complete, demonstrate the functioning app before the deadline to the instructor.

## Bonus
- Change the teleport code so that the player moves when they release the button and add a preview of where the user will teleport. Make the preview something that stands out and is easy to see. **1.5pts**
- Discover another button on the controller and create a that allows you to grab and move objects in the scene. **1.5pts**
