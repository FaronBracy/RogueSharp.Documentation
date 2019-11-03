# RogueSharp #

[![Tutorials: Github.io](https://img.shields.io/badge/tutorials-github.io-blue.svg)](https://faronbracy.github.io/RogueSharp/articles/01_introduction.html)
[![Documentation: Github.io](https://img.shields.io/badge/docs-github.io-blue.svg)](https://faronbracy.github.io/RogueSharp/api/index.html)
[![Posts: Blog](https://img.shields.io/badge/blog_posts-wordpress-blue.svg)](https://roguesharp.wordpress.com/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/FaronBracy/RogueSharp/master/LICENSE.txt)

[![Build status](https://dreamersdesign.visualstudio.com/RogueSharp/_apis/build/status/RogueSharp%20Continuous)](https://dreamersdesign.visualstudio.com/RogueSharp/_build/latest?definitionId=1)
[![Build status](https://ci.appveyor.com/api/projects/status/mx09mla59wsgrkkj?svg=true)](https://ci.appveyor.com/project/FaronBracy/roguesharp-20n28)
[![Nuget](https://buildstats.info/nuget/roguesharp)](https://www.nuget.org/packages/RogueSharp)

[![Test status](https://img.shields.io/azure-devops/tests/dreamersdesign/RogueSharp/1.svg)](https://dreamersdesign.visualstudio.com/RogueSharp/_build/latest?definitionId=1)
[![Code coverage](https://img.shields.io/azure-devops/coverage/dreamersdesign/RogueSharp/1.svg)](https://dreamersdesign.visualstudio.com/RogueSharp/_build/latest?definitionId=1)

[![Build History](https://buildstats.info/appveyor/chart/FaronBracy/roguesharp-20n28)](https://ci.appveyor.com/project/FaronBracy/roguesharp-20n28/history)

[![RogueSharp with SadConsole](../images/Index/sadconsoleport.png)](https://github.com/FaronBracy/RogueSharpSadConsoleSamples)

- [Sample Game using RogueSharp with SadConsole](https://github.com/FaronBracy/RogueSharpSadConsoleSamples)
- [Sample Game using RogueSharp with RLNet](https://github.com/FaronBracy/RogueSharpRLNetSamples)

## What is RogueSharp ##

RogueSharp is a free library written in C# to help [roguelike](http://en.wikipedia.org/wiki/Roguelike "roguelike") developers get a head start on their game. RogueSharp provides many utility functions for dealing with map generation, field-of-view calculations, path finding, random number generation and more.

It is loosely based on the popular [libtcod](https://github.com/libtcod/libtcod "libtcod") or "Doryen Library" though not all features overlap.

## Getting Started ##

1. Visit the [RogueSharp Blog](https://roguesharp.wordpress.com/ "RogueSharp Blog") for tips and tutorials.
2. The quickest way to add RogueSharp to your project is by using the [RogueSharp nuget package](https://www.nuget.org/packages/RogueSharp "RogueSharp nuget package").
3. If building the assembly yourself, the solution file "RogueSharp.sln" contains the main library and unit tests. This should be all you need.
4. Class documentation is located on [Github Pages](https://faronbracy.github.io/RogueSharp/api/index.html "RogueSharp class documentation").

## Features ##

### Pathfinding ###

With or without diagonals

![Pathfinding with Diagonals](../images/Index/diagonalpathfinder.gif)

```cs
/// Constructs a new PathFinder instance for the specified Map
/// that will consider diagonal movement by using the specified diagonalCost
public PathFinder( IMap map, double diagonalCost )

/// Returns a shortest Path containing a list of Cells
/// from a specified source Cell to a destination Cell
public Path ShortestPath( ICell source, ICell destination )
```

### Cell Selection ###

Select rows, columns, circles, squares and diamonds with and without borders.

![Cell Selection](../images/Index/selection.gif)

```cs
/// Get an IEnumerable of Cells in a circle around the center Cell up
/// to the specified radius using Bresenham's midpoint circle algorithm
public IEnumerable<ICell> GetCellsInCircle( int xCenter, int yCenter, int radius )
  
/// Get an IEnumerable of outermost border Cells in a circle around the center
/// Cell up to the specified radius using Bresenham's midpoint circle algorithm
public IEnumerable<ICell> GetBorderCellsInCircle( int xCenter, int yCenter, int radius )
```

### Weighted Goal Maps ###

Set multiple goals weights for desirability. Add obstacles to avoid.

![Weighted Goal Maps](../images/Index/diagonalgoalmap.gif)

```cs
/// Constructs a new instance of a GoalMap for the specified Map
/// that will consider diagonal movements to be valid if allowDiagonalMovement is set to true.
public GoalMap( IMap map, bool allowDiagonalMovement )

/// Add a Goal at the specified location with the specified weight
public void AddGoal( int x, int y, int weight )

/// Add an Obstacle at the specified location. Any paths found must not go through Obstacles
public void AddObstacle( int x, int y )
```

### Field-of-View ###

Efficient field-of-view calulation for specified distance. Option to light walls or not.

![Field of View](../images/Index/circularfieldofview.gif)

```cs
/// Constructs a new FieldOfView objec for the specified Map
public FieldOfView( IMap map )

/// Performs a field-of-view calculation with the specified parameters.
public ReadOnlyCollection<ICell> ComputeFov( int xOrigin, int yOrigin, int radius, bool lightWalls )

/// Performs a field-of-view calculation with the specified parameters
/// and appends it any existing field-of-view calculations.
public ReadOnlyCollection<ICell> AppendFov( int xOrigin, int yOrigin, int radius, bool lightWalls )
```

## Creating a Map ##

Most interactions with RogueSharp is based around the concept of a `Map` which is a rectangular grid of `Cells`.

Each `Cell` in a `Map` has the following properties:

- `IsTransparent`: true if visibility extends through the `Cell`.
- `IsWalkable`: true if a the `Cell` may be traversed by the player
- `IsExplored`: true if the player has ever had line-of-sight to the `Cell`
- `IsInFov`: true if the `Cell` is currently in the player's field-of-view

To instantiate a new `Map` you can use its constructor which takes a width and height and will create a new map of those dimensions with the properties of all `Cells` set to false.

### Simple Map Creation - Usage ###

```cs
IMap boringMapOfSolidStone = new Map( 5, 3 );
Console.WriteLine( boringMapOfSolidStone.ToString() );
```

### Simple Map Creation - Output ###

```txt
#####
#####
#####
```

Notice that the `ToString()` operator is overridden on the `Map` class to provide a simple visual representation of the map. An optional bool parameter can be provided to `ToString()` to indicate if you want to use the field-of-view or not. If the parameter is not given it defaults to false.

The symbols used are as follows:

- `%`: `Cell` is not in field-of-view
- `.`: `Cell` is transparent, walkable, and in field-of-view
- `s`: `Cell` is walkable and in field-of-view (but not transparent)
- `o`: `Cell` is transparent and in field-of-view (but not walkable)
- `#`: `Cell` is in field-of-view (but not transparent or walkable)

A more interesting way to create a map is to use the `Map` class's static method `Create` which takes an `IMapCreationStrategy`. Some simple classes implementing `IMapCreationStrategy` are provided with RogueSharp but this is easily extended by creating your own class that implements the strategy.

### Random Rooms Map Creation - Usage ###

```cs
IMapCreationStrategy<Map> mapCreationStrategy = new RandomRoomsMapCreationStrategy<Map>( 17, 10, 30, 5, 3 )
IMap somewhatInterestingMap = Map.Create( mapCreationStrategy );
Console.WriteLine( somewhatInterestingMap.ToString() );
```

### Random Rooms Map Creation - Output ###

```txt
#################
#################
##...#######...##
##.............##
###.###....#...##
###...##.#####.##
###...##...###..#
####............#
##############..#
#################
```
