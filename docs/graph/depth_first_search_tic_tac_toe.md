# DFS Tic-Tac-Toe (Minimax Game Tree)

## 1. Overview

DFS Tic-Tac-Toe demonstrates the application of depth-first search to game tree exploration using the Minimax algorithm. It finds the optimal move for a player by recursively exploring all possible game states, assuming both players play optimally.

This is a classic example of game theory and adversarial search algorithms.

## 2. Mathematical Foundation

### 2.1 Game Tree

A game tree represents all possible states and moves:
- **Nodes:** Game states
- **Edges:** Moves/transitions
- **Leaves:** Terminal states (win/lose/draw)

### 2.2 Minimax Theorem

For zero-sum two-player games with perfect information:
$$\max_a \min_b V(a, b) = \min_b \max_a V(a, b) = V^*$$

The optimal value exists and both players can achieve it.

### 2.3 State Space

Tic-Tac-Toe:
- Total positions: 3⁹ = 19,683 (upper bound)
- Legal positions: ~5,478 (accounting for symmetry)
- Terminal states: Much fewer due to early wins

## 3. Minimax Algorithm

### 3.1 Concept

- **Max player (X):** Maximizes score
- **Min player (O):** Minimizes score
- **Scores:** Win = +1, Loss = -1, Draw = 0

### 3.2 Pseudocode

```
MINIMAX(state, is_maximizing):
    if is_terminal(state):
        return evaluate(state)
    
    if is_maximizing:
        best = -∞
        for each move in legal_moves(state):
            new_state = apply_move(state, move)
            score = MINIMAX(new_state, false)
            best = max(best, score)
        return best
    else:
        best = +∞
        for each move in legal_moves(state):
            new_state = apply_move(state, move)
            score = MINIMAX(new_state, true)
            best = min(best, score)
        return best

BEST-MOVE(state, is_maximizing):
    best_move = None
    best_score = -∞ if is_maximizing else +∞
    
    for each move in legal_moves(state):
        new_state = apply_move(state, move)
        score = MINIMAX(new_state, not is_maximizing)
        
        if is_maximizing and score > best_score:
            best_score = score
            best_move = move
        if not is_maximizing and score < best_score:
            best_score = score
            best_move = move
    
    return best_move
```

## 4. Example

```
Board state (X's turn):
 X | O | X
-----------
 O | X | .
-----------
 . | . | O

X evaluates moves at positions 5, 6, 7:

Position 5:         Position 6:         Position 7:
 X | O | X          X | O | X          X | O | X
-----------        -----------        -----------
 O | X | X         O | X | .          O | X | .
-----------        -----------        -----------
 . | . | O          X | . | O          . | X | O
                   (X wins!)

Best move: Position 6 (immediate win)
```

## 5. Alpha-Beta Pruning

### 5.1 Optimization

Avoid exploring branches that cannot affect the outcome:
- **Alpha:** Best score Max can guarantee
- **Beta:** Best score Min can guarantee
- Prune when alpha ≥ beta

### 5.2 Pseudocode

```
ALPHABETA(state, depth, α, β, is_maximizing):
    if depth = 0 or is_terminal(state):
        return evaluate(state)
    
    if is_maximizing:
        value = -∞
        for each move in legal_moves(state):
            value = max(value, ALPHABETA(apply(state, move), depth-1, α, β, false))
            α = max(α, value)
            if α ≥ β:
                break  // β cutoff
        return value
    else:
        value = +∞
        for each move in legal_moves(state):
            value = min(value, ALPHABETA(apply(state, move), depth-1, α, β, true))
            β = min(β, value)
            if β ≤ α:
                break  // α cutoff
        return value
```

## 6. Complexity

| Aspect | Minimax | Alpha-Beta |
|--------|---------|------------|
| Time | O(b^d) | O(b^(d/2)) best case |
| Space | O(d) | O(d) |

Where b = branching factor, d = depth.

For Tic-Tac-Toe: b ≤ 9, d ≤ 9.

## 7. Implementation

