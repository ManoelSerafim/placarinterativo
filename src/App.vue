<script setup lang="ts">
import { ref, computed } from 'vue'

// Pontuações iniciais, ambas começam em zero (requisito)
const scoreA = ref<number>(0)
const scoreB = ref<number>(0)

// Controla se o jogo já começou (exibição condicional)
const gameStarted = ref<boolean>(false)

// Função para aumentar a pontuação de um jogador
function addScore(player: 'A' | 'B') {
  if (!gameStarted.value) gameStarted.value = true
  if (player === 'A') scoreA.value += 1
  else scoreB.value += 1
}

// Função para diminuir a pontuação; não permite valores negativos
function subScore(player: 'A' | 'B') {
  if (player === 'A') {
    if (scoreA.value > 0) scoreA.value -= 1
  } else {
    if (scoreB.value > 0) scoreB.value -= 1
  }
}

// Reinicia o jogo: zera pontuações e marca como não iniciado
function resetGame() {
  scoreA.value = 0
  scoreB.value = 0
  gameStarted.value = false
}

// Propriedade computada que retorna o status do jogo: quem está vencendo ou empate (requisito)
const statusMessage = computed((): string => {
  if (!gameStarted.value) return 'Jogo ainda não iniciado. Dê a primeira pontuação para começar.'
  if (scoreA.value === scoreB.value) return `Empate — ${scoreA.value} x ${scoreB.value}`
  if (scoreA.value > scoreB.value) return `Jogador A está vencendo — ${scoreA.value} x ${scoreB.value}`
  return `Jogador B está vencendo — ${scoreB.value} x ${scoreA.value}`
})

// Computed adicional para indicar o tipo do status (apenas para estilização)
const statusType = computed((): 'idle' | 'tie' | 'a' | 'b' => {
  if (!gameStarted.value) return 'idle'
  if (scoreA.value === scoreB.value) return 'tie'
  return scoreA.value > scoreB.value ? 'a' : 'b'
})
</script>

<template>
  <div class="app-root">
    <!-- botão de reiniciar fixo no canto superior direito -->
    <button class="reset-icon" @click="resetGame" title="Reiniciar jogo" aria-label="Reiniciar jogo">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
        <path d="M21 12a9 9 0 10-2.1 5.7" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
        <path d="M21 3v6h-6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
      </svg>
    </button>
    <!-- mensagem de status exibida acima do card: não iniciado / empate / vencedor -->
    <div :class="['start-message', statusType]" role="status" aria-live="polite">
      {{ statusMessage }}
    </div>
    <div class="card">
      <h1>Placar Interativo</h1>

  <!-- A mensagem inicial agora é exibida via badge (status) para manter consistência visual -->

      <section class="scoreboard">
      <div class="player-card">
        <h2>Jogador A</h2>
        <p class="score">{{ scoreA }}</p>
        <div class="controls">
          <button class="btn primary" @click="addScore('A')">+1</button>
          <button class="btn neutral" @click="subScore('A')" :disabled="scoreA === 0">-1</button>
        </div>
      </div>

      <div class="player-card">
        <h2>Jogador B</h2>
        <p class="score">{{ scoreB }}</p>
        <div class="controls">
          <button class="btn primary" @click="addScore('B')">+1</button>
          <button class="btn neutral" @click="subScore('B')" :disabled="scoreB === 0">-1</button>
        </div>
      </div>
    </section>

      <!-- status interno removido (agora mostrado acima do card) -->
    </div>
  </div>
</template>

<style scoped>
.app-root {
  min-height: 100vh;
  display: flex;
  flex-direction: column; /* empilha banner e card verticalmente */
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 2rem;
  /* deixar transparente para herdar o gradiente do body (garante que sejam idênticos) */
  background: transparent;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* se o #app estiver usando grid (styles globais), garanta que este wrapper ocupe ambas as colunas */
  grid-column: 1 / -1;
}

.card {
  width: 100%;
  max-width: 780px;
  /* deixar o card transparente para que o gradiente venha do body (idêntico) */
  background: transparent;
  border-radius: 14px;
  padding: 2rem;
  box-shadow: 0 10px 30px rgba(2,6,23,0.18);
  /* leve backdrop para melhorar leitura sobre o gradiente */
  backdrop-filter: blur(6px) saturate(120%);
  border: 1px solid rgba(255,255,255,0.06);
  font-family: system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
  margin: 0 auto;
}

h1 {
  text-align: center;
  margin: 0 0 0.5rem 0;
  font-size: 1.9rem;
  letter-spacing: -0.02em;
  color: #ffffff;
  font-weight: 700;
}

