---
title: Building Battleship - Part II
description: Building a graphically interactive console game using the Spectre.Console library
date: 2025-07-04 21:01:00 +0000
categories: [Code, Projects]
author: rholder
tags: [games, console, Spectre.Console, c#]
---


## The Dreaded 'God' Class
The code below shows a portion  of the `GameDisplay` class taken from GameDisplay.cs.

```c#
public partial class GameDisplay
{
     private Layout gameLayout;
     private LiveDisplay liveDisplay;
     private Player gamePlayer1;
     private Player gamePlayer2;
     private Player victoriousPlayer;

     private GameStatus gameStatus;
     private DisplayMode displayMode;
     private ShipPlacementMode shipPlacementMode;
     private ShipOrientation shipOrientation;
     private Menu mainMenu;
     private Menu shipMenu;

     public GameDisplay()
     {
         SetupPlayers();
         CreateMenus();
         SetGameStatus(GameStatus.NotStarted);
         SetDisplayMode(DisplayMode.MainMenu);
         CreateGameLayouts();
     }
     public void PlayGame()
     {
         ShowDisplay();
     }
     private void ShowDisplay()
     {
         SetupConsole();
         SetupLiveDisplay(gameLayout);
         StartLiveDisplay();
     }
}
```

<br>

## Future Improvements
The god class `GameDisplay` can definitely be refactored into a set of smaller single responsibility classes now that I have the big picture of how everything needs to work and fit together. There is also some code duplication in terms of how updating the layout is handled which can be removed by refactoring. 
## Conclusion

<br>

|![HumanContent](/assets/posts/badges/HumanContent_08.png) ![MadeByAHuman](/assets/posts/badges/MadeByAHuman_07.png) ![NeverByAI](/assets/posts/badges/NeverByAi_01.png)| 
