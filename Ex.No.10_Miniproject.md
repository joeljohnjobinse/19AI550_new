# Ex.No: 10  Implementation of 2D Using A* Algorithm 
### DATE: 23/5/2026                                                                          
### REGISTER NUMBER: 212223240062

### AIM: 
To develop "Maze Runner" game in Unity integrating A* algorithm and depth first search backtracking algorithm. 

### Algorithm:
1. Create a new 2D Unity project.
2. Design the maze using a script that randomly generates walls and paths each time the game runs.
3. Use a grid layout to place wall and floor tiles (as sprites or prefabs).
4. Place the player, exit, and ghost in specific or random positions on the maze.
5. Write player movement so they can walk only on paths.
6. Implement the A algorithm* so the ghost can calculate the best path to the player.
7. Make the ghost follow the player along that path and update the path as the player moves.
8. Detect collision between ghost and player to trigger “Game Over.”
9. Detect when player reaches exit to trigger “You Win.”
10. Repeat: Each time you run the game, the maze is new, and the chase begins again.

### Program:
Player Movement
```
using System.Collections;
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    public float moveSpeed = 5f;  // Speed at which the player moves
    private Vector2Int gridPos;   // Current position in the grid

    private MazeGenerator mazeGen;
    private int[,] maze;
    private Vector3 targetPos;    // Target position to move toward

    private bool isMoving = false; // To track if the player is in the process of moving
    private GameObject exit;      // Reference to the exit object

    void Start()
    {
        mazeGen = FindFirstObjectByType<MazeGenerator>();
        maze = mazeGen.GetMaze();
        gridPos = new Vector2Int(Mathf.RoundToInt(transform.position.x), Mathf.RoundToInt(transform.position.y));
        targetPos = transform.position;

        // Find the exit object
        exit = GameObject.FindGameObjectWithTag("Exit");
    }

    void Update()
    {
        if (!isMoving)
        {
            MovePlayer();

            // Check if player reached the exit
            if (Vector3.Distance(transform.position, exit.transform.position) < 0.5f)
            {
                EndGame("You Win!");
            }
        }
    }

    void MovePlayer()
    {
        gridPos.x = Mathf.RoundToInt(transform.position.x);
        gridPos.y = Mathf.RoundToInt(transform.position.y);

        if (Input.GetKey(KeyCode.W) || Input.GetKey(KeyCode.UpArrow))
            TryMove(Vector2Int.up);
        if (Input.GetKey(KeyCode.S) || Input.GetKey(KeyCode.DownArrow))
            TryMove(Vector2Int.down);
        if (Input.GetKey(KeyCode.A) || Input.GetKey(KeyCode.LeftArrow))
            TryMove(Vector2Int.left);
        if (Input.GetKey(KeyCode.D) || Input.GetKey(KeyCode.RightArrow))
            TryMove(Vector2Int.right);
    }

    void TryMove(Vector2Int direction)
    {
        Vector2Int targetGridPos = gridPos + direction;

        if (InBounds(targetGridPos) && maze[targetGridPos.x, targetGridPos.y] == 0)
        {
            targetPos = new Vector3(targetGridPos.x, targetGridPos.y, 0);
            StartCoroutine(SmoothMove());
        }
    }

    IEnumerator SmoothMove()
    {
        isMoving = true;

        float timeElapsed = 0f;
        Vector3 startingPos = transform.position;

        while (timeElapsed < 1f)
        {
            transform.position = Vector3.Lerp(startingPos, targetPos, timeElapsed);
            timeElapsed += Time.deltaTime * moveSpeed;
            yield return null;
        }

        transform.position = targetPos;
        gridPos = new Vector2Int(Mathf.RoundToInt(transform.position.x), Mathf.RoundToInt(transform.position.y));

        isMoving = false;
    }

    bool InBounds(Vector2Int targetPos)
    {
        return targetPos.x >= 0 && targetPos.y >= 0 && targetPos.x < maze.GetLength(0) && targetPos.y < maze.GetLength(1);
    }

    void EndGame(string message)
    {
        Debug.Log(message);
        // You can add UI to display "You Win!" or "Game Over" here
        Time.timeScale = 0;  // Stop the game
    }
}
```

