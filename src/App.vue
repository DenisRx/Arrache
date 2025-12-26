<script setup>
import {computed, onMounted, ref, watch} from 'vue';

// --- 1. DATA ---
// Import des données brutes (sans ID)
import sourceGames from './data/games.json';

// On génère les ID dynamiquement (Index + 1) pour gérer l'unicité
const allGames = sourceGames.map((game, index) => ({
  ...game,
  id: index + 1,
}));

// --- 2. STATE ---
const history = ref([]);
const isGameOver = ref(false);

// --- 3. PERSISTENCE LOGIC ---
const STORAGE_KEY = 'arrache_app_v1_history';
const GAMEOVER_KEY = 'arrache_app_v1_gameover';

onMounted(() => {
  document.title = 'Arraché 🍻';

  // Restauration Historique
  const savedHistory = localStorage.getItem(STORAGE_KEY);
  if (savedHistory) {
    history.value = JSON.parse(savedHistory);
  }

  // Restauration État Fin de partie
  const savedGameOver = localStorage.getItem(GAMEOVER_KEY);
  if (savedGameOver) {
    isGameOver.value = JSON.parse(savedGameOver);
  }
});

// Sauvegardes automatiques
watch(
  history,
  (newVal) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(newVal));
  },
  { deep: true },
);

watch(isGameOver, (newVal) => {
  localStorage.setItem(GAMEOVER_KEY, JSON.stringify(newVal));
});

// --- 4. COMPUTED PROPERTIES ---

const currentGame = computed(() => {
  if (history.value.length === 0) return null;
  const lastId = history.value[history.value.length - 1];
  return allGames.find((g) => g.id === lastId);
});

const availableGames = computed(() => {
  return allGames.filter((game) => !history.value.includes(game.id));
});

const playedCount = computed(() => history.value.length);
const totalCount = allGames.length;

const isFinished = computed(() => isGameOver.value);

// --- 5. METHODS ---
const handleButtonClick = () => {
  // Cas 1 : Tirer un nouveau jeu
  if (availableGames.value.length > 0) {
    const randomIndex = Math.floor(Math.random() * availableGames.value.length);
    const selectedGame = availableGames.value[randomIndex];
    history.value.push(selectedGame.id);
  }
  // Cas 2 : Plus de jeu dispo, on valide la fin de soirée
  else if (availableGames.value.length === 0 && isGameOver.value === false) {
    isGameOver.value = true;
  }
  // Cas 3 : La soirée est finie, le bouton sert à recommencer
  else {
    resetGame();
  }
};

const resetClicked = () => {
  if (confirm('Es-tu sûr de vouloir recommencer ?')) {
    resetGame();
  }
}

const resetGame = () => {
  history.value = [];
  isGameOver.value = false;
  localStorage.removeItem(STORAGE_KEY);
  localStorage.removeItem(GAMEOVER_KEY);
};
</script>

<template>
  <div
    class="min-h-screen bg-slate-900 text-white flex flex-col items-center justify-between p-6 font-sans"
  >
    <header
      class="w-full flex justify-between items-center text-slate-400 font-bold text-sm uppercase tracking-widest"
    >
      <span class="text-pink-500 tracking-tighter text-lg">ARRACHÉ</span>
      <span class="bg-slate-800 px-3 py-1 rounded-full text-xs">
        {{ playedCount }} / {{ totalCount }}
      </span>
    </header>

    <main
      class="grow flex flex-col justify-center items-center w-full max-w-md text-center"
    >
      <div v-if="isFinished">
        <h1 class="text-4xl font-bold text-green-400 mb-4">Jeu terminé !</h1>
        <p class="text-slate-300">
          Vous avez épuisé tous les jeux.<br />Pas encore arraché ? Recommence !
          😈
        </p>
      </div>

      <div
        v-else-if="currentGame"
        class="bg-slate-800 p-8 rounded-2xl shadow-xl w-full border border-slate-700 transition-all transform duration-300"
      >
        <h2 class="text-3xl font-extrabold text-pink-500 mb-6 uppercase">
          {{ currentGame.title }}
        </h2>
        <p class="text-lg text-slate-200 leading-relaxed font-medium">
          {{ currentGame.instructions }}
        </p>
      </div>

      <div v-else class="text-slate-500 flex flex-col items-center gap-4">
        <div class="text-6xl">🍺</div>
        <p class="text-xl font-light">Prêts à vous faire arracher ?</p>
      </div>
    </main>

    <footer class="w-full max-w-md pb-8">
      <button
        @click="handleButtonClick"
        class="w-full bg-linear-to-r from-purple-600 to-pink-600 hover:from-purple-500 hover:to-pink-500 active:scale-95 disabled:opacity-50 disabled:grayscale disabled:cursor-not-allowed text-white font-bold text-xl py-6 rounded-2xl shadow-lg transition-all duration-200"
      >
        <span v-if="!currentGame">Lancer la soirée</span>
        <span v-else-if="availableGames.length > 0">Jeu Suivant</span>
        <span v-else-if="availableGames.length === 0 && !isFinished"
          >Terminer le jeu</span
        >
        <span v-else>Rejouer</span>
      </button>

      <button
        v-if="history.length > 0 && !isFinished"
        @click="resetClicked"
        class="block mx-auto mt-6 text-xs text-slate-600 hover:text-slate-400 transition-colors"
      >
        Reset
      </button>
    </footer>
  </div>
</template>
