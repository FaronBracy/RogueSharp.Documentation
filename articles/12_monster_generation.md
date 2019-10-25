# Monster Generation

During this tutorial we’ll create our first monster, the lowly Kobold. We’ll also randomly create our Kobold and his friends in rooms across our map. At this point the Kobold will not act or even be able to be attacked. That will come later.

![alt text](../images/V3Tutorial/12_kobolds.png "Kobolds rendered in map")

## Choosing a Kobold Color

The most important part of creating our monster is choosing a color for him. I’m going to use an existing color from our swatch `DbBrightWood`. Pick any color you want and add it to `Colors.cs`.

```cs
public static RLColor KoboldColor = Swatch.DbBrightWood;
```

## Create a Monster Base Class

Next we need to create the `Monster` base class that all of our monsters going forward will inherit from. At this point it will be pretty lackluster and not have any functionality but we will be adding to it in the future. Our `Monster` class is a type of `Actor` so it needs to inherit from `Actor`. Create a new class `Monster.cs` in the `Core` folder and add code that looks like the following:

```cs
public class Monster : Actor
{
  // Empty for now, we will add functionality later
}
```

## Creating a Kobold

Our first `Monster` will be the [Reptilian Kobold](https://en.wikipedia.org/wiki/Kobold_(Dungeons_%26_Dragons)). Create a new folder called `Monsters` and create a new class in the `Monsters` folder called `Kobold.cs`. The `Kobold` class will inherit from the `Monster` class that we made up above.

```cs
public class Kobold : Monster
{
  public static Kobold Create( int level )
  {
    int health = Dice.Roll( "2D5" );
    return new Kobold {
      Attack = Dice.Roll( "1D3" ) + level / 3,
      AttackChance = Dice.Roll( "25D3" ),
      Awareness = 10,
      Color = Colors.KoboldColor,
      Defense = Dice.Roll( "1D3" ) + level / 3,
      DefenseChance = Dice.Roll( "10D4" ),
      Gold = Dice.Roll( "5D5" ),
      Health = health,
      MaxHealth = health,
      Name = "Kobold",
      Speed = 14,
      Symbol = 'k'
    };
  }
}
```

For now the `Kobold` will only have a `Create()` method. We’ll pass the current level of the dungeon into the `Create()` method so that as the player progresses deeper into the dungeon the `Kobolds` will get stronger. Feel free to assign the stats to whatever you want. I used the `Dice` class to roll for some random values.

One thing to note is that we didn’t create a `Draw()` method for the `Kobold`, yet he will still be able to be drawn to our map just fine. The reason for this is because of the inheritance chain we set up. `Kobold : Monster : Actor`.  Because an `Actor` already has a `Draw()` method the `Kobold` will get it automatically through inheritance!

## Adding Monsters to the DungeonMap

Now that we have our first monster we need a way to be able to add them to our `DungeonMap`. Open `DungeonMap.cs` and add a new private field that is a `List<Monster>`. Make sure that you initialize the list in the constructor.

```cs
private readonly List<Monster> _monsters;

public DungeonMap()
{
  // Initialize all the lists when we create a new DungeonMap
  _monsters = new List<Monster>();
}
```

Next we’ll need to add a few new methods to `DungeonMap.cs` to help us add monsters.

```cs
public void AddMonster( Monster monster )
{
  _monsters.Add( monster );
  // After adding the monster to the map make sure to make the cell not walkable
  SetIsWalkable( monster.X, monster.Y, false );
}

// Look for a random location in the room that is walkable.
public Point GetRandomWalkableLocationInRoom( Rectangle room )
{
  if ( DoesRoomHaveWalkableSpace( room ) )
  {
    for ( int i = 0; i < 100; i++ )
    {
      int x = Game.Random.Next( 1, room.Width - 2 ) + room.X;
      int y = Game.Random.Next( 1, room.Height - 2 ) + room.Y;
      if ( IsWalkable( x, y ) )
      {
        return new Point( x, y );
      }
    }
  }

  // If we didn't find a walkable location in the room return null
  return null;
}

// Iterate through each Cell in the room and return true if any are walkable
public bool DoesRoomHaveWalkableSpace( Rectangle room )
{
  for ( int x = 1; x <= room.Width - 2; x++ )
  {
    for ( int y = 1; y <= room.Height - 2; y++ )
    {
      if ( IsWalkable( x + room.X, y + room.Y ) )
      {
        return true;
      }
    }
  }
  return false;
}
```

The first method `AddMonster()` will add a `Monster` to list of monsters currently on the map and make sure that the `Cell` the `Monster` is located in will not be walkable.

The next two methods `GetRandomWalkableLocationInRoom()` and `DoesRoomHaveWalkableSpace()` will both be useful for us in determining a good location to place our `Monsters`.

The last change to DungeonMap.cs will be to update the `Draw()` method to make sure that all of the Monsters on the map are also drawn.

```cs
// Iterate through each monster on the map and draw it after drawing the Cells
foreach ( Monster monster in _monsters )
{
  monster.Draw( mapConsole, this );
}
```

## Generating and Placing Monsters

We now have the ability to add `Monsters` to the `DungeonMap`. The only thing left to do is to actually generate our `Monsters`. We’ll update `MapGenerator.cs` and add a method to `PlaceMonsters()`. The algorithm will basically go through each room and roll a 10-sided die. If the result of the roll is 1-6 then we’ll roll 4-sided die and add that many monsters to the room in any open `Cells`. We will make extensive use of the methods that we previously added to `DungeonMap.cs` to help us out.

Add the following method to `MapGenerator.cs`

```cs
private void PlaceMonsters()
{
  foreach ( var room in _map.Rooms )
  {
    // Each room has a 60% chance of having monsters
    if ( Dice.Roll( "1D10" ) < 7 )
    {
      // Generate between 1 and 4 monsters
      var numberOfMonsters = Dice.Roll( "1D4" );
      for ( int i = 0; i < numberOfMonsters; i++ )
      {
        // Find a random walkable location in the room to place the monster
        Point randomRoomLocation = _map.GetRandomWalkableLocationInRoom( room );
        // It's possible that the room doesn't have space to place a monster
        // In that case skip creating the monster
        if ( randomRoomLocation != null )
        {
          // Temporarily hard code this monster to be created at level 1
          var monster = Kobold.Create( 1 );
          monster.X = randomRoomLocation.X;
          monster.Y = randomRoomLocation.Y;
          _map.AddMonster( monster );
        }
      }
    }
  }
}
```

Don’t forget to add a call to `PlaceMonsters()` in the `CreateMap()` method.

```cs
public DungeonMap CreateMap()
{
  // ... Existing code
  PlacePlayer();

  // After the existing PlacePlayer() call, add another call to PlaceMonsters()
  PlaceMonsters();
}
```

If you run the game now you should have some ‘k’ shaped kobolds on your map. They won’t do anything interesting just sit there and act like walls. We will give them some interesting behaviors later on.

As always the code for the tutorial series so far can be found on GitHub:
<https://github.com/FaronBracy/RogueSharpV3Tutorial/tree/10MonsterGeneration>

Bored waiting for the next tutorial? The completed project is already available on Github.
<https://github.com/FaronBracy/RogueSharpRLNetSamples>
