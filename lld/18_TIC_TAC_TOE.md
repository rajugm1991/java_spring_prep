# Tic-Tac-Toe Game

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Analysis](#requirements-analysis)
3. [Class Design](#class-design)
4. [Game Logic](#game-logic)
5. [Complete Implementation](#complete-implementation)

---

## Problem Statement

Design a Tic-Tac-Toe game that can:
- Handle two players (X and O)
- Validate moves
- Check for win conditions
- Detect draw
- Support game reset

---

## Requirements Analysis

### Functional Requirements

1. **Game Board**
   - 3x3 grid
   - Track cell states (X, O, Empty)

2. **Game Logic**
   - Alternate turns
   - Validate moves
   - Check win conditions
   - Detect draw

3. **Player Management**
   - Two players
   - Current player tracking

---

## Class Design

### Core Classes

```java
// Cell State
public enum CellState {
    EMPTY,
    X,
    O
}

// Player
public enum Player {
    X,
    O
}

// Game Board
public class Board {
    private static final int SIZE = 3;
    private CellState[][] cells;
    
    public Board() {
        this.cells = new CellState[SIZE][SIZE];
        initializeBoard();
    }
    
    private void initializeBoard() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                cells[i][j] = CellState.EMPTY;
            }
        }
    }
    
    public boolean makeMove(int row, int col, Player player) {
        if (!isValidMove(row, col)) {
            return false;
        }
        
        cells[row][col] = player == Player.X ? CellState.X : CellState.O;
        return true;
    }
    
    public boolean isValidMove(int row, int col) {
        return row >= 0 && row < SIZE && 
               col >= 0 && col < SIZE && 
               cells[row][col] == CellState.EMPTY;
    }
    
    public boolean isFull() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (cells[i][j] == CellState.EMPTY) {
                    return false;
                }
            }
        }
        return true;
    }
    
    public boolean hasWinner() {
        return checkRows() || checkColumns() || checkDiagonals();
    }
    
    private boolean checkRows() {
        for (int i = 0; i < SIZE; i++) {
            if (cells[i][0] != CellState.EMPTY &&
                cells[i][0] == cells[i][1] &&
                cells[i][1] == cells[i][2]) {
                return true;
            }
        }
        return false;
    }
    
    private boolean checkColumns() {
        for (int j = 0; j < SIZE; j++) {
            if (cells[0][j] != CellState.EMPTY &&
                cells[0][j] == cells[1][j] &&
                cells[1][j] == cells[2][j]) {
                return true;
            }
        }
        return false;
    }
    
    private boolean checkDiagonals() {
        // Main diagonal
        if (cells[0][0] != CellState.EMPTY &&
            cells[0][0] == cells[1][1] &&
            cells[1][1] == cells[2][2]) {
            return true;
        }
        
        // Anti-diagonal
        if (cells[0][2] != CellState.EMPTY &&
            cells[0][2] == cells[1][1] &&
            cells[1][1] == cells[2][0]) {
            return true;
        }
        
        return false;
    }
    
    public Player getWinner() {
        if (!hasWinner()) {
            return null;
        }
        
        // Check rows
        for (int i = 0; i < SIZE; i++) {
            if (cells[i][0] != CellState.EMPTY &&
                cells[i][0] == cells[i][1] &&
                cells[i][1] == cells[i][2]) {
                return cells[i][0] == CellState.X ? Player.X : Player.O;
            }
        }
        
        // Check columns
        for (int j = 0; j < SIZE; j++) {
            if (cells[0][j] != CellState.EMPTY &&
                cells[0][j] == cells[1][j] &&
                cells[1][j] == cells[2][j]) {
                return cells[0][j] == CellState.X ? Player.X : Player.O;
            }
        }
        
        // Check diagonals
        if (cells[0][0] != CellState.EMPTY &&
            cells[0][0] == cells[1][1] &&
            cells[1][1] == cells[2][2]) {
            return cells[0][0] == CellState.X ? Player.X : Player.O;
        }
        
        if (cells[0][2] != CellState.EMPTY &&
            cells[0][2] == cells[1][1] &&
            cells[1][1] == cells[2][0]) {
            return cells[0][2] == CellState.X ? Player.X : Player.O;
        }
        
        return null;
    }
}

// Game Status
public enum GameStatus {
    IN_PROGRESS,
    X_WON,
    O_WON,
    DRAW
}

// Tic-Tac-Toe Game
public class TicTacToe {
    private Board board;
    private Player currentPlayer;
    private GameStatus status;
    
    public TicTacToe() {
        this.board = new Board();
        this.currentPlayer = Player.X;
        this.status = GameStatus.IN_PROGRESS;
    }
    
    public boolean makeMove(int row, int col) {
        if (status != GameStatus.IN_PROGRESS) {
            return false;
        }
        
        if (!board.makeMove(row, col, currentPlayer)) {
            return false;
        }
        
        // Check for winner
        if (board.hasWinner()) {
            status = currentPlayer == Player.X ? GameStatus.X_WON : GameStatus.O_WON;
        } else if (board.isFull()) {
            status = GameStatus.DRAW;
        } else {
            // Switch player
            currentPlayer = currentPlayer == Player.X ? Player.O : Player.X;
        }
        
        return true;
    }
    
    public void reset() {
        this.board = new Board();
        this.currentPlayer = Player.X;
        this.status = GameStatus.IN_PROGRESS;
    }
    
    public Player getCurrentPlayer() {
        return currentPlayer;
    }
    
    public GameStatus getStatus() {
        return status;
    }
    
    public Board getBoard() {
        return board;
    }
}
```

---

## Game Logic Flow

```
[Start Game]
     │
     ▼
[Player X Turn]
     │
     ▼
[Make Move]
     │
     ├─── Invalid ───> [Retry]
     │
     └─── Valid ───> [Check Win]
                         │
              ┌───────────┼───────────┐
              │           │           │
            Win?        Draw?      Continue
              │           │           │
              ▼           ▼           ▼
         [Game Over]  [Game Over]  [Switch Player]
              │           │           │
              │           │           └──> [Player O Turn]
              │           │
              └───────────┘
```

---

## Summary

**Key Components:**
1. ✅ **Board:** 3x3 grid management
2. ✅ **Game Logic:** Win detection, draw detection
3. ✅ **Player Management:** Turn alternation
4. ✅ **Game Status:** Track game state

**Next Steps:**
- Learn [Design Principles](./19_DESIGN_PRINCIPLES_DEEP_DIVE.md)
- Review [Class Design](./04_CLASS_DESIGN.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

---

**References:**
- [Class Design](./04_CLASS_DESIGN.md)
- [OOP Principles](./02_OOP_PRINCIPLES_FOR_LLD.md)

