---
title: Building Battleship - Part II
description: Building a graphically interactive console game using the Spectre.Console library
date: 2025-09-23 20:00:00 +0000
categories: [Code, Projects]
author: rholder
tags: [games, console, Spectre.Console, c#]
image:
  path: /assets/preview_images/BattleShip-1200x630.jpg
---

A preliminary draft of this post has been sitting in my draft folder for the past several weeks, just screaming at the top its lungs for completion. Life did what it does and "lifed", and so it never got done. I've finally gotten around to finishing it. So here goes...

In the [last post](/posts/Building-Battleship-Part-1/) about the Battleship project, I looked at the layouts and how they were composed. I also introduced the `GameDisplay` class. I had promised to delve a bit deeper into the inner workings of the `GameDisplay` class in this post, but I've decided instead to give it its own post due to its comparatively large size and complexity. So there will at some point in time be a third installment documenting my efforts for this Battleship project.

In this installment, I'll be focusing on the changes made to the models in order to achieve the goal of making the game playable by a human player against a computer opponent.

## Overview of Changes
In order to make the game playable, updates to the code had to be made to keep track of ship coordinates for intial player positioning and in-battle targetting.

A couple new properties,`ShipType` and `IsPlaced`, were added to the abstract class `Ship` to support ship positionining:

```c#
public abstract class Ship
{
  public string Name { get; set; }
  public int Width { get; set; }
  public int Hits { get; set; }
  public OccupationType OccupationType { get; set; }
  public bool IsSunk
  {
      get
      {
          return Hits >= Width;
      }
  }

  public ShipType ShipType { get; set;} // new
  public bool IsPlaced { get; set; }    // new
}
```
As the name suggests, the `ShipType` property stores the type of battle ship. A new enum called `ShipType` was defined because the defintion of the enum `OccupationType` was too broad and didn't quite fit how I wanted the ship positioning algorithms to work. The `IsPlaced` property was added to facilitate checks on whether or not a ship had been placed on the board.

To further support the ship positioning functionality, a new class `ShipPlacements` was created to store and log the positions of player ships:

```c#
public class ShipPlacements
{
  public ShipType TypeOfShip { get; set; }
  public ShipOrientation Orientation { get; set; }
  public Coordinates[] ShipCoordinates { get; set; } 
  public string PlacementLog { get; set; }

  public ShipPlacements(){}

  public ShipPlacements(ShipType shipType, ShipOrientation shipOrientation, Coordinates[] shipCoordinates)
  {
      TypeOfShip = shipType;
      Orientation = shipOrientation;
      UpdateCoordinates(shipCoordinates);
  }

  public void UpdateCoordinates(Coordinates[] coordinates)
  {
      ShipCoordinates = new Coordinates[coordinates.Length];

      for (int i = 0; i < coordinates.Length; i++)
      {
          ShipCoordinates[i] = new Coordinates(coordinates[i].Row, coordinates[i].Column);
      }

      UpdatePlacementLog();
  }

  private void UpdatePlacementLog()
  {
      int shipStartRow = ShipCoordinates[0].Row;
      int shipStartCol = ShipCoordinates[0].Column;
      int shipEndRow = ShipCoordinates[ShipCoordinates.Length - 1].Row;
      int shipEndCol = ShipCoordinates[ShipCoordinates.Length - 1].Column;

      string shipName = TypeOfShip.GetAttributeOfType<DescriptionAttribute>().Description;

      PlacementLog = $"> {shipName} placed at ({shipStartRow},{shipStartCol} -- ({shipEndRow},{shipEndCol})";
  }
}
```
The first three properties I think are self-explanatory. The `Orientation` property stores the orientation of the ship which
can either be vertical or horizontal. A new enum `ShipOrientation` was defined for this purpose. The `ShipCoordinates` property stores the end to end coordinates of the ship. This makes it easier to calculate new coordinates when a ship needs to be rotated or repositioned. 

The `PlacementLog` property is used to store a text string containing the ship name and its end to end coordinates. As shown in the code, each time the coordinates for a ship's position are updated, the `PlacementLog` property is also updated. This property is used to facilitate live updates of ship positions when they are being placed by the player. These updates can be seen on the 'Ship Placement Updates' panel:

![ShipPlacementUpdatesPanel](/assets/posts/20250924/ShipPlacementUpdatesPanel.jpg){: width="609" height="128"}
_Ship placement updates panel_

The functionality supplied by this class was used extensively in the `Player` class in its placement and rotation functions. The `Player` class had the most changes of all of the pre-existing models in order to allow for manual ship positioning and firing of shots. This will be looked at more closely in the following section.

## The Player Class

Extensive changes were made to the `Player` class, with 11 new properties and 18 new methods being added to the class. I will go through the new properties first followed by a basic overview of the new methods. I won't be digging too deep into the details of the method code.

```c#
public class Player
{
  // Original properties
  public string Name { get; set; }
  public GameBoard GameBoard { get; set; }
  public FiringBoard FiringBoard { get; set; }
  public List<Ship> Ships { get; set; }
  public bool HasLost {...}
  
  // Newly added properties
  public List<ShipPlacements> ShipLocations { get; set; }
  public List<string> ShipPlacementLogs { get; }
  public string FiredShot { get; set; }
  public string ReceivedShot { get; set; }
  public string ShipStatus { get; set; }
  public int RoundNumber { get; set; }
  public int CrosshairsX {...}
  public int CrosshairsY {...}

  private bool ShipsAlreadyPlaced {...} 
  private Coordinates CrosshairsPosition { get; set;}
  private OccupationType PriorOccupationType { get; set; }

  // remainder of class follows on
}
```

## Refactoring the Player Class
As I was reviewing the code, I started to see the design faux pas I'd made when I was modifying Player class.

![ShipPlacementUpdatesPanel](/assets/posts/20250924/TestUMLDiagram.jpg){: width="677" height="602"}
_Test UML Diagram (to be replaced)_

## Conclusion
Insert conclusion here

<br>

|![HumanContent](/assets/posts/badges/HumanContent_08.png) ![MadeByAHuman](/assets/posts/badges/MadeByAHuman_07.png) ![NeverByAI](/assets/posts/badges/NeverByAi_01.png)| 
