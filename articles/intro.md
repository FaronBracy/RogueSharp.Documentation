# Introduction

Since the pre-release of RogueSharp V3 I have been working on a complete tutorial for creating a Roguelike game in C#. Along with the announcement of V3 being fully released I would also like to announce the start of a new tutorial series.

![alt text](/images/V3Tutorial/01_tutorialgame.png "Running tutorial game")

## Questions and Answers

**Question:** You never finished the previous MonoGame + RogueSharp tutorials you were working on. How do you expect this one will be different?

**Short Answer:** The complete tutorial project is already finished and the source code is available on BitBucket. All that is left to do is for me to clean up the commits and write about it in a series of blog posts. I’m confident that I can accomplish that during the upcoming weeks.

**Long Answer:** The history of RogueSharp is that it was a library originally created for myself for a personal project I was making using MonoGame (XNA then) and C#. At the time I couldn’t find a similar native library for handling path-finding, field-of-view, etc. so I created one myself. I decided that it would be beneficial to share it with the community especially since the inspiration came from so many other open projects like libtcod. This blog was a bit of an afterthought. I wanted people to use the library so I opened up visual studio and started writing a sample project without any planning or forethought. The intent was really just to give examples of RogueSharp features and how to use them. I didn’t have a well thought out plan for making a complete sample game.

**Question:** Why did you choose to use RLNet this time instead of MonoGame?

**Answer:** MonoGame is a great library. I choose RLNet this time because of simplicity. I feel that with the simple ASCII graphics I can focus more on the general design and code and less on the graphics, animation, cameras and all the other bits that go along with MonoGame. Also, I feel like there are already a lot of tutorials available for XNA / MonoGame.

## Goals

The RogueSharp V3 Tutorial project was created with the following goals in mind:

* An inventory system for the player
  * Equipment slots for head, body, hands and feet
  * Item slots for consumables like potions, wands and scrolls
    * Items have a number of charges before they used up
* A robust example set of abilities that the player can learn
  * Targeted attacks like Magic Missiles that can affect single monsters at range
  * Area attacks like Fireballs that hit all monsters in an area
  * Line attacks like a Lightning bolt that can be fired through monsters
  * Healing abilities
  * Map revealing abilities
  * Melee special attacks like a whirlwind attack that hits all adjacent enemies
* An interesting set of enemies with differing AI
  * Goblins that can pick up and equip items they find
  * Enemies that flee when wounded and alert other groups of monsters to the player’s presence
  * Oozes that split into multiple copies when struck
* A unique combat system
  * Percentage based
  * Opposing rolls
* A scheduling system
  * No more I-Go / You-Go
  * Player speed changes based on the items equipped
  * Monster can vary wildly
* A messaging system
  * Show detailed combat messages in a log
* Systems for generating random levels
  * Distribution of loot and monsters based on level depth
  * Map features such as rooms, hallways, doors and stairs that can be interacted with by both the player and monsters

I’m looking forward to writing more about this tutorial project. I have the next 4 posts already started. In the meantime there is a great community you can visit to learn more about Rougelike development.

* Roguelike Dev SubReddit – <https://www.reddit.com/r/roguelikedev>

Also, don’t forget about the upcoming seven day roguelike for 2016. If you are interested in C# I’d love for you to try out RougeSharp this year and give me any feedback you have. If you aren’t interested in C# there are lots of other great libraries and languages out there. Whether you pick an existing one or roll your own I think you’ll have a lot of fun.

* 7DRL 2016 – <http://7drl.org/2016/01/13/7drl-2016-announced-for-5-13-march/>
