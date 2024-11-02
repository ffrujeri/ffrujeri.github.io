---
layout: archive
title: 'Board Games DSL'
date: 2024-10-15
permalink: /posts/2024/10/board-games-dsl/
header:
  teaser: "We try to desing a DSL for board games."
tags:
  - board games
  - dsl
---  

## Library vs DSL

The difference between a library and a domain-specific language (DSL) lies primarily in their scope, purpose, and level of abstraction.

1. Definition and Purpose:
- Library:
    - A library is a collection of pre-written code (functions, classes, etc.) that developers can reuse in their applications. It provides specific functionality, such as handling mathematical operations, working with files, or interacting with APIs. 
    
    - It is used within an existing general-purpose programming language (e.g., Python, Java, etc.), and the user writes code using that language to interact with the library.

- Domain-Specific Language (DSL):
    - A DSL is a specialized programming language designed to handle tasks within a specific domain or field. It is crafted to express concepts in that domain more naturally or succinctly than a general-purpose language (GPL) can.

    - A DSL is often more limited than general-purpose languages and tailored for domain-specific problems (e.g., SQL for databases, HTML for web design, or Verilog for hardware design).

2. Level of Abstraction:
- Library:
    - Works within the framework of a general-purpose language and follows its rules and syntax. It provides reusable components (functions, classes) to avoid rewriting code but relies on the core language for structure and control flow. For example, NumPy in Python is a library that provides mathematical tools, but you still write Python code to use it.
    
- DSL:
    - Operates as a language in its own right, with its own syntax and semantics. It is often designed to be more concise and expressive within its domain, potentially hiding complexity from the user. For example, SQL is a DSL specifically designed for querying databases. It's distinct from general-purpose languages like Python or Java.

3. Flexibility:
- Library:
    - More flexible, since it can be used to solve a wide range of problems beyond the specific domain it targets. Developers can combine it with other libraries to create complex applications. Example: You can combine NumPy (for math) and Flask (for web development) in the same Python program.
- DSL:
    - More focused and limited in scope. It often targets one type of problem or domain, and typically cannot be easily combined with general-purpose logic (though it can be embedded in or called from a general-purpose language). Example: SQL is highly effective for querying databases but is not suitable for general-purpose tasks like file I/O or complex application logic.

- 4. Examples:
- Library:
    - NumPy (Python) – for numerical computations.
    - Pandas (Python) – for data manipulation.
    - jQuery (JavaScript) – for simplifying DOM manipulation in web development.
- DSL:
    - SQL – for querying databases.
    - HTML – for web markup.
    - Regex (Regular Expressions) – for pattern matching in strings.
    - MATLAB (sometimes considered a DSL for numerical computations).

- Summary:
    - Library: A set of reusable tools within a general-purpose language. It provides functionality (functions, classes, etc.) but requires the use of the general-purpose language's syntax and constructs to work.
    - DSL: A specialized language with its own syntax and rules, designed for solving specific types of problems within a particular domain.

## Board Games DSL
There are domain-specific languages (DSLs) designed specifically for describing board games, both in terms of their rules and mechanics. These DSLs allow for a formal and structured way to define the components, actions, and flow of a board game. Here are a few notable DSLs or frameworks that can be used for describing board games

1. GDL (Game Description Language)
- Purpose: GDL was developed to describe the rules of general games (including board games), particularly for AI research and general game-playing systems.

- Scope: It can describe any game that is finite, deterministic, and has perfect information (like chess or tic-tac-toe). The language is designed to model game mechanics, rules, players' roles, and possible moves.

- Applications: GDL is commonly used in AI to create general game-playing systems where the AI can play any game as long as it’s described in GDL.

```prolog
role(player1).
role(player2).

init(cell(1, 1, blank)).
init(cell(1, 2, blank)).
...
legal(player1, mark(X, Y)).
next(cell(X, Y, player1)) :- does(player1, mark(X, Y)).
```

This snippet shows how initial board states and legal moves could be described in GDL.

2. Ludi
- Purpose: Ludi is a system for evolving and generating board games using an AI-driven approach. The core idea behind Ludi is not just to describe existing games but also to create new games based on formal descriptions.

- Scope: It provides a formal grammar for defining board games (including components like pieces, players, board configuration, goals, etc.). Ludi focuses heavily on strategy games and can be used to describe games similar to chess, checkers, and other abstract games.

