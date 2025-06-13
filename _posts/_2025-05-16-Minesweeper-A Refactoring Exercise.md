---
title: Minesweeper - A Refactoring Exercise
description: First explorations of Spectre.Console while refactoring Minesweeper
date: 2025-05-16 19:20:00 +0000
categories: [Code, Projects]
author: rholder
tags: [refactoring, Spectre.Console, c#]
---

I was browsing GitHub a couple months ago when I came across a basic version of console Minesweeper. It gave me flashblacks of attempting to play it as a teenager and always hitting bombs. I never understood the game back then, but that was because I'd never read the rules. In any case, it seemed quite boring at the time when compared to games which contained beautifully rendered graphics, movable characters, and titillating sounds. 

After reading the rules (which turned out to be rather simple), I downloaded and built the code for the console project. On my first run of the game I was able to win my very first game of Minesweeper. After that I found a few sites online, played a few games, and for a short period of time became hooked on Minesweeper. I thoroughly enjoyed working out where the bombs were and discovering the patterns.

Looking at the code for the game I'd downloaded, I saw that there were roughly 460 lines all contained in the Program.cs file. I thought that it would be a fun exercise to improve the game by refactoring the code. I also decided that this would be a good time to try out the [Spectre.Console](https://spectreconsole.net/) library in order to add some graphical improvements.  The remainder of this post will be a general description of that process.


## Creating the models

The first step in the refactor was evacuating the code from Program.cs into an **Engine** class. Doing this significantly reduced the lines of executable code in Program.cs: 

```c#
namespace MineSweeper;

public class Program
{
    static void Main(string[] args)
    {
        Engine minesweeperEngine = new Engine();
        minesweeperEngine.Run();
    }
}
```

Program.cs went from 460+ lines of code to approximately 10 lines. Further analysis of the code then revealed that the game board and its cell contents could be modeled separately in their own classes called **Board** and **BoardElement** respectively. Using the Spectre.Console library in order to have colour graphics and basic menu functionality resulted in the creation of the **Display**, **Menu**, and **MenuOption** classes. 

So I ended up with a total of six new classes:
- MenuOption
- Menu
- Display
- BoardElement
- Board
- Engine

## Overview of the classes

The **MenuOption** class stores the data associated with one menu item:

```c#
public class MenuOption
{
    public string OptionName { get; set; }
    public int OptionValue { get; set; }

    public MenuOption(string name, int value)
    {
        OptionName = name;
        OptionValue = value;
    }
}
```

The **Menu** class aggregates MenuOption objects into a list via its constructor:

```c#
public Menu(MenuOption[] options)
{
    optionsList = new List<MenuOption>();

    foreach (MenuOption option in options)
    {
        optionsList.Add(option);
    }
}
```

This list is then used to build and display a text menu using the Spectre.Console function: `AnsiConsole.Prompt`.  
The  `AnsiConsole.Prompt` function makes it easy to display, select, and retrieve menu options:  

```c#
public int ShowMenu()
{
    var menuOptions = GetNamesArray();
    var menu = AnsiConsole.Prompt(new SelectionPrompt<string>()
                       .Title(MenuPrompt)
                       .PageSize(5)
                       .AddChoices(menuOptions));

    return GetOptionValue(menu);
}
```

The **Display** class manages all of graphical output for the game. It handles the display of the game title, menus, game board, and game over panels. The **Menu** class was used in the **Display** class to build the three menus used in the game. 

The **BoardElement** class was used to store and retrieve the identifying properties of the different types of board cells. It was used in both the **Display** and **Board** classes for comparison and formatting purposes in the Display and Board classes.



