---
title: "Learning Python by Implementing Battleship"
layout: post
date: 2019-02-06 12:00
tag: 
- Python
- Battleship
- New Language
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "Learning Python by Implementing Battleship"
category: project
author: Jerry Li
externalLink: false
--- 
## Overview
One of the goals I set for myself was to learn Python. Since I was quite familiar with other object oriented programming languages such as Java and C#, I was able to go through the Python Beginner's guide quite quickly but the key to success in really learning any new skill is to practice. I decided to practice Python by implementing the classic game of Battleship. My goal was to start small - a 1 vs 1 against the computer and then slowly add features and refactor the code to a more Pythonic standard.

## Implementation in Steps
### First Iteration: Working Game
The first step was to implement a working version of the game. In this iteration, the goal was to complete the game and make a working copy. The easiest way was to face a computer opponent. The display was text-based and all the board values were represented with characters. The user sees two boards, one for his or her own map and another one for the enemy map with only the hits or misses. The user can place the ships on the board and then choose which cell to attack. All of this is done through the console, the user would select a cell by entering a coordinate such as "C10" and then the program would parse the input into a coordinate on the map. The computer places ships randomly by choosing the coordinate from a random number generator. The coordinate the computer chooses is also randomly selected. This is less than optimal. A more efficient algorithm could take into account current hits to determine where to choose a next strike instead of randomly choosing a spot on the board.

The board looks like the following:
```
    A   B   C   D   E   F   G   H   I   J   
1   C   -   -   -   -   -   -   -   -   -   
2   C   -   -   -   -   -   -   -   -   -   
3   C   -   B   B   B   B   -   -   M   -   
4   C   -   -   -   -   -   -   -   -   -   
5   C   -   -   -   P   P   -   D   -   -   
6   -   -   -   -   -   -   -   D   -   -   
7   -   -   -   -   -   -   -   D   -   -   
8   -   -   M   S   -   -   -   -   -   -   
9   -   -   -   S   -   -   -   -   -   -   
10  -   -   -   S   -   -   -   -   -   -   
```

## Future Improvements
### Two player over LAN
Allow for two players on separate computers play against each other. Each game instance would send over each player's coordinate choice. The only change needed would be to implement socket communication and modification of some logic to have one of the players host a "server" where all the game logic is handled \(whose turn, if the game is won, etc\).

### GUI Interface
Playing with a text-based map is not fun - it's hard to decipher what each cell is. Introducing a GUI would make the game more user friendly and not require as much effort to play.

### Code Refactoring
The initial goal was to get the game to work with python - there are portions of code that can be moved elsewhere or is redundant \(check all the enums and dictionary mappings - there are way too many of them\). The code can definitely be improved upon.

## Final Thoughts
In the end, the goal was to learn Python - this project was only the tip of the Python iceberg for me. Implementing this game, I learned a few things:
* Python is certainly different than the other languages I'm used to.
* Python is a lot less verbose and Java or C#
* A lot of the nitty-gritty details are abstracted away, this is awesome for the programmer. I can do so much with a lot less code.
* Perhaps it was just my IDE \(PyCharm\), the user experience I had developing Python was not as smooth as developing C# \(Visual Studio\) or Java \(IntelliJ\).

**Find the project [here](https://github.com/JerryBLi/Battleship).**

