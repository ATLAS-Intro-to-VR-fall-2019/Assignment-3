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
The biggest challenge we face to getting our controllers working is poor documentation. In the case of the controller suggested for this class we will need to probe Unity's input system to get a correct mapping of the buttons.

## Joystick Movement and Look Teleportation
At this point let's make VR enabled again by editing the player settings like you did for assignment 2. Also add a couple of elements to the scene to use as landmarks so you know you are moving, and a Plane object named floor that creates a ground plane along the bottom of the scene.


## Bonus
- Add a preview of where the user will teleport. Make it something that stands out and is easy to see.
- 