- Example: Ludi provides ways to describe games like:
    - The type of board (e.g., grid, hexagonal, etc.).
    - Movement rules for different types of pieces.
    - Victory conditions and scoring rules.

3. VGDL (Video Game Description Language)
- Purpose: VGDL is primarily designed for describing video games, but it has been used to define board games as well, especially simple ones or games that translate easily into video game form (like puzzles or grid-based games).

- Scope: It uses a declarative language to specify game entities, rules, and the physics of interaction. Though not specialized purely for board games, VGDL can handle simpler board game structures and rules.

```python
Avatar > MovingAvatar 
Goal > Immovable

InteractionSet:
  MovingAvatar Goal > win
  MovingAvatar Wall > stepBack
```

4. Machinations Framework
- Purpose: Machinations is a visual DSL designed for modeling and analyzing game economies and mechanics, particularly for board games and video games. It’s not a textual DSL like GDL, but rather a diagrammatic language that helps designers visually map out the interactions and dynamics of game systems.

- Scope: It's used to describe the flow of resources, player actions, and game state changes, especially in games with complex economic or strategic layers (e.g., managing resources, earning points, or controlling territories).

Example: In Machinations, you might create diagrams that show how player actions lead to the gain or loss of resources (e.g., health, money, or victory points).

5. Taru Framework (Board Game DSL by Daisuke Ishii)
Purpose: Taru is a lesser-known DSL created to describe traditional board games (like chess, go, and shogi). It focuses on defining components such as pieces, boards, and rules for moving pieces.
Scope: It is more niche compared to GDL and VGDL but well-suited for traditional turn-based board games.
Example: Taru provides a structured way to define game rules and pieces, though details on its usage are less widespread.

6. PDDL (Planning Domain Definition Language) for Turn-Based Games
Purpose: PDDL is a general-purpose DSL for describing planning problems in AI. While not originally designed for board games, it has been adapted in research to model turn-based games, especially those involving strategic moves and decisions.
Scope: PDDL describes states, actions, and outcomes, which can be adapted to represent board game dynamics like movement, capture, or resource allocation.
Example: A simple PDDL problem might describe a player’s options in a chess-like game, such as moving a piece or capturing another piece.

Summary of DSLs for Board Games:
GDL: Well-suited for AI-driven game playing and perfect information board games.
Ludi: Specialized in generating and evolving new board games.
VGDL: Primarily for video games, but can be adapted for simple board games.
Machinations: Visual modeling for complex game mechanics, including resource flows.
Taru Framework: Designed specifically for traditional turn-based board games.
PDDL: Can be adapted for modeling strategic board games with planning elements.

## Designing a Board Game DSL
Designing a Domain-Specific Language (DSL) for board games is a challenging but rewarding task. The process requires careful planning, as the DSL must balance expressiveness, usability, and domain specificity. Here's a step-by-step guide to help you get started with the design of your board game DSL:

1. Define the Scope of Your DSL
    Start by deciding what your DSL will focus on within the board game domain. Some key questions to ask include:
    - Which types of board games? Will your DSL focus on abstract strategy games (like chess or Go), resource management games (like Settlers of Catan), or more complex genres (like deck-building or role-playing games)?

    - What game elements will your DSL model? Consider the following:
        - Boards: Grids, hexagonal tiles, card decks, etc.
        - Pieces: Players, tokens, cards, resources, etc.
        - Player Actions: Moves, card draws, resource exchanges, etc.
        - Game State: Score tracking, win conditions, and dynamic elements.

    - Who is the target audience? Game designers? Game developers? AI researchers?
    - Defining a clear domain boundary will help you narrow the features your DSL must support and keep its scope manageable.

2. Identify the Core Abstractions
Next, identify the key components and abstractions that your DSL needs to support. These will form the foundation of your language and should match the essential mechanics of a board game. Common abstractions include:

Board: Define whether it is a grid, hexagonal, graph-based, or even card-based. For example, how should a 2D or 3D board be represented?

```python
board Grid(8, 8)
```

Pieces: Define game pieces, such as pawns, tokens, cards, etc. Think about attributes (e.g., name, type, position).

```python
piece Pawn(player1) at (0, 0)
```

Actions: Define actions that players can take, such as moving a piece, drawing a card, or spending a resource. Actions might need to specify conditions (e.g., legal moves) and outcomes (e.g., move a pawn to a new position).

