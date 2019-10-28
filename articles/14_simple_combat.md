# Simple Combat

During this tutorial we will setup a basic system for combat. At this point only the Player will be able to attack and they will be limited to a single type of attack when trying to move into a space that is occupied by a monster. Dice will be rolled and all the results will be printed to the message log. This is often referred to as “bump attack” because the player bumps into a monster while moving.

![Combat demonstration](../images/V3Tutorial/14_simplecombat.png "Image demonstrating combat")

## Combat Overview

The combat system was covered in the past but I’m going to repeat some of that here for reference. For this tutorial I decided combat would be a percentage based opposing roll system. First the attacker will roll a number of 100 sided dice equal to their attack value. Each die in the roll will have a percentage chance to be a success. Each success has the potential to inflict 1 point of damage. Thus the maximum amount of damage an actor can do is their `Attack` value, if they get all successes and their is no defense.

So how about defense? It will work the same way. The defender rolls a number of 100 sided dice equal to their `Defense` value. Each die has a percentage chance to be a success. The number of defender successes is subtracted from the number of attacker successes and if the remaining number is positive then that much damage is applied to the defender.

## Important Stats for Combat

* `Attack` – int – Number of dice to roll when performing an attack. Also represents the maximum amount of damage that can be inflicted in a single attack.
* `AttackChance` – int – Percentage chance that each die rolled is a success. A success for a die means that 1 point of damage was inflicted.
* `Defense` – int – Number of dice to roll when defending against an attack. Also represents the maximum amount of damage that can blocked or dodged from an incoming attack.
* `DefenseChance` – int – Percentage chance that each die rolled is a success. A success for a die means that 1 point of damage was blocked.
* `Health` – int – The current health total of the actor. If this value is 0 or less then the actor is killed.
* `MaxHealth` – int – How much health the actor has when fully healed.

## Additional DungeonMap Methods

Before we start rolling dice we need to add a couple more methods to `DungeonMap.cs` to handle getting a monster at a particular position, and removing a monster from the map. We already have a method `AddMonster()` so lets put the new methods directly below that. Open `DungeonMap.cs` and add the following code.

```cs
public void RemoveMonster( Monster monster )
{
  _monsters.Remove( monster );
  // After removing the monster from the map, make sure the cell is walkable again
  SetIsWalkable( monster.X, monster.Y, true );
}

public Monster GetMonsterAt( int x, int y )
{
  return _monsters.FirstOrDefault( m => m.X == x && m.Y == y );
}
```

## Adding the Attack Command

Now for the fun part, the actual combat. Attacking will be another command just like moving so we will add the code to `CommandSystem.cs`. The first thing we need to do is update the `MovePlayer()` method to use our new `DungeonMap.GetMonsterAt( x, y )` method to see if there is a monster in the `Cell` we are trying to move into. When there is then instead of moving we will attack it.

Add the following code to the `MovePlayer()` method in `CommandSystem.cs` after the `if` block where we try to `SetActorPosition`. This is also after we have established our x and y position based on the direction the player is trying to move.

```cs
Monster monster = Game.DungeonMap.GetMonsterAt( x, y );

if ( monster != null )
{
  Attack( Game.Player, monster );
  return true;
}
```

Now that is in place, lets make our `Attack()`r method.