```rust
#[derive(Clone, Copy, PartialEq, Eq)]
pub enum Player {
    X,
    O,
}

impl Player {
    fn opponent(&self) -> Player {
        match self {
            Player::X => Player::O,
            Player::O => Player::X,
        }
    }
}

#[derive(Clone, Copy, PartialEq, Eq)]
pub enum Cell {
    Empty,
    X,
    O,
}

#[derive(Clone)]
pub struct TicTacToe {
    board: [[Cell; 3]; 3],
    current_player: Player,
}

impl TicTacToe {
    pub fn new() -> Self {
        TicTacToe {
            board: [[Cell::Empty; 3]; 3],
            current_player: Player::X,
        }
    }

    pub fn make_move(&mut self, row: usize, col: usize) -> bool {
        if self.board[row][col] != Cell::Empty {
            return false;
        }
        
        self.board[row][col] = match self.current_player {
            Player::X => Cell::X,
            Player::O => Cell::O,
        };
        self.current_player = self.current_player.opponent();
        true
    }

    pub fn undo_move(&mut self, row: usize, col: usize) {
        self.board[row][col] = Cell::Empty;
        self.current_player = self.current_player.opponent();
    }

    pub fn check_winner(&self) -> Option<Player> {
        // Check rows
        for row in 0..3 {
            if self.board[row][0] != Cell::Empty
                && self.board[row][0] == self.board[row][1]
                && self.board[row][1] == self.board[row][2]
            {
                return match self.board[row][0] {
                    Cell::X => Some(Player::X),
                    Cell::O => Some(Player::O),
                    Cell::Empty => None,
                };
            }
        }
        
        // Check columns
        for col in 0..3 {
            if self.board[0][col] != Cell::Empty
                && self.board[0][col] == self.board[1][col]
                && self.board[1][col] == self.board[2][col]
            {
                return match self.board[0][col] {
                    Cell::X => Some(Player::X),
                    Cell::O => Some(Player::O),
                    Cell::Empty => None,
                };
            }
        }
        
        // Check diagonals
        if self.board[0][0] != Cell::Empty
            && self.board[0][0] == self.board[1][1]
            && self.board[1][1] == self.board[2][2]
        {
            return match self.board[0][0] {
                Cell::X => Some(Player::X),
                Cell::O => Some(Player::O),
                Cell::Empty => None,
            };
        }
        
        if self.board[0][2] != Cell::Empty
            && self.board[0][2] == self.board[1][1]
            && self.board[1][1] == self.board[2][0]
        {
            return match self.board[0][2] {
                Cell::X => Some(Player::X),
                Cell::O => Some(Player::O),
                Cell::Empty => None,
            };
        }
        
        None
    }

    pub fn is_draw(&self) -> bool {
        self.check_winner().is_none()
            && self.board.iter().all(|row| row.iter().all(|&c| c != Cell::Empty))
    }

    pub fn get_available_moves(&self) -> Vec<(usize, usize)> {
        let mut moves = Vec::new();
        for row in 0..3 {
            for col in 0..3 {
                if self.board[row][col] == Cell::Empty {
                    moves.push((row, col));
                }
            }
        }
        moves
    }
}

/// Minimax evaluation: +1 for X win, -1 for O win, 0 for draw
pub fn minimax(game: &mut TicTacToe, is_maximizing: bool) -> i32 {
    // Check terminal state
    if let Some(winner) = game.check_winner() {
        return match winner {
            Player::X => 1,
            Player::O => -1,
        };
    }
    
    if game.is_draw() {
        return 0;
    }
    
    let moves = game.get_available_moves();
    
    if is_maximizing {
        let mut best = i32::MIN;
        for (row, col) in moves {
            game.make_move(row, col);
            best = best.max(minimax(game, false));
            game.undo_move(row, col);
        }
        best
    } else {
        let mut best = i32::MAX;
        for (row, col) in moves {
            game.make_move(row, col);
            best = best.min(minimax(game, true));
            game.undo_move(row, col);
        }
        best
    }
}

/// Find best move using minimax
pub fn best_move(game: &mut TicTacToe) -> Option<(usize, usize)> {
    let moves = game.get_available_moves();
    if moves.is_empty() {
        return None;
    }
    
    let is_maximizing = game.current_player == Player::X;
    let mut best_move = None;
    let mut best_score = if is_maximizing { i32::MIN } else { i32::MAX };
    
    for (row, col) in moves {
        game.make_move(row, col);
        let score = minimax(game, !is_maximizing);
        game.undo_move(row, col);
        
        if is_maximizing && score > best_score {
            best_score = score;
            best_move = Some((row, col));
        } else if !is_maximizing && score < best_score {
            best_score = score;
            best_move = Some((row, col));
        }
    }
    
    best_move
}

/// Minimax with alpha-beta pruning
pub fn minimax_alphabeta(
    game: &mut TicTacToe,
    alpha: i32,
    beta: i32,
    is_maximizing: bool,
) -> i32 {
    if let Some(winner) = game.check_winner() {
        return match winner {
            Player::X => 1,
            Player::O => -1,
        };
    }
    
    if game.is_draw() {
        return 0;
    }
    
    let moves = game.get_available_moves();
    let mut alpha = alpha;
    let mut beta = beta;
    
    if is_maximizing {
        let mut best = i32::MIN;
        for (row, col) in moves {
            game.make_move(row, col);
            best = best.max(minimax_alphabeta(game, alpha, beta, false));
            game.undo_move(row, col);
            
            alpha = alpha.max(best);
            if beta <= alpha {
                break;  // Beta cutoff
            }
        }
        best
    } else {
        let mut best = i32::MAX;
        for (row, col) in moves {
            game.make_move(row, col);
            best = best.min(minimax_alphabeta(game, alpha, beta, true));
            game.undo_move(row, col);
            
            beta = beta.min(best);
            if beta <= alpha {
                break;  // Alpha cutoff
            }
        }
        best
    }
}
```