```python
action Move(Pawn, (x, y), (newX, newY))
```

Players: Define how players interact with the game, including roles, turn order, and decisions.

```python
player(Player1, Player2)
```

Rules and Win Conditions: Describe how to enforce rules (e.g., valid moves, piece interactions) and define end-game conditions.

```python
winCondition Player1 captures all pawns
```

Game State: Represent how the game state is updated after each action, including scores, inventory (cards, resources), and dynamic board changes.

```python
state(score Player1 0)
```

3. Choose the Type of DSL: Embedded or Standalone
There are two main ways to design a DSL:

Embedded DSL: This is embedded inside an existing general-purpose language (e.g., Python, JavaScript). It leverages the syntax and features of the host language.

Pros: Easier to implement, can reuse existing language tools (parsers, debuggers), and integrates smoothly with other code.
Cons: Limited by the syntax and rules of the host language.
Example: A DSL embedded in Python:

```python
board = Grid(8, 8)
piece = Pawn(player1, position=(0, 0))
move(piece, (0, 1))
```

Standalone DSL: This has its own syntax, semantics, and potentially its own interpreter or compiler. It gives more control over the design but requires building a parser, interpreter, or compiler from scratch.

Pros: Full control over syntax and semantics, can be more expressive and specialized.
Cons: Harder to implement, requires more tooling.
Example: A standalone DSL might look like:

```text
board 8x8
player1 Pawn at (0, 0)
move player1 Pawn to (0, 1)
```

4. Design the Syntax and Semantics
Now that you know your core abstractions, think about how to express them syntactically. Your syntax should be:

Concise: Minimize verbosity to make it easy to write and understand.
Expressive: Allow users to define the essential aspects of a board game without unnecessary complexity.
Consistent: Keep similar concepts structured similarly (e.g., defining a piece should look similar to defining a player).
Consider whether your syntax will be more:

Declarative (describe "what" the game components and rules are):
text

```text
board 8x8
pawn player1 at (0, 0)
pawn player2 at (7, 7)
```

Imperative (describe "how" the game actions unfold):

```text
move player1 pawn from (0, 0) to (0, 1)
```

ou also need to define the semantics (the meaning behind the syntax). For instance, if a user writes move piece from (x1, y1) to (x2, y2), you need to define what happens behind the scenes (e.g., update the game state, check for legal moves).

5. Define the Execution Model
Consider how your DSL will be executed or interpreted. Will it:

Generate code in another language? (e.g., compile DSL into Python or JavaScript).
Be interpreted directly? (e.g., interpret DSL commands at runtime, like SQL).
Be used to describe data that an engine processes? (e.g., defining game rules that a game engine reads and processes).
If it's interpreted, think about building an interpreter or parser for your DSL. If compiled, consider creating a code generator to convert DSL instructions into another language.

6. Implement a Parser and Interpreter/Compiler
If you’re building a Standalone DSL, you'll need to implement a parser to break down your DSL syntax into an Abstract Syntax Tree (AST) or intermediate representation (IR). Tools like ANTLR, PLY, or Parboiled can help with building parsers.
If your DSL is embedded within another language, you can skip this step by directly using the host language’s syntax.
For execution, create an interpreter that reads the parsed DSL and executes actions in your game engine or system.

7. Test with Real Board Games
Once your DSL is designed, test it by modeling existing board games. Start with simpler games (like Tic-Tac-Toe, Chess, or Go) and gradually move to more complex ones (like Monopoly or Settlers of Catan). This will help ensure that your DSL is expressive and flexible enough to handle a wide variety of games.

8. Iterate and Improve
Refactor the syntax based on usability.
Add support for additional features, such as multi-player turns, randomness (dice rolls), or resource management.
Gather feedback from other game designers or developers and refine the language to suit their needs.

Example of a Simple Board Game DSL
Here’s a simple conceptual example for a DSL describing a basic turn-based game like Tic-Tac-Toe:

```text
game TicTacToe

board Grid(3, 3)

player Player1
player Player2

piece X by Player1
piece O by Player2

action Move(player, (x, y)) if board[x, y] is empty
win Player1 if 3 X in a row
win Player2 if 3 O in a row
```

In this example:

The board is defined as a 3x3 grid.
Players and pieces are defined.
Moves and win conditions are declared.
This is a starting point, but the DSL can be extended to handle more complex games.

## References
-