Ghost AI
```
using System.Collections;
using UnityEngine;

public class GhostAI : MonoBehaviour
{
    public float moveSpeed = 2f;
    private Transform player;
    private MazeGenerator mazeGen;
    private int[,] maze;
    private Vector2Int ghostPos;

    void Start()
    {
        StartCoroutine(Initialize());
    }

    IEnumerator Initialize()
    {
        // Wait until MazeGenerator is available (using the newer method)
        yield return new WaitUntil(() => FindFirstObjectByType<MazeGenerator>() != null);
        mazeGen = FindFirstObjectByType<MazeGenerator>();

        // Wait until the maze is generated
        yield return new WaitUntil(() => mazeGen.GetMaze() != null);
        maze = mazeGen.GetMaze();

        // Find the Player
        player = GameObject.FindGameObjectWithTag("Player").transform;

        // Start chasing the player
        StartCoroutine(FollowPlayer());
    }

    IEnumerator FollowPlayer()
    {
        while (true)
        {
            // Get the positions of ghost and player
            Vector2Int start = new Vector2Int(Mathf.RoundToInt(transform.position.x), Mathf.RoundToInt(transform.position.y));
            Vector2Int end = new Vector2Int(Mathf.RoundToInt(player.position.x), Mathf.RoundToInt(player.position.y));

            // Find path to the player using A* algorithm
            var path = Pathfinding.FindPath(maze, start, end);

            if (path != null && path.Count > 1)
            {
                // Move the ghost towards the next position in the path
                Vector3 nextPos = new Vector3(path[1].x, path[1].y, 0);
                float t = 0;
                Vector3 initial = transform.position;

                while (t < 1f)
                {
                    t += Time.deltaTime * moveSpeed;
                    transform.position = Vector3.Lerp(initial, nextPos, t);
                    yield return null;
                }
            }
            else
            {
                // If no path is found, wait a bit and try again
                yield return new WaitForSeconds(0.5f);
            }

            // Check if the ghost has reached the player
            if (Vector3.Distance(transform.position, player.position) < 0.5f)
            {
                EndGame("Game Over! Ghost caught you!");
            }
        }
    }

    void EndGame(string message)
    {
        Debug.Log(message);
        // You can add UI to display "You Win!" or "Game Over" here
        Time.timeScale = 0;  // Stop the game
    }
}
```
PathNode
```
public class PathNode
{
    public int x, y;
    public int gCost, hCost;
    public PathNode parent;

    public int FCost => gCost + hCost;

    public PathNode(int x, int y)
    {
        this.x = x;
        this.y = y;
    }

    public override bool Equals(object obj)
    {
        if (obj is PathNode other)
            return x == other.x && y == other.y;
        return false;
    }

    public override int GetHashCode() => x * 1000 + y;
}
```
Pathfinding
```
using System.Collections.Generic;
using UnityEngine;

public class Pathfinding : MonoBehaviour
{
    public static List<Vector2Int> FindPath(int[,] maze, Vector2Int start, Vector2Int end)
    {
        var open = new List<PathNode>();
        var closed = new HashSet<PathNode>();

        PathNode startNode = new PathNode(start.x, start.y);
        PathNode endNode = new PathNode(end.x, end.y);

        open.Add(startNode);

        while (open.Count > 0)
        {
            open.Sort((a, b) => a.FCost.CompareTo(b.FCost));
            PathNode current = open[0];
            open.RemoveAt(0);
            closed.Add(current);

            if (current.x == endNode.x && current.y == endNode.y)
                return RetracePath(current);

            foreach (var dir in Directions())
            {
                int nx = current.x + dir.x;
                int ny = current.y + dir.y;

                if (!InBounds(maze, nx, ny) || maze[nx, ny] == 1 || closed.Contains(new PathNode(nx, ny)))
                    continue;

                PathNode neighbor = new PathNode(nx, ny)
                {
                    gCost = current.gCost + 1,
                    hCost = Mathf.Abs(nx - end.x) + Mathf.Abs(ny - end.y),
                    parent = current
                };

                if (!open.Exists(n => n.Equals(neighbor) && n.FCost <= neighbor.FCost))
                    open.Add(neighbor);
            }
        }

        return null;
    }

    static List<Vector2Int> RetracePath(PathNode endNode)
    {
        var path = new List<Vector2Int>();
        var current = endNode;
        while (current != null)
        {
            path.Add(new Vector2Int(current.x, current.y));
            current = current.parent;
        }
        path.Reverse();
        return path;
    }

    static bool InBounds(int[,] maze, int x, int y) =>
        x >= 0 && y >= 0 && x < maze.GetLength(0) && y < maze.GetLength(1);

    static List<Vector2Int> Directions() => new()
    {
        Vector2Int.up, Vector2Int.down, Vector2Int.left, Vector2Int.right
    };
}
```
Maze Generator
```
using System.Collections.Generic;
using UnityEngine;

public class MazeGenerator : MonoBehaviour
{
    public int[,] GetMaze() => maze;
    public int width = 21;  // Must be odd
    public int height = 21; // Must be odd

    public GameObject wallPrefab;
    public GameObject floorPrefab;
    public GameObject playerPrefab;
    public GameObject ghostPrefab;
    public GameObject exitPrefab;

    private int[,] maze;

    void Start()
    {
        GenerateMaze();
        DrawMaze();
        SpawnCharacters();
    }

    void GenerateMaze()
    {
        maze = new int[width, height];

        // Initialize the outer boundary as walls (1)
        for (int x = 0; x < width; x++)
            for (int y = 0; y < height; y++)
                maze[x, y] = 1;

        // Start DFS from (1, 1) to carve out paths inside the maze
        CarveMaze(1, 1);
    }

    void CarveMaze(int x, int y)
    {
        maze[x, y] = 0;  // Carve a path

        var directions = new List<Vector2Int>
        {
            new Vector2Int(0, 2),  // Move down by 2
            new Vector2Int(0, -2), // Move up by 2
            new Vector2Int(2, 0),  // Move right by 2
            new Vector2Int(-2, 0)  // Move left by 2
        };

        Shuffle(directions);

        foreach (var dir in directions)
        {
            int nx = x + dir.x;
            int ny = y + dir.y;

            if (IsInBounds(nx, ny) && maze[nx, ny] == 1)
            {
                maze[x + dir.x / 2, y + dir.y / 2] = 0;  // Carve the wall between the current and next tile
                CarveMaze(nx, ny);
            }
        }
    }

    void DrawMaze()
    {
        for (int x = 0; x < width; x++)
        {
            for (int y = 0; y < height; y++)
            {
                GameObject prefab = maze[x, y] == 1 ? wallPrefab : floorPrefab;
                Instantiate(prefab, new Vector3(x, y, 0), Quaternion.identity, transform);
            }
        }
    }

    void SpawnCharacters()
    {
        // Spawn Player at a random open spot (Z = 1, above the floor)
        Vector3 playerPos = GetRandomOpenPosition();
        Instantiate(playerPrefab, playerPos, Quaternion.identity);
        Debug.Log($"Player spawned at {playerPos}");

        // Spawn Exit at a random open spot (Z = 0, on the ground level)
        Vector3 exitPos = GetRandomOpenPosition();
        exitPos.z = 0; // Ensure Exit is placed on the ground (Z = 0)
        Instantiate(exitPrefab, exitPos, Quaternion.identity);
        Debug.Log($"Exit spawned at {exitPos}");

        // Spawn Ghost at a random open spot near the exit (Z = 1, above the floor)
        Vector3 ghostPos = GetRandomOpenPosition();
        Instantiate(ghostPrefab, ghostPos, Quaternion.identity);
        Debug.Log($"Ghost spawned at {ghostPos}");
    }

    Vector3 GetRandomOpenPosition()
    {
        Vector3 position = Vector3.zero;
        do
        {
            int x = Random.Range(1, width - 1);
            int y = Random.Range(1, height - 1);

            if (maze[x, y] == 0) // Only place the character on an open space (floor)
            {
                position = new Vector3(x, y, 1); // Z = 1 for visibility above the floor
                break;
            }
        } while (true);

        return position;
    }

    bool IsInBounds(int x, int y)
    {
        return x > 0 && x < width - 1 && y > 0 && y < height - 1;
    }

    void Shuffle(List<Vector2Int> list)
    {
        for (int i = 0; i < list.Count; i++)
        {
            int rnd = Random.Range(i, list.Count);
            var temp = list[i];
            list[i] = list[rnd];
            list[rnd] = temp;
        }
    }

    public Vector2Int GetStartPosition() => new Vector2Int(1, 1);
    public Vector2Int GetEndPosition() => new Vector2Int(width - 2, height - 2);
}
```
### Output:
<img width="1069" height="465" alt="image" src="https://github.com/user-attachments/assets/3e74d357-0c3f-42b3-9709-31790b8dc09d" />


### Result:
Thus, the game Maze Runner was developed using Unity and adopted A* algorithm.