## 8. Tic-Tac-Toe Optimal Play

With perfect play from both sides:
- First player (X) can always force at least a draw
- Second player (O) can always force at least a draw
- **Result: Always a draw**

## 9. Extensions

### 9.1 Larger Boards

For Connect-4, Chess, Go:
- Use evaluation functions instead of full search
- Apply depth limits
- Use transposition tables

### 9.2 Iterative Deepening

```rust
fn iterative_deepening_minimax(game: &mut TicTacToe, max_depth: usize) -> Option<(usize, usize)> {
    let mut best = None;
    for depth in 1..=max_depth {
        best = best_move_depth_limited(game, depth);
    }
    best
}
```

### 9.3 Move Ordering

Search likely-best moves first to improve alpha-beta pruning effectiveness.

## 10. Applications

1. **Board games:** Chess, Checkers, Go (with modifications)
2. **Card games:** Poker (with probability)
3. **AI opponents:** Video game AI
4. **Decision making:** Any adversarial scenario
5. **Theorem proving:** Game-theoretic proofs

## 11. Related Algorithms

| Algorithm | Use Case |
|-----------|----------|
| Minimax | Perfect information games |
| Alpha-Beta | Optimized minimax |
| Monte Carlo Tree Search | Large state spaces (Go) |
| Expectimax | Games with chance |
| Negamax | Simplified minimax coding |

## 12. Edge Cases

| Case | Handling |
|------|----------|
| Empty board | Return any corner (optimal) |
| One move left | Direct evaluation |
| Already won/lost | Return terminal value |
| Draw imminent | Return 0 |

## 13. Common Pitfalls

1. **Forgetting to undo:** Must restore state after recursive call
2. **Wrong player evaluation:** Match maximizing with correct player
3. **Depth limits:** For complex games, limit search depth
4. **Symmetric states:** Can cache to avoid recomputation

## 14. References

- Shannon, C. (1950). "Programming a Computer for Playing Chess"
- Knuth, D. E.; Moore, R. W. (1975). "An Analysis of Alpha-Beta Pruning"
- Russell, S.; Norvig, P. "Artificial Intelligence: A Modern Approach", Chapter 5