```cs
public void Attack( Actor attacker, Actor defender )
{
  StringBuilder attackMessage = new StringBuilder();
  StringBuilder defenseMessage = new StringBuilder();

  int hits = ResolveAttack( attacker, defender, attackMessage );

  int blocks = ResolveDefense( defender, hits, attackMessage, defenseMessage );

  Game.MessageLog.Add( attackMessage.ToString() );
  if ( !string.IsNullOrWhiteSpace( defenseMessage.ToString() ) )
  {
    Game.MessageLog.Add( defenseMessage.ToString() );
  }

  int damage = hits - blocks;

  ResolveDamage( defender, damage );
}

// The attacker rolls based on his stats to see if he gets any hits
private static int ResolveAttack( Actor attacker, Actor defender, StringBuilder attackMessage )
{
  int hits = 0;

  attackMessage.AppendFormat( "{0} attacks {1} and rolls: ", attacker.Name, defender.Name );

  // Roll a number of 100-sided dice equal to the Attack value of the attacking actor
  DiceExpression attackDice = new DiceExpression().Dice( attacker.Attack, 100 );
  DiceResult attackResult = attackDice.Roll();

  // Look at the face value of each single die that was rolled
  foreach ( TermResult termResult in attackResult.Results )
  {
    attackMessage.Append( termResult.Value + ", " );
    // Compare the value to 100 minus the attack chance and add a hit if it's greater
    if ( termResult.Value >= 100 - attacker.AttackChance )
    {
      hits++;
    }
  }

  return hits;
}

// The defender rolls based on his stats to see if he blocks any of the hits from the attacker
private static int ResolveDefense( Actor defender, int hits, StringBuilder attackMessage, StringBuilder defenseMessage )
{
  int blocks = 0;

  if ( hits > 0 )
  {
    attackMessage.AppendFormat( "scoring {0} hits.", hits );
    defenseMessage.AppendFormat( "  {0} defends and rolls: ", defender.Name );

    // Roll a number of 100-sided dice equal to the Defense value of the defendering actor
    DiceExpression defenseDice = new DiceExpression().Dice( defender.Defense, 100 );
    DiceResult defenseRoll = defenseDice.Roll();

    // Look at the face value of each single die that was rolled
    foreach ( TermResult termResult in defenseRoll.Results )
    {
      defenseMessage.Append( termResult.Value + ", " );
      // Compare the value to 100 minus the defense chance and add a block if it's greater
      if ( termResult.Value >= 100 - defender.DefenseChance )
      {
        blocks++;
      }
    }
    defenseMessage.AppendFormat( "resulting in {0} blocks.", blocks );
  }
  else
  {
    attackMessage.Append( "and misses completely." );
  }

  return blocks;
}

// Apply any damage that wasn't blocked to the defender
private static void ResolveDamage( Actor defender, int damage )
{
  if ( damage > 0 )
  {
    defender.Health = defender.Health - damage;

    Game.MessageLog.Add( $"  {defender.Name} was hit for {damage} damage" );

    if ( defender.Health <= 0 )
    {
      ResolveDeath( defender );
    }
  }
  else
  {
    Game.MessageLog.Add( $"  {defender.Name} blocked all damage" );
  }
}

// Remove the defender from the map and add some messages upon death.
private static void ResolveDeath( Actor defender )
{
  if ( defender is Player )
  {
    Game.MessageLog.Add( $"  {defender.Name} was killed, GAME OVER MAN!" );
  }
  else if ( defender is Monster )
  {
    Game.DungeonMap.RemoveMonster( (Monster) defender );

    Game.MessageLog.Add( $"  {defender.Name} died and dropped {defender.Gold} gold" );
  }
}
```

That’s a lot of code but I tried to add some comments to help clarify. It should read very similar to the Combat Overview section above. You’ll notice that instead of having one gigantic public method with 100 lines of code we extracted some private methods. I like doing this because you can quickly look at the public method and see what it is doing without having to know all of the details. Resolve the attack rolls, resolve the defense rolls, write the results to the message log, and resolve any damage that was sustained.

Giving the private methods clear names for their purpose can help with readability. Someone who doesn’t care about the internals can just skip over those, or if they do need to see the details of what it means to resolve an attack, they dive in and look at that private method.

You might notice that in the death message it mentions dropping gold. We haven’t actually implemented that yet but we’ll get to it soon. I don’t mind having the message in place and ready to go for when we do.

If the dice expressions seem a little tricky here you can find the API documentation for them here:

* [Dice Notation Documentation](../api/RogueSharp.DiceNotation.yml)

Run the program now and you should be able to move around the map and bump into monsters to damage them.

As always the code for the tutorial series so far can be found on GitHub:
<https://github.com/FaronBracy/RogueSharpV3Tutorial/tree/12SimpleCombat>

## Closing Thoughts

When you are going through the code examples here, please don’t feel like you have to type them in letter for letter. For example in this tutorial try changing the messages that get added to the log to make them more interesting. Create additional stats for actors and make up your own more exciting combat system. Rename and reorganize anything and everything you want so that it makes sense to you.

The code presented in this tutorial series is far from perfect. I am finding lots of issues that bug me every time I go back to write one of these posts. The point is that the examples I provide here are just one way of doing things. Get some ideas here and then visit other blogs and read other tutorials to collect more ideas. Eventually you’ll settle upon what works best for you and your own unique style.

Bored waiting for the next tutorial? The completed project is already available on Github.
<https://github.com/FaronBracy/RogueSharpRLNetSamples>
