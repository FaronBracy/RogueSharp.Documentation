# Player Stats

During this tutorial we’ll add all of the necessary stats that the player and other actors will require to perform combat actions. We will also be rendering the player stats to the console that was previously just a placeholder on the right of the map.

![Drawing player stats](../images/V3Tutorial/11_drawstats.png "A console with player stats rendered in the top right corner")

## Combat Overview

The combat system that I’ve decided upon for the tutorial is going to be a percentage based opposing roll system. First the attacker will roll a number of 100 sided dice equal to their attack value. Each die in the roll will have a percentage chance to be a success. Each success has the potential to inflict 1 point of damage. Thus the maximum amount of damage an actor can do is their Attack value, if they get all successes and their is no defense.

So how about defense? It will work the same way. The defender rolls a number of 100 sided dice equal to their defense value. Each die has a percentage chance to be a success. The number of defender successes is subtracted from the number of attacker successes and if the remaining number is positive then that much damage is applied to the defender.

Whew! Hopefully that’s not too confusing. We’ll go through the stats one-by-one in the next section.

## Defining the Stats

* `Attack` – int – Number of dice to roll when performing an attack. Also represents the maximum amount of damage that can be inflicted in a single attack.

* `AttackChance` – int – Percentage chance that each die rolled is a success. A success for a die means that 1 point of damage was inflicted.

* `Awareness` – int – For the player this determines how far their field-of-view extends. For other actors or monsters this is the distance that they can see and hear. If the player gets within that distance then it is likely the monster will be alerted to their presence.

* `Defense` – int – Number of dice to roll when defending against an attack. Also represents the maximum amount of damage that can blocked or dodged from an incoming attack.

* `DefenseChance` – int – Percentage chance that each die rolled is a success. A success for a die means that 1 point of damage was blocked.

* `Gold` – int – How much money the actor has. Most monsters will drop gold upon death.

* `Health` – int – The current health total of the actor. If this value is 0 or less then the actor is killed.

* `MaxHealth` – int – How much health the actor has when fully healed.

* `Name` – string – Name of the actor

* `Speed` – int – How fast the actor is. This determines the rate at which they perform actions. A lower number is faster. An actor with a speed of 10 will perform twice as many actions in the same time as an actor with a speed of 20.

## Adding Stats to Actors

Before we add the stats directly to our `Actor` class we should first add them to the `IActor` interface. Open `IActor.cs` and update the code to look like the following.

```cs
public interface IActor
{
  int Attack { get; set; }
  int AttackChance { get; set; }
  int Awareness { get; set; }
  int Defense { get; set; }
  int DefenseChance { get; set; }
  int Gold { get; set; }
  int Health { get; set; }
  int MaxHealth { get; set; }
  string Name { get; set; }
  int Speed { get; set; }
}
```

Next open `Actor.cs` and make sure that each of these properties is implemented. Note that we won’t be doing auto-properties this time because later we’ll want to update the `getters` on the each property to account for additional stats from items that we equip. For instance the `Defense` stat might have a base value of 2, but then have an additional +2 bonus from armor that is being worn. So although it is more typing, we’ll implement these properties with their own private backing fields to allow us to add that additional functionality in later.

```cs
public class Actor : IActor, IDrawable
{
  // IActor
  private int _attack;
  private int _attackChance;
  private int _awareness;
  private int _defense;
  private int _defenseChance;
  private int _gold;
  private int _health;
  private int _maxHealth;
  private string _name;
  private int _speed;

  public int Attack
  {
    get { return _attack; }
    set { _attack = value; }
  }

  public int AttackChance
  {
    get { return _attackChance; }
    set { _attackChance = value; }
  }

  public int Awareness
  {
    get { return _awareness; }
    set { _awareness = value; }
  }

  public int Defense
  {
    get { return _defense; }
    set { _defense = value; }
  }

  public int DefenseChance
  {
    get { return _defenseChance; }
    set { _defenseChance = value; }
  }

  public int Gold
  {
    get { return _gold; }
    set { _gold = value; }
  }

  public int Health
  {
    get { return _health; }
    set { _health = value; }
  }

  public int MaxHealth
  {
    get { return _maxHealth; }
    set { _maxHealth = value; }
  }

  public string Name
  {
    get { return _name; }
    set { _name = value; }
  }

  public int Speed
  {
    get { return _speed; }
    set { _speed = value; }
  }
}
```

## Drawing Player Stats

Now that we have the stats out of the way, we want to be able to draw the `Player` stats in the upper right corner of the `statConsole`. Open `Player.cs` and set all of the starting stats for the `Player` in the constructor. Also add the `DrawStats()` method to look like the following:

```cs
public class Player : Actor
{
  public Player()
  {
    Attack = 2;
    AttackChance = 50;
    Awareness = 15;
    Color = Colors.Player;
    Defense = 2;
    DefenseChance = 40;
    Gold = 0;
    Health = 100;
    MaxHealth = 100;
    Name = "Rogue";
    Speed = 10;
    Symbol = '@';
  }

  public void DrawStats( RLConsole statConsole )
  {
    statConsole.Print( 1, 1, $"Name:    {Name}", Colors.Text );
    statConsole.Print( 1, 3, $"Health:  {Health}/{MaxHealth}", Colors.Text );
    statConsole.Print( 1, 5, $"Attack:  {Attack} ({AttackChance}%)", Colors.Text );
    statConsole.Print( 1, 7, $"Defense: {Defense} ({DefenseChance}%)", Colors.Text );
    statConsole.Print( 1, 9, $"Gold:    {Gold}", Colors.Gold );
  }
}
```

Make sure to also add the colors to `Colors.cs`. Pick any colors you like. I used these:

```cspublic static RLColor TextHeading = RLColor.White;
public static RLColor Text = Swatch.DbLight;
public static RLColor Gold = Swatch.DbSun;
```

## Updating the Game

The final thing left to do is to update `Game.cs` to make sure that `Player.DrawStats()` is called. During the last tutorial we also added some code to test out our the message log by printing a line every time the player took a step. I’ll be deleting that code now too.

```cs
// DELETE all of the following lines
private static int _steps = 0;

MessageLog.Add( $"Step # {++_steps}" );

_statConsole.SetBackColor( 0, 0, _statWidth, _statHeight, Swatch.DbOldStone );
_statConsole.Print( 1, 1, "Stats", Colors.TextHeading );
```

Then update `OnRootConsoleRender()` to call `Player.DrawStats()`

```cs
if ( _renderRequired )
{
  // Old code to keep
  DungeonMap.Draw( _mapConsole );
  Player.Draw( _mapConsole, DungeonMap );

  // New code after Player.Draw()
  Player.DrawStats( _statConsole );
}
```

If you run the program now you should get something similar to the screenshot at the top of this post.

## Code on GitHub

As always the code for the tutorial series so far can be found on GitHub:

* <https://github.com/FaronBracy/RogueSharpV3Tutorial/tree/09PlayerStats>

Bored waiting for the next tutorial? The complete tutorial project is already finished and the source code is available on Github:

* Sample Roguelike game using RogueSharp and RLNet console
  * <https://github.com/FaronBracy/RogueSharpRLNetSamples>
* Sample Roguelike game using RogueSharp and SadConsole
  * <https://github.com/FaronBracy/RogueSharpSadConsoleSamples>
