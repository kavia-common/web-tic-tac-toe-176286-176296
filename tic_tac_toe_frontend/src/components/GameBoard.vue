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

const statusLabel = computed(() => {
  if (state.winner) return `Winner: ${state.winner}`
  if (state.isDraw) return 'Draw'
  return `Turn: ${state.currentPlayer}`
})

const statusClass = computed(() => {
  if (state.winner) return 'status win'
  if (state.isDraw) return 'status draw'
  return 'status'
})

function ariaForCell(i: number): string {
  const v = state.cells[i]
  const pos = `row ${Math.floor(i / 3) + 1} column ${ (i % 3) + 1 }`
  return v ? `Cell ${pos} with ${v}` : `Empty cell ${pos}, press to place ${state.currentPlayer}`
}

function isWinningCell(i: number): boolean {
  return state.winningLine.includes(i)
}
</script>

<template>
  <div>
    <div :class="statusClass" role="status" aria-live="polite">
      <span class="dot" aria-hidden="true"></span>
      <span class="label">{{ statusLabel }}</span>
      <span class="subtle" v-if="!state.winner && !state.isDraw">Players: X (blue) · O (amber)</span>
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
        <span class="mark" v-if="cell">{{ cell }}</span>
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
/* The styles rely on variables from theme.css. Component adds minimal overrides if needed. */
</style>
