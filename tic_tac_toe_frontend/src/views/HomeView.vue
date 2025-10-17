<script setup lang="ts">
import GameBoard from '../components/GameBoard.vue'
import '../assets/theme.css'
</script>

<template>
  <main class="app-shell">
    <div class="card" role="region" aria-label="Tic Tac Toe Game">
      <div class="card-header">
        <h1 class="card-title">
          <svg width="22" height="22" viewBox="0 0 24 24" aria-hidden="true">
            <path fill="currentColor" d="M3 3h2v2H3V3m8 0h2v2h-2V3m8 0h2v2h-2V3M3 11h2v2H3v-2m8 0h2v2h-2v-2m8 0h2v2h-2v-2M3 19h2v2H3v-2m8 0h2v2h-2v-2m8 0h2v2h-2v-2Z"/>
          </svg>
          Ocean Tic Tac Toe
        </h1>
      </div>
      <div class="card-content">
        <GameBoard />
        <p class="footer-note">Play locally in your browser. No data is stored.</p>
      </div>
    </div>
  </main>
</template>
