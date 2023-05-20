# Unity-Pathfinding-3D
A blogpost-repository about how to do pathfinding in 3D using a sparse voxelisation octree (SVO) with a specialised A* pathfinding algorithm using Unity in C#.

## Introduction
Have you ever wanted had a 3D world where you needed some sort of pathfinding that would NOT just work on the surface of the world but EVERYWHERE? If your answer is a definite YES (or you are just curious anyway) you came to the right place. Whether you want to navigate airplanes, spaceships, birds, or anything with free movement from a start to an endpoint, finding a path in 3D can be challenging, especially if avoiding obstacles is a primary concern. In this blog post's case, we will walk through the logic behind building a fast framework for finding paths in 3D in a constrained environment (a finite 3D space with non-traversable objects in it).

### What this tutorial IS and IS NOT
In this tutorial we will be covering the construction of a sparse voexlisation octree as well as the traversal of it using an optimised A* pathfinding algorithm. The techniques presented will only work for static environments, meaning that dynamic obstacle avoidance must be handled separately which is NOT covered in this tutorial. However, if you know how to do dynamic obstacle avoidance you can use the framework described in this tutorial to handle the static pathfinding and then just build your dynamic obstacle avoidance framework on top of it without a problem. If you want to know about the topic of dynamic obstacle avoidance I recommend reading up on **Optimal Reciprocal Collision Avoidance** (ORCA).

## The problem to solve
The game [Ruins of the Lost]([https://duckduckgo.com](https://portfolio.fh-salzburg.ac.at/projects/2022-ruins-of-the-lost)) wanted to introduce a ranged weapon that could automatically search for its targets when fired without the need of aiming at the targets. As part of the solution, an automatically triggered lightning emitter and a bow shooting enemy-seeking chain lightnings were desired.

## Pathfinding
Usually, pathfinding in 2 dimensions (eg. on the surface of a 3D world) is solved by baking a navmesh and using navmesh-agents, which is a simple task in unity. How would you bake a navmesh in 3D? There is not a surface to follow, as the volume of the free space in the world is the medium to be traversed. So let us introduce a spatial tree structure, as is common for other game engine (sub-)systems like physics for example. Let us define the following requirements for that structure:
* It must be **fast** to traverse the structure.
* It must be memory efficient, as we want to support pathfinding in large environments.
Let us consider a regular 3D-Grid with uniform spacing first. This would be the trivial solution to the problem, as you would just subdivide the world in grid cells with a given size and dimensions and check whether any static (geometric) objects overlap each cell (more on that later). It would be reasonably fast to traverse this structure, as any cell can be accessed in constant O(1) time. However, often times (especially for large, open worlds) a lot of space is unoccupied by any geometry, but a regular grid allocated a cell regardless of any overlap with geometry. This not only leads to excessive memory requirements to have a uniform grid for a large space, it also leads to worse pathfinding runtime because most cells are empty but can not be skipped in traversal. Introducing:

## The **S**parse **V**oxelisation **O**ctree
An octree is a relatively simple spatial data structure that splits a parent space into eight child spaces each time a subdivision is required. **Sparse** in this context means, that only certain space (in our case space that is occupied at least in part) is actually allocated as a child node in the octree. The voxelisation part means, that one voxel (the smallest space unit we define for pathfinding) is the smallest space the octree can subdivide to. This also means for our case, that any occupied voxel inserted into our octree will immediately cause all subdivisions necessary to embed it into the lowest level of the octree. This type of octree is advantageous for our use case, because it satisfies the requirements very well:
* It is reasonably fast to be traversed, as accessing any (voxel-)node  inside the tree has guaranteed O(log n) logarithmic time complexity (even better if subsequent traversal for neighbouring nodes is done from a starting node instead of the root). By nature of having much fewer nodes than a uniform grid has cells, traversal is sped up because fewer nodes cover the whole space; therefore, requiring less traversal time.
* It is very memory efficient, as large unoccupied space is not stored explicity but by the fact that no node is allocated for that space. On top of that, large occupied space can also be reduced into bigger occupied nodes if we remove the constraint of a node always being as small as a voxel.

# Sources
https://www.gdcvault.com/play/1022016/Getting-off-the-NavMesh-Navigating
"Fast Parallel Surface and Solid Voxelisation on GPUs" by Schwarz and Seidel
