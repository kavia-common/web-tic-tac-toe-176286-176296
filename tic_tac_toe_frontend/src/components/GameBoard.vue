<script setup lang="ts">
import { computed, reactive } from 'vue'

type Player = 'X' | 'O'
type Cell = Player | null

// Board state and minimal in-component state management
const state = reactive({
  cells: Array<Cell>(9).fill(null),
  currentPlayer: 'X' as Player,
  winner: null as Player | null,
  winningLine: [] as number[],
  isDraw: false,
})

const lines: number[][] = [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8],
  [0, 4, 8],
  [2, 4, 6],
]

// Map players to chess Unicode symbols
const chessIconFor = (p: Player): string => (p === 'X' ? '♞' : '♛') // Knight for X, Queen for O

// PUBLIC_INTERFACE
function resetGame(): void {
  /** Reset the game to an initial clean state. */
  state.cells = Array<Cell>(9).fill(null)
  state.currentPlayer = 'X'
  state.winner = null
  state.winningLine = []
  state.isDraw = false
}

function checkWinner(): void {
  for (const [a, b, c] of lines) {
    const va = state.cells[a]
    if (va && va === state.cells[b] && va === state.cells[c]) {
      state.winner = va
      state.winningLine = [a, b, c]
      state.isDraw = false
      return
    }
  }
  // If no winner and all cells filled => draw
  if (state.cells.every((v) => v !== null)) {
    state.isDraw = true
  }
}

function makeMove(index: number): void {
  if (state.winner || state.isDraw) return
  if (state.cells[index] !== null) return

  state.cells[index] = state.currentPlayer
  checkWinner()
  if (!state.winner && !state.isDraw) {
    state.currentPlayer = state.currentPlayer === 'X' ? 'O' : 'X'
  }
}

/* status text is directly constructed in template with visible icons to avoid redundancy */

const statusClass = computed(() => {
  if (state.winner) return 'status win'
  if (state.isDraw) return 'status draw'
  return 'status'
})

function ariaForCell(i: number): string {
  const v = state.cells[i]
  const pos = `row ${Math.floor(i / 3) + 1} column ${ (i % 3) + 1 }`
  if (v) {
    const piece = v === 'X' ? 'knight' : 'queen'
    return `Cell ${pos} with ${v}, ${piece}`
  }
  const pieceToPlace = state.currentPlayer === 'X' ? 'knight' : 'queen'
  return `Empty cell ${pos}, press to place ${state.currentPlayer}, ${pieceToPlace}`
}

function isWinningCell(i: number): boolean {
  return state.winningLine.includes(i)
}
</script>

<template>
  <div>
    <div :class="statusClass" role="status" aria-live="polite">
      <span class="dot" aria-hidden="true"></span>
      <span class="label">
        <!-- Visual icon for status -->
        <template v-if="state.winner">
          <span
            v-if="state.winner === 'X'"
            class="icon mark x"
            aria-hidden="true"
          >{{ chessIconFor('X') }}</span>
          <span
            v-else
            class="icon mark o"
            aria-hidden="true"
          >{{ chessIconFor('O') }}</span>
          <span class="sr-only">Winner:</span>
          Winner
        </template>
        <template v-else-if="state.isDraw">
          Draw
        </template>
        <template v-else>
          <span class="sr-only">Turn:</span>
          <span
            v-if="state.currentPlayer === 'X'"
            class="icon mark x"
            aria-hidden="true"
          >{{ chessIconFor('X') }}</span>
          <span
            v-else
            class="icon mark o"
            aria-hidden="true"
          >{{ chessIconFor('O') }}</span>
          Turn
        </template>
      </span>
      <span class="subtle" v-if="!state.winner && !state.isDraw">
        Players: <span class="icon-inline x" aria-hidden="true">♞</span> (blue) ·
        <span class="icon-inline o" aria-hidden="true">♛</span> (amber)
        <span class="sr-only">X is knight, O is queen</span>
      </span>
    </div>

    <div class="board" role="grid" aria-label="Tic Tac Toe Board">
      <button
        v-for="(cell, i) in state.cells"
        :key="i"
        class="cell"
        :class="[{ disabled: !!cell || !!state.winner || state.isDraw, win: isWinningCell(i), o: cell === 'O' }]"
        role="gridcell"
        type="button"
        :aria-pressed="cell !== null"
        :aria-label="ariaForCell(i)"
        @click="makeMove(i)"
        @keyup.enter="makeMove(i)"
      >
        <span
          v-if="cell"
          class="mark"
          :class="[{ x: cell === 'X', o: cell === 'O' }]"
          aria-hidden="true"
        >{{ chessIconFor(cell) }}</span>
        <span v-else class="sr-only">Empty</span>
      </button>
    </div>

    <div class="controls">
      <button class="btn btn-primary" type="button" @click="resetGame" aria-label="Start a new game">
        New Game
      </button>
      <button
        v-if="state.winner || state.isDraw"
        class="btn"
        type="button"
        @click="resetGame"
        aria-label="Play again"
      >
        Play Again
      </button>
    </div>
  </div>
</template>

<style scoped>
/* Accessibility helper for screen readers */
.sr-only {
  position: absolute !important;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap; /* added line */
  border: 0;
}

/* Icon styles inherit existing theme sizing */
.icon,
.mark {
  font-size: 1em; /* keep consistent with cell font-size scaling */
  line-height: 1;
  display: inline-block;
}

/* Colors follow theme: X (knight) uses primary blue, O (queen) uses secondary amber */
.x {
  color: var(--ocean-primary);
}
.o {
  color: var(--ocean-secondary);
}

/* Inline icons for small status text */
.icon-inline {
  font-weight: 800;
}

/* Ensure board marks remain crisp */
.cell .mark {
  transform: translateZ(0);
  will-change: transform, color;
}

/* Existing .cell.o .mark color rule is now complemented by .o class above */
</style>
