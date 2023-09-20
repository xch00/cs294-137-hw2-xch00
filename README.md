# HW2: Building Unity Game in Simulated AR

In this homework you will be building a simple AR Game. The exact game you make is up to you and can be very simple (think Board games such as Tic-Tac-Toe). This document will help you set up an unity game in a simulate environment. It will guide you through the steps of: finding planes in a scene, selecting your game board location, and creating basic interaction elements. 

It will be up to you to use what you have learned to make a game. Your game must have an end condition and indicate to the user the result of the game (ie. whether they have won or lost, or what their final score is). 
---
HW2[https://youtu.be/anLk0ITxypI]
## Logistics

Please fork this assignment into a private repo. Rename the repo to cs294-137-hw2-YourGitID 

Note, this stack overflow post covers on how to fork into a private repo: https://stackoverflow.com/questions/10065526/github-how-to-make-a-fork-of-public-repository-private

### Deadline

HW2 is due Friday 09/16/2022, 11:59PM Pacific. Both your code and video need to be turned in for your submission to be complete; HWs which are turned in after 11:59pm will use one of your slip days -- there are no slip minutes or slip hours. The root folder of the repositoy should contain a text file with the link to the Video (can be hosted on Youtube or other video sharing sites). 

### Academic honesty
Please do not post code to a public GitHub repository, even after the class is finished, since these HWs will be reused both  in the future.

This HW is to be completed individually. You are welcome to discuss the various parts of the HWs with your classmates, but you must implement the HWs yourself -- you should never look at anyone else's code.

## Deliverables: 

### 1. Video
You will make a 2 minute video showing off the features of your game. The video must include a verbal description of your project and it’s features. You must also include captions corresponding to the audio. This will be an important component of all your homework assignments and your final project so it is best you get this set up early. 


### 2. Code
You will also need to push your project folder to your your private repo. 
Add the following github IDs so that we can access these:

bjo3rn 
jamesdsmith

**Submit a link to your repo and your video on bCourses.** Do not modify your repo after the submission deadline.

## Setting Up Your Project:
Note the assets used in this assignmente was tested on Unity 2020.3.17f1. It may work in other versions too.

Download the SimEnvironments.unitypackage from Google Drive : https://drive.google.com/file/d/13Xf_rKldxM0mgLmgZgbKe2Dg_VzIJBgL/view?usp=sharing 

In Unity Hub create a new 3D Project. Import the ARFoundationSim.unitypackage and SimEnvironments.unitypackage using Assets->Import 
![i1.JPG](/Instructions/i1.JPG)

First delete the main camera that is placed in the scene by default by unity. We will replace this by it’s AR Equivalent. Search for the "AR Camera" Prefab and drag it into the scene hierarchy.


Next, we we load a sample simulation scene. Search for "Miniworld_FloorPlan229_physics" prefab and load this into the scene. This is the virtual environment in which you will spawn your AR game. This contains the scene, as well as any associated AR planes in it.
![i3.JPG](/Instructions/i3.JPG)


We see that, the AR camera is seeing the wrong parts of the scene. Set the transform values through the inspector, that will make sure AR camers gets a good view of the scene. For instance, here's a set of values you may use:
![i2.JPG](/Instructions/i2.JPG)


Now to help interface with all the AR Planes and their associated interations, we will add our ARPlaneManager, and ARRaycastManager into the scene. To do this, create a new game Object in the scene hierarchy and call it "Managers". In the inspection tab select Add Component and in the search box type “AR Plane Manager” and add it. You will notice that the plane prefab field is empty. We will fill this field by creating our own plane prefab. 

![i4.JPG](/Instructions/i4.JPG)

Now, save the scene. Click Play. In the scene hierarchy, select "Managers" and Change the Value of "Detection Mode". You should observe that different types of planes are detected in the AR scene.





## Placing Your Game Board:

Go to GameObject -> 3D Object -> Cube and name your new cube, “Game Board”. Lets change this into more of a game board shape. Select the Game Board and in the inspector set the scale x,y and z values to 0.6, 0.02, and 0.6 respectively. Unity is set up such that the values of 1 unit in game coordinates corresponds to 1 meter in physical coordinates. Since the cube model is a 1 unit cube, these scale parameters correspond to a game board that is 60cm width and height with a 2cm thickness. 

For now, we will also deactivate our game board so that will not be present at the start of our game. To do this, deselect the checkbox at the very top of the inspector tab for your game board (right above the word “tag”).

![image14.png](/Instructions/image14.png)

Now we need to set up the code that allows the user to choose a position for the game board. Since we don’t know what the area will look like in advance, we will let the user choose where they want to place the game board. To the Managers, we will add the component ARRaycastManager. Raycasting is how we convert a 2D position on the screen to a 3D position in world space. The ARRaycastManager lets us raycast to the planar regions detected by the AR Plane Manager we created in the previous step. 

Next we will add the script to actually place the game board. To your Managers select Add Component -> New Script and name it PlaceGameBoard. Double click on the script (in the box next to the word script, not the component itself) to open it. 

Set up your script as shown below. For all code is this document, be sure to read the code and comments to understand what the code is doing. 
```C++

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
//This allows us to user the AR Foundation simulator functions
using cs294_137.hw2;
public class PlaceGameBoard : MonoBehaviour
{
    // Public variables can be set from the unity UI.
    // We will set this to our Game Board object.
    public GameObject gameBoard;
    // These will store references to our other components.
    private ARRaycastManager raycastManager;
    private ARPlaneManager planeManager;
    // This will indicate whether the game board is set.
    private bool placed = false;

    // Start is called before the first frame update.
    void Start()
    {
        // GetComponent allows us to reference other parts of this game object.
        raycastManager = GetComponent<ARRaycastManager>();
        planeManager = GetComponent<ARPlaneManager>();

        //We want to place our board only on hortizontal planes. So we tell the plane manager only to detect those
        planeManager.detectionMode = PlaneDetectionMode.Horizontal;
    }

    // Update is called once per frame.
    void Update()
    {
        if (!placed)
        {
            if (Input.GetMouseButtonDown(0))
            {
                Vector2 touchPosition = Input.mousePosition;

                // Raycast will return a list of all planes intersected by the
                // ray as well as the intersection point.
                List<ARRaycastHit> hits = new List<ARRaycastHit>();
                if (raycastManager.Raycast(
                    touchPosition, ref hits, TrackableType.PlaneWithinPolygon))
                {
                    // The list is sorted by distance so to get the location
                    // of the closest intersection we simply reference hits[0].
                    var hitPosition = hits[0].hitPosition;
                    // Now we will activate our game board and place it at the
                    // chosen location.
                    gameBoard.SetActive(true);
                    gameBoard.transform.position = hitPosition;
                    placed = true;
                    // After we have placed the game board we will disable the
                    // planes in the scene as we no longer need them.
                    
                    planeManager.detectionMode = PlaneDetectionMode.None;

                }
            }
        }
        else
        {
            // The plane manager will set all detected planes to active by 
            // default so we will continue to disable these.
            //planeManager.SetTrackablesActive(false); //For older versions of AR foundation
            planeManager.detectionMode = PlaneDetectionMode.None;
        }
    }

    // If the user places the game board at an undesirable location we 
    // would like to allow the user to move the game board to a new location.
    public void AllowMoveGameBoard()
    {
        placed = false;
        //planeManager.SetTrackablesActive(true);
        planeManager.detectionMode = PlaneDetectionMode.Horizontal;
    }

    // Lastly we will later need to allow other components to check whether the
    // game board has been places so we will add an accessor to this.
    public bool Placed()
    {
        return placed;
    }
}

```
When you return to the unity editor you should see your script component now has a field for “Game Board”. Drag your game board object from your scene hierarchy into this field. 


You will notice we haven’t actually used our function to allow the user to move the game board once it is placed. So let’s implement that. For this we are going to add a button to the screen that calls “AllowMoveGameBoard” when pressed. 

Select GameObject->UI->Canvas to add a Canvas to the scene. The Canvas is where we will place all 2D UI elements that are meant to appear attached to the screen. Now create a button GameObject->UI->Button. It should automatically appear under the canvas in the hierarchy.

![image5.png](/Instructions/image5.png)

First let’s set the button position and size to be more reasonable than the default. Select your button and inside the inspector change the width and height to 160 and 80 respectively. Then click on the icon showing multiple boxes and lines, above the word “Anchors”. This allows you to choose the general position of your button on the screen. Select bottom+center.  Now change the X and Y values of the Pivot field to 0.5 and 0 respectively. Lastly zero out the transform position, as these will have changed as you were modifying the other values to try and keep the button at its original position.

![image11.png](/Instructions/image11.png)

If you build and run now you should see a button present at the bottom of your screen, but clicking it doesn’t actually do anything. 

To change this, in your button inspector, scroll down to the field labeled “On Click”. Press the + selection at the bottom of this field. Where the word “none” appears now, drag your Managers from your scene hierarchy into this location. Lastly change the “No Function” selection to PlaceGameBoard->AllowMoveGameBoard(). If you build and run now, after placing your game board, pressing this button should bring back the place visualization and allow you to move your game board to a new location. 

![i6.PNG](/Instructions/i6.PNG)
Lastly let’s change the button label to something more intuitive than “Button”. Expand your Button object in the scene hierarchy and select the Text object that appears below it. Change the text field under “Text (Script)” in the inspector to “Move Board”. Build and Run to see your changes. 

### Making A Simple Interactable Object:

Making interactable objects in AR is fairly easy. The short version is, we just have to check if a raycast from a user’s touch intersects with an interactable object, and call a function from that object. 

So let’s make a couple objects to interact with. Create two new cubes in the scene, rescale them to be 10cm x 10cm x 10cm, and rename them “AR Button 1” and “AR Button 2”. Drag them to be part of the Game Board in your hierarchy. Reactivate your game board so you can see it, and then reposition your buttons to be at two separate locations on your game board. Don’t forget to deactivate your game board again once you do this. 

![image6.png](/Instructions/image6.png)

To make it easy for us to call different functions for each button we will create an abstract class which each button will inherit from. In the project window at the bottom of the screen, Right Click->Create->C# Script and name it OnTouch3D. All you need to place in this script is:

```C++
public interface OnTouch3D
{
    void OnTouch();
}  
```

And then our AR Button 1 will inherit from this script and implement this function. For a simple interaction we will make the object move up by 10cm when pressed. In AR Button 1, Add Component -> New Script and name it “ARButton1”. Open this script and place the following code:

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// Adding OnTouch3D here forces us to implement the 
// OnTouch function, but also allows us to reference this
// object through the OnTouch3D class.
public class ARButton1 : MonoBehaviour, OnTouch3D
{
    // Debouncing is a term from Electrical Engineering referring to 
    // preventing multiple presses of a button due to the physical switch
    // inside the button "bouncing".
    // In CS we use it to mean any action to prevent repeated input. 
    // Here we will simply wait a specified time before letting the button
    // be pressed again.
    // We set this to a public variable so you can easily adjust this in the
    // Unity UI.
    // While this is not so much useful in simulation where we just do click usign a mouse, it is 
    // essential when we dpeloy our app to a phone, where the input is through touch.
    
    public float debounceTime = 0.3f;
    // Stores a counter for the current remaining wait time.
    private float remainingDebounceTime;

    void Start()
    {
        remainingDebounceTime = 0;
    }

    void Update()
    {
        // Time.deltaTime stores the time since the last update.
        // So all we need to do here is subtract this from the remaining
        // time at each update.
        if (remainingDebounceTime > 0)
            remainingDebounceTime -= Time.deltaTime;
    }

    public void OnTouch()
    {
        // If a touch is found and we are not waiting,
        if (remainingDebounceTime <= 0)
        {
            // Move the object up by 10cm and reset the wait counter.
            this.gameObject.transform.Translate(new Vector3(0, 0.1f, 0));
            remainingDebounceTime = debounceTime;
        }
    }
}
```


Next let’s add a tag to our object to make it easy to tell that this object is interactable. Select your AR Button 1 and at the top of the inspector window, expand the “Tag” dropdown list and select “Add Tag”. Select + and add the tag “Interactable”. This tag will show up on the Tag dropdown list from now on. Select this as the tag for AR Button 1.

![image4.png](/Instructions/image4.png)

Now we need to create the script to actually perform the raycasting to this object. In Managers, Add Component -> New Script and name it “ARButton Manager”. In this script place the following: 

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using cs294_137.hw2;

public class ARButtonManager : MonoBehaviour
{
    private Camera arCamera;
    private PlaceGameBoard placeGameBoard;

    void Start()
    {
        // Here we will grab the AR camera 
        // This camera acts like any other camera in Unity.
        arCamera = FindObjectOfType<ARCamera>().GetComponent<Camera>();
        // We will also need the PlaceGameBoard script to know if
        // the game board exists or not.
        placeGameBoard = GetComponent<PlaceGameBoard>();
    }

    void Update()
    {
        if (placeGameBoard.Placed() && Input.GetMouseButtonDown(0))
        {
            Vector2 touchPosition = Input.mousePosition;
            // Convert the 2d screen point into a ray.
            Ray ray = arCamera.ScreenPointToRay(touchPosition);
            // Check if this hits an object within 100m of the user.
            //RaycastHit hit;
            //if (Physics.Raycast(ray, out hit,100))
            RaycastHit[] hits;
            hits = Physics.RaycastAll(ray, 100.0F);
            for (int i = 0; i < hits.Length; i++)
            {
                // Check that the object is interactable.
                if(hits[i].transform.tag=="Interactable")
                    // Call the OnTouch function.
                    // Note the use of OnTouch3D here lets us
                    // call any class inheriting from OnTouch3D.
                    hits[i].transform.GetComponent<OnTouch3D>().OnTouch();
            }
        }
    }
}
```
Build and run your game. When you place your game board, you should now see that this button moves up when you touch it. 

It is worth noting that that the button does not actually have to be visible for you to interact with this. This is useful if you want the user to be able to interact with empty spaces on the game board, or just want the selectable region to be bigger than the object that is displayed. 

Let’s turn AR Button 2 into an invisible button. First, don’t forget to add the “Interactable” tag since we are now going to set up interaction for this button. Then disable the checkbox next to the “Mesh Renderer” component. This stops the button from being rendered, making it essentially invisible. 

![image12.png](/Instructions/image12.png)

Since moving an invisible object doesn’t make much sense, let’s instead have this object display a message on the screen. Add a new script to AR Button 2 and name it “ARButton2”. Open the script and add the following: 

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ARButton2 : MonoBehaviour, OnTouch3D
{
    public Text messageText;

    public void OnTouch()
    {
        messageText.gameObject.SetActive(true);
        messageText.text = "Button2Pressed";
    }
}
```

We will then need to create the Text object for this to reference. Add a Text object to the scene GameObject->UI->Text and rename it “Message Text”. We will leave this object centered, but be sure the (X,Y,Z) positions are all set to 0. Change the width and height to 250 and 60 respectively and change the font size to 28.  As with our Game Board we are going to set this to inactive to start. 

![image13.png](/Instructions/image13.png)

To make this a proper message, let’s also add a script to make the text go inactive again after a specified time. Add a new script component and name it DisappearingText. Open it and add the following:

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DisappearingText : MonoBehaviour
{
    // displayTime will be set to a public float
    // so that you can easily change it in the Unity UI
    public float displayTime = 1;

    private float timeRemaining;

    // Start is called before the first frame update
    void Start()
    {
        timeRemaining = displayTime;
    }

    // Update is called once per frame
    // but only while the object is active
    void Update()
    {
        timeRemaining -= Time.deltaTime;
        if (timeRemaining < 0)
        {
            timeRemaining = displayTime;
            this.gameObject.SetActive(false);
        }
    }
}
```

Next let’s add a panel to make the text more visible against the background scene. Right Click on your Message Text object in the scene hierarchy and select UI->Panel. This should automatically place a panel under the Message Text object. 

![image5.png](/Instructions/image5.png)

Select the panel and change the “Source Image” in the inspector to UISprite. 

![image1.png](/Instructions/image1.png)

Finally, select your AR Button 2 again and drag your Message Text object into the Message Text section of your AR Button 2 script. 

![image2.png](/Instructions/image2.png)

Build and run your game. After placing your game board you should now be able to click on the location where the button would be (if it were visible) and a message should appear on screen. 

## Finish your game:

The rest is up to you! Using these techniques you need to now design your own game that can run in this simulated environment.
