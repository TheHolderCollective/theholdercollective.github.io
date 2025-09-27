---
title: Building Battleship - Part II
description: Building a graphically interactive console game using the Spectre.Console library
date: 2025-09-25 20:00:00 +0000
categories: [Code, Projects]
author: rholder
tags: [games, console, Spectre.Console, c#, csharp, battleship]
image:
  path: /assets/preview_images/mckenzie-warship-1200x630.jpg
  alt: Image by Francis Cooper-McKenzie
---

A preliminary draft of this post has been sitting in my draft folder for the past several weeks, just screaming at the top its lungs for completion. Life did what it does and "lifed", and so it never got done. I've finally gotten around to finishing it. So here goes...

In the [last post](/posts/Building-Battleship-Part-1/) about the Battleship project, I looked at the layouts and how they were composed. I also introduced the `GameDisplay` class. I had promised to delve a bit deeper into the inner workings of the `GameDisplay` class in this post, but I've decided instead to give it its own post due to its comparatively large size and complexity. So there will, at some point in time, be a third installment documenting my efforts for this project.

In this installment, I'll be focusing on the changes made to the models in order to achieve the goal of making the game playable by a human player against a computer opponent.

## Overview of Changes
In order to make the game playable, updates to the code had to be made to keep track of ship coordinates for initial player positioning and in-battle targeting.

A couple new properties,`ShipType` and `IsPlaced`, were added to the abstract class `Ship` to support ship positioning:

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
The first three properties of the `ShipPlacements` class, I think, are self-explanatory. The `Orientation` property stores the orientation of the ship which can either be vertical or horizontal. A new enum `ShipOrientation` was defined for this purpose. The `ShipCoordinates` property stores the end to end coordinates of the ship. This makes it easier to calculate new coordinates when a ship needs to be rotated or repositioned. 

The `PlacementLog` property is used to store a text string containing the ship name and its end to end coordinates. As shown in the code, each time the coordinates for a ship's position are updated, the `PlacementLog` property is also updated. This property is used to facilitate live updates of ship positions when they are being placed by the player. These updates can be seen on the 'Ship Placement Updates' panel (shown below).

![ShipPlacementUpdatesPanel](/assets/posts/20250924/ShipPlacementUpdatesPanel.jpg){: width="609" height="128"}
_Ship placement updates panel_

The functionality supplied by this class was used extensively in the `Player` class in its newly added placement and rotation methods. This class will be looked at more closely in the following section.

## The Player Class

The `Player` class had the most changes of all of the pre-existing models in order to allow for manual ship positioning and firing of shots. 
Eleven new properties and 20 new methods were added to the class. Two pre-existing methods were also removed. I will go through the new methods first, focusing on the what of the methods rather than the how. I therefore won't be digging into the details of the method code. The properties and their relationship to the methods will be covered after.

### The Methods
The `Player` class methods, new and old, are shown in the code listing below. Their bodies have been omitted for brevity.

```c#
public class Player
{
  // Pre-existing methods
  public void ProcessShotResult(Coordinates coords, ShotResult result) {...}
  public ShotResult ProcessShot(Coordinates coords) {...}
  private Coordinates SearchingShot() {...}
  private Coordinates RandomShot() {...}

  // New methods -- Outputting board statuses
  public string[,] OutputGameBoard() {...}
  public string[,] OutputFiringBoard() {...}

  // New methods -- Targeting
  public void MoveCrosshairs(Direction direction) {...}
  private void PlaceInitialCrossHairs() {...}
  private void PlaceCrosshairs(int row, int column) {...}
  
  // New methods -- Ship positioning and placement
  public void PlaceShipsRandomly() {...}
  public bool PlaceShip(ShipType shipType, ShipOrientation shipOrientation, int startRow, int startCol) {...}
  public bool MoveShip(ShipType shipType, Direction shipDirection) {...}
  public bool RotateShip(ShipType shipType) {...}
  
  // New methods -- Firing shots
  public bool IsShotAvailable() {...}
  public Coordinates FireManualShot() {...}
  public Coordinates FireAutoShot() {...}

  // New methods -- Ship position logging
  private void UpdateShipPlacementLog() {...}
  
  // New methods -- Helpers for ship positioning and placement
  private Ship GetShip(ShipType shipType) {...}
  private ShipPlacements GetShipLocation(ShipType shipType) {...}
  private bool IsBoardRangeUnoccupied(List<GameBoardPanel> selectedPanels) {...}
  private bool IsShipWithinBounds(int startRow, int startCol, int endRow, int endCol) {...}
  private Coordinates[] CalculateRotatedShipCoordinates(ShipOrientation shipOrientation, int pivotPanel, int shipLength, int pivotRow, 
                                       int pivotColumn, int panelCountBeforePivot, int panelCountAfterPivot) {...}
  private void SetPivotAndAdjacentPanelCounts(ShipType shipType,ref int pivotPanel, ref int beforePivotPanelCount, 
                                       ref int afterPivotPanelCount) {...}
  private Coordinates CalculateShipEndPoint(Ship ship, ShipOrientation shipOrientation,
                                       Coordinates shipStart) {...}
}
```
`OutputBoards` and `FireShot` were the two methods removed from the original implementation of the `Player` class.`OutputBoards` was removed because it output the game and firing boards directly to the console and a different kind of functionality was needed. It was replaced by the methods `OutputGameBoard` and  `OutputFiringBoard` which output the game and firing boards as string arrays instead. This allowed for greater flexibility in the positioning of the boards on their respective panels.

The `FireShot` method technically wasn't removed. It was renamed as `FireAutoShot` to differentiate between shots fired by a player versus shots fired by an AI. The `FireManualShot` method was added to handle shots taken by a human player, and the `IsShotAvailable` method was added to check whether or not the targeted spot has been shot at. 

In order to "manually" take a shot, some targeting functionality was needed and so the method `MoveCrosshairs`, and its supporting helper methods `PlaceInitialCrossHairs` and `PlaceCrosshairs` were added. `MoveCrosshairs` updates the `CrosshairsPosition` property with respect to  the direction specified by its parameter. The `FiringBoard` property is also updated by the `PlaceCrosshairs` method called within `MoveCrosshairs` provided that the new position falls within the allowed boundaries. `PlaceInitialCrossHairs` is used within the constructor to set the initial position of the crosshairs. 


The methods `PlaceShip`,`MoveShip`, and `RotateShip` are used to position and place ships on the gameboard manually. These are typically called when a player is placing their ships before commencing a battle. The pre-existing method `PlaceShips` was renamed to `PlaceShipsRandomly`, and does exactly what its name suggests. This method is used by the AI player to place its ships, and can be invoked when a human player opts for random ship placement.

`UpdateShipPlacementLog` is a helper function which updates the `ShipPlacementLogs` property which stores every ship's position on the game board. It is called each time a ship's position is changed or a ship is placed. 

The remaining methods in the last section of the code listing are helper methods which are used in either the `PlaceShip`,`MoveShip`, or `RotateShip` methods to simplify the code and calculations done in order to reposition or place a ship. I won't be looking at them beyond this cursory mention.

### The Properties
The  `Player` class properties are shown in the code snippet below.

```c#
public class Player
{
  // Pre-existing properties
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

The `ShipLocations` property stores all of the ship positions, and it is used by the `UpdateShipPlacementLog` method to update the `ShipPlacementLogs` property as was previously mentioned. The `ShipsAlreadyPlaced` property functions as a flag to indicate if all ships have been placed on the game board. It is used by the `PlaceShipsRandomly` method to determine if the board is clear for ship placement.

The `FiredShot` and `ReceivedShot` properties store the text information for the shots fired and received by a player respectively. The `ShipStatus` property stores text about the last ship sunk. The `RoundNumber` property keeps track of the number of completed rounds. These properties are all used in conjunction with each other to give the complete text updates shown on the 'Battle Updates' panel about the state of the battle as each round progresses. 


The `CrosshairsPosition` property stores the current position of the player's crosshair. The  `CrosshairsX ` and `CrosshairsY` properties return the row and column values of `CrosshairsPosition` respectively. They are used to generate the targeting info shown on the 'Targeting Dashboard' panel. The `PriorOccupationType` property stores the last occupation type of a position, and is used to determine whether or not a game board panel location is available to be fired at by a player. This is used by the `IsShotAvailable` method.


## Refactoring the Player Class
As I was reviewing the code, I started to see the design faux pas I'd made when I was modifying `Player` class. I realised there should have been a distinction made between a human player and an AI player. This would have made the code handling player interactions cleaner, as well as made it easier to add different playing strategies to the AI player with minimal impact to the rest of the `Player` class code.  

So I refactored the `Player` class making it an abstract class, moved all human player specific operations to a class called `HumanPlayer`, and all AI player specific operations to a class called `AIPlayer`. Both classes are derived from the `Player` class.  The updated code sans method bodies is shown below.

```c#
public abstract class Player
{
  // Properties
  public string Name { get; set; }
  public GameBoard GameBoard { get; set; }
  public FiringBoard FiringBoard { get; set; }
  public List<Ship> Ships { get; set; }
  public List<ShipPlacements> ShipLocations { get; set; }
  public List<string> ShipPlacementLogs { get; }
  public string FiredShot { get; set; }
  public string ReceivedShot { get; set; }
  public string ShipStatus { get; set; }
  public int RoundNumber { get; set; }
  public bool HasLost {...}
  protected OccupationType PriorOccupationType { get; set; }
  private bool ShipsAlreadyPlaced {...}
  
  // Methods
  public Player(string name) {...}
  public abstract Coordinates FireShot();
  public string[,] OutputGameBoard(GameBoardType gameBoardType) {...}
  public ShotResult ProcessShot(Coordinates coords) {...}
  public void ProcessShotResult(Coordinates coords, ShotResult result) {...}
  public bool IsShotAvailable() {...}
  public void PlaceShipsRandomly() {...}

  // Helper methods
  private void UpdateShipPlacementLog() {...}
}

public class HumanPlayer : Player
{
  // Properties
  private Coordinates CrosshairsPosition { get; set; }
  public int CrosshairsX {...}
  public int CrosshairsY {...}

   // Methods
  public HumanPlayer(string name) : base(name) {...}
  public override Coordinates FireShot() {...}
  public void MoveCrosshairs(Direction direction) {...}
  public bool PlaceShip(ShipType shipType, ShipOrientation shipOrientation, int startRow, int startCol) {...}
  public bool MoveShip(ShipType shipType, Direction shipDirection) {...}
  public bool RotateShip(ShipType shipType) {...}

  // Helper methods
  private void PlaceCrosshairs(int row, int column) {...}
  private void PlaceInitialCrossHairs() {...}
  private Ship GetShip(ShipType shipType){...}
  private ShipPlacements GetShipLocation(ShipType shipType){...}
  private bool IsBoardRangeUnoccupied(List<GameBoardPanel> selectedPanels){...}
  private bool IsShipWithinBounds(int startRow, int startCol, int endRow, int endCol){...}
  private Coordinates[] CalculateRotatedShipCoordinates(ShipOrientation shipOrientation, int pivotPanel, int shipLength, int pivotRow, 
                                       int pivotColumn, int panelCountBeforePivot, int panelCountAfterPivot){...}
  private void SetPivotAndAdjacentPanelCounts(ShipType shipType,ref int pivotPanel, ref int beforePivotPanelCount, 
                                       ref int afterPivotPanelCount){...}
  private Coordinates CalculateShipEndPoint(Ship ship, ShipOrientation shipOrientation,
                                       Coordinates shipStart){...}
}

public class AIPlayer : Player
{
  // Methods
  public AIPlayer(string name) : base(name) {...}
  public override Coordinates FireShot() {...}

  // Helper methods
  private Coordinates RandomShot() {...}
  private Coordinates SearchingShot() {...}
}

```

The `OutputGameBoard` method in the `Player` class was refactored to take a `GameBoardType`, and the `OutputFiringBoard` method was removed. This eliminated the code duplication in the class.

The `FireManualShot` and `FireAutoShot` methods disappeared, and became one abstract method `FireShot` in the base class `Player`. The code for `FireAutoShot`  went into the implementation of the `FireShot` for the `AIPlayer` class, and likewise the code for `FireManualShot` went into the implementation of the `FireShot` for the `HumanPlayer` class. 

This had the nice side effect of reducing a bit of code duplication in the GameDisplay.Play.cs module. The module code before refactoring is shown below.

```c#
public partial class GameDisplay
{
  private void PlayRound() 
  {
      if (gamePlayer1.IsShotAvailable() && gameStatus != GameStatus.GameOver)
      {
          FireShotHumanPlayer();

          if (gamePlayer2.HasLost)
          {
              SetGameStatus(GameStatus.GameOver);
              victoriousPlayer = gamePlayer1;
              return;
          }

          FireShotComputerPlayer();

          if (gamePlayer1.HasLost)
          {
              SetGameStatus(GameStatus.GameOver);
              victoriousPlayer = gamePlayer2;
              return;
          }
      }
  }
  private void FireShotHumanPlayer()
  {
      var coordinates = gamePlayer1.FireManualShot();
      var result = gamePlayer2.ProcessShot(coordinates);
      gamePlayer1.ProcessShotResult(coordinates, result);
  }

  private void FireShotComputerPlayer()
  {
      var coordinates = gamePlayer2.FireAutoShot();
      var result = gamePlayer1.ProcessShot(coordinates);
      gamePlayer2.ProcessShotResult(coordinates, result);
  }
}
```
After refactoring, the `FireShotHumanPlayer` and `FireShotComputerPlayer` methods became one method `FireShotAtOpponent`. I think the intent of the code became a bit easier to understand after this change, if not cleaner.

```c#
public partial class GameDisplay
{
  private void PlayRound() 
  {
    if (gamePlayer1.IsShotAvailable() && gameStatus != GameStatus.GameOver)
    {
        FireShotAtOpponent(gamePlayer1, gamePlayer2);

        if (gamePlayer2.HasLost)
        {
            SetGameStatus(GameStatus.GameOver);
            victoriousPlayer = gamePlayer1;
            return;
        }

        FireShotAtOpponent(gamePlayer2, gamePlayer1);

        if (gamePlayer1.HasLost)
        {
            SetGameStatus(GameStatus.GameOver);
            victoriousPlayer = gamePlayer2;
            return;
        }
    }
  }
  private void FireShotAtOpponent(Player firingPlayer, Player opponent)
  {
      var coordinates = firingPlayer.FireShot();
      var result = opponent.ProcessShot(coordinates);
      firingPlayer.ProcessShotResult(coordinates, result);
  }
}
```

 The refactored Player class and its derivatives can be found in the ***player-update*** branch of the GitHub [repo](https://github.com/TheHolderCollective/Battleship). I did some intial light testing of the game after making the changes and nothing seems to be broken. It will be merged into the main branch once I'm satisfied that nothing is really broken.

The one drawback of this refactoring was that I had to explicitly declare one of the players as a `HumanPlayer` in the `GameDisplay` class because of the coupling of the `PlaceShip`, `MoveShip`, and `RotateShip` methods with some of the update display functionality in this class. I hope to be able to eventually decouple and hide these functions from the `GameDisplay` class when it is eventually refactored. There is certainly a lot of work to do on that front. 


## Conclusion
Time away from a project gives a fresh perspective, and it allowed me to see the mess I overlooked when I was learning how to use Spectre.Console in order to build the game. I was reasonably satisfied with the results of refactoring the Player class, and I have some ideas going forward as to how the Gordian knot which is the `GameDisplay` class can untangled into something cleaner and more manageable. I will be tackling this class in all of its convoluted glory in the next post of this unintentional series about building Battleship.



<br>

|![HumanContent](/assets/posts/badges/HumanContent_08.png) ![MadeByAHuman](/assets/posts/badges/MadeByAHuman_07.png) ![NeverByAI](/assets/posts/badges/NeverByAi_01.png)| 