/* nota: a mensagem "jogo ainda não iniciado" agora é exibida via .status-badge (estado idle) */

.scoreboard {
  display: flex;
  gap: 1.25rem;
  justify-content: center;
  margin-bottom: 1.25rem;
}

.player-card {
  border-radius: 12px;
  padding: 1.25rem;
  width: 220px;
  text-align: center;
  /* usar uma placa translúcida para destacar os campos dentro do card */
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.12);
}

.player-card h2 {
  margin: 0;
  font-size: 1.05rem;
  color: #ffffff;
  font-weight: 600;
  letter-spacing: 0.01em;
}

.score {
  font-size: 3rem;
  margin: 0.5rem 0;
  font-weight: 800;
  color: #ffffff;
}

.controls .btn {
  padding: 0.5rem 0.9rem;
  margin: 0 0.25rem;
  font-size: 1rem;
  border: none;
  border-radius: 8px;
  color: white;
  box-shadow: 0 6px 18px rgba(16,24,40,0.06);
  transition: transform .12s ease, box-shadow .12s ease, opacity .12s ease;
  cursor: pointer;
}

.controls .btn.primary {
  /* botao principal é branco sobre o gradiente para garantir contraste */
  background: #ffffff;
  color: #0f172a;
  box-shadow: 0 8px 22px rgba(2,6,23,0.18);
}

.controls .btn.neutral {
  background: rgba(255,255,255,0.14);
  color: #f8fafc;
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255,255,255,0.06);
}

.controls .btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 12px 26px rgba(16,24,40,0.12);
}

.controls .btn:active:not(:disabled) {
  transform: translateY(0);
}

.controls .btn:disabled {
  background: #f3f4f6;
  color: #9ca3af;
  box-shadow: none;
  cursor: not-allowed;
  opacity: 0.95;
}



/* reset icon no canto superior direito */
.reset-icon {
  position: fixed;
  top: 14px;
  right: 14px;
  width: 44px;
  height: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  background: rgba(255,255,255,0.12);
  color: #fff;
  border: 1px solid rgba(255,255,255,0.12);
  box-shadow: 0 8px 28px rgba(2,6,23,0.18);
  cursor: pointer;
  z-index: 60;
  transition: transform .12s ease, box-shadow .12s ease, opacity .12s ease;
}

.reset-icon:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 34px rgba(2,6,23,0.22);
}

.reset-icon:active {
  transform: translateY(0);
}

.status {
  text-align: center;
  margin-top: 1rem;
}

/* mensagem mostrada acima do card quando o jogo ainda não começou */
.start-message {
  display: block;
  margin: 0 auto 1rem auto;
  max-width: 780px;
  text-align: center;
  background: rgba(2,6,23,0.75);
  color: #fff;
  padding: 0.6rem 1rem;
  border-radius: 10px;
  font-weight: 700;
  box-shadow: 0 8px 20px rgba(2,6,23,0.22);
}

/* variantes de status aplicadas à start-message */
.start-message.idle {
  background: rgba(2,6,23,0.75);
  color: #ffffff;
}

.start-message.tie {
  background: rgba(255,255,255,0.18);
  color: #071022;
}

.start-message.a {
  background: rgba(255,255,255,0.12);
  color: #ffffff;
}

.start-message.b {
  background: rgba(255,255,255,0.12);
  color: #042023;
}

.status-badge {
  display: inline-block;
  padding: 0.6rem 1.1rem;
  border-radius: 999px;
  font-weight: 800;
  font-size: 1.05rem;
  color: white;
  box-shadow: 0 10px 30px rgba(2,6,23,0.18);
  letter-spacing: -0.01em;
  transform: translateZ(0);
  transition: transform .12s ease, box-shadow .12s ease, opacity .12s ease;
}

.status-badge.idle {
  /* estado inicial: translúcido escuro para contraste sobre o gradiente */
  background: rgba(2,6,23,0.6);
  color: #ffffff;
  border: 1px solid rgba(255,255,255,0.06);
  box-shadow: 0 8px 28px rgba(2,6,23,0.28);
}

.status-badge.tie {
  background: rgba(255,255,255,0.18);
  color: #071022;
  box-shadow: 0 8px 20px rgba(7,16,34,0.06);
}

.status-badge.a {
  background: rgba(255,255,255,0.12);
  color: #ffffff;
  border: 1px solid rgba(255,255,255,0.12);
  box-shadow: 0 10px 28px rgba(67,56,202,0.12);
}

.status-badge.b {
  background: rgba(255,255,255,0.12);
  color: #042023;
  border: 1px solid rgba(255,255,255,0.12);
  box-shadow: 0 10px 28px rgba(6,182,148,0.10);
}
</style>
