# Ex.No: 5  Implementation of Steering behaviour-Pursue and Evade in Unity
### DATE:                                                                            
### REGISTER NUMBER : 
### AIM: 
To write a program to simulate the process of Pursue and Evade behavior in Unity using NavigationMeshAgent. 
### Algorithm:
1. Create a New Unity Project by Open the  Unity Hub and create a new 3D Project.
2. Name the project "SteeringBehaviors" and select a location. Click Create.
3.Open Unity Scene (default is SampleScene).
  In the Hierarchy, create a Plane:
  Right-click → 3D Object → Plane (this will be the ground).
  Set its Scale to (10, 1, 10) for a larger surface.
  Create three Capsule for the Player, Pursuer, and Evader:
  Rename them to "Player", "Pursuer", and "Evader".
  Set their Y Position to 0.5 (so they sit on the ground).
  Change their Material for better distinction (optional).
3. Check AI navigation in window.
 Window → AI → Navigation (opens the Navigation tab).  If it is not available then add package by name "com.unity.ai.navigation"
4. Select the Plane, go to the Navigation tab, and mark it as Navigation Static.
   Go to the Bake tab and click Bake.
   or
   Add navMeshSurface to plane and bake 
4. Add NavMeshAgent Component 
    Select Pursuer, and Evader.
    Click Add Component → Search for NavMeshAgent and add it.
    Adjust NavMeshAgent Settings:
    Player: Set Speed = 5.
    Pursuer: Set Speed = 4.
    Evader: Set Speed = 6.
5. Write a script for  Player_movement behavior and save it
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class pursueandevade : MonoBehaviour
{
    public NavMeshAgent pursuerAgent;
    public NavMeshAgent evaderAgent;
    public NavMeshAgent playerAgent;

    public Transform player;
    public Transform pursuer;
    public Transform evader;

    public float pursueSpeed = 4f;
    public float evadeSpeed = 6f;

    void Start()
    {
        pursuerAgent = pursuer.GetComponent<NavMeshAgent>();
        evaderAgent = evader.GetComponent<NavMeshAgent>();
        playerAgent = player.GetComponent<NavMeshAgent>();

        // Player initial movement
        Vector3 randomPos = new Vector3(
            UnityEngine.Random.Range(-4f, 4f),
            0.5f,
            UnityEngine.Random.Range(-4f, 4f)
        );

        playerAgent.SetDestination(randomPos);
    }

    void pursue()
    {
        Vector3 targetvelocity = player.position - pursuer.position;

        Vector3 futurepos =
            pursuer.position +
            targetvelocity.normalized * pursueSpeed;

        pursuerAgent.SetDestination(futurepos);
    }

    void evade()
    {
        Vector3 fleedir = evader.position - player.position;

        Vector3 evadeposition =
            evader.position +
            fleedir.normalized * evadeSpeed;

        evaderAgent.SetDestination(evadeposition);
    }

    void playerMove()
    {
        if (!playerAgent.pathPending &&
            playerAgent.remainingDistance < 0.5f)
        {
            Vector3 newPos = new Vector3(
                UnityEngine.Random.Range(-4f, 4f),
                0.5f,
                UnityEngine.Random.Range(-4f, 4f)
            );

            playerAgent.SetDestination(newPos);
        }
    }

    void Update()
    {
        playerMove();
        pursue();
        evade();
    }
}
```
7. Attach the Script to each player,pursuer and Evader.
   Drag & Drop the Target from the Hierarchy into the "Target" field in the script component ( For pursuer and Evader).
12. Run the game 
13. Stop the program

### Output:


<img width="1918" height="1001" alt="image" src="https://github.com/user-attachments/assets/ce479f7b-7bc6-4cf3-9947-6742fbc1a562" />







### Result:
Thus the simple pursue and evade behavior was implemented successfully.
