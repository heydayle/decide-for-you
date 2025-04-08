<template>
  <div class="mystical-container">
    <div class="mystical-background"></div>
      <div class="fixed top-10 text-center z-10">
        <h1 class="title !mb-2">The Devaloka's Guidance</h1>
        <p class="mt-0 text-2xl">Seek wisdom from the divine. The cards hold your answers.</p>
        <transition name="fade">
          <div v-if="!shuffling && !cardsDealt && !selectedCard" class="intro-message !mt-4">
            <button @click="shuffleAndDeal" class="mystical-button">
              <span class="button-text">Consult the Oracle</span>
            </button>
          </div>
        </transition>
      </div>
      <div class="fixed bottom-0 z-10">
        <a href="https://thinh.io.vn">@Thinh Le</a>
      </div>


    <div class="cards-container" :class="{ dealt: cardsDealt }">
      <transition-group name="card" tag="div" class="cards-layout">
        <div
          v-for="(card, index) in cards"
          :key="card.id"
          class="card"
          :class="{
            'card-back': !card.revealed,
            'card-front': card.revealed,
            selected: selectedCardIndex === index,
            unselected: selectedCardIndex !== null && selectedCardIndex !== index,
            'shuffle-card': shuffling
          }"
          :style="getCardStyle(index)"
          @click="selectCard(index)"
          @mouseenter="onCardHover(index, true)"
          @mouseleave="onCardHover(index, false)"
        >
          <div class="card-inner">
            <div class="card-back-face">
              <div class="card-symbol"></div>
            </div>
            <div class="card-front-face">
              <h3 class="magic-text" :class="selectedCardIndex || selectedCardIndex === 0 ? 'block' : 'hidden'">
                <span v-for="(item, index) in card.title.split(' ')" :key="index" :class="selectedCard ? 'inline-block' : 'hidden'" :style="`animation-delay: ${index < 1 ? 1 : 1 + index*0.15}s`">
                  {{ item }} <span> </span>
                </span>
              </h3>
              <p class="magic-text" :class="selectedCardIndex || selectedCardIndex === 0 ? 'block' : 'hidden'">
                <span v-for="(item, index) in card.message.split(' ')"  :class="selectedCard ? 'inline-block' : 'hidden'" :key="index" :style="`animation-delay: ${index < 1 ? 2 : 2 + index*0.15}s`">
                  {{ item }} <span> </span>
                </span>
              </p>
            </div>
          </div>
        </div>
      </transition-group>
    </div>

    <button v-if="selectedCard" @click="shuffleAndDeal" class="mystical-button reset-button">
      <span class="button-text">Consult Again</span>
    </button>
  </div>
</template>

<script>
import { ref, computed } from 'vue'
import { decisions } from '@/assets/mock/decides'

export default {
  name: 'MysticalCards',
  setup() {
    const cardsData = decisions

    const cards = ref([])
    const cardsDealt = ref(false)
    const selectedCardIndex = ref(null)
    const hoveredCardIndex = ref(null)
    const shuffling = ref(false);

    const selectedCard = computed(() => {
      if (selectedCardIndex.value !== null) {
        return cards.value[selectedCardIndex.value]
      }
      return null
    })

    function shuffleAndDeal() {
      resetCards()
      // Reset state
      selectedCardIndex.value = null;

      // Create a copy of the cards data and shuffle it
      const shuffled = [...cardsData]
        .map(card => ({ ...card, revealed: false }))
        .sort(() => Math.random() - 0.5);

      // Take only 5 cards
      cards.value = shuffled.slice(0, 30);

      // Start shuffle animation
      shuffling.value = true;

      // After shuffle animation completes, deal the cards
      setTimeout(() => {
        shuffling.value = false;

        // Set cards as dealt after a short delay
        setTimeout(() => {
          cardsDealt.value = true;
        });
      }, 4000); // Shuffle animation duration
    }

    function selectCard(index) {
      if (!cardsDealt.value || selectedCardIndex.value !== null || shuffling.value) return

      selectedCardIndex.value = index

      // Reveal the card after it moves to center
      setTimeout(() => {
        cards.value[index].revealed = true
      }, 800)
    }

    function resetCards() {
      cardsDealt.value = false
      selectedCardIndex.value = null
      cards.value = []
    }

    function onCardHover(index, isHovered) {
      hoveredCardIndex.value = isHovered ? index : null
    }

    function getCardStyle(index) {
      if (shuffling.value) {
        // During shuffle animation, each card gets a random position
        // This is handled by CSS, so we just return an empty style
        return {};
      }
      if (!cardsDealt.value) {
        // Initial position (stacked)
        return {
          transform: `translateX(0) translateY(0) rotate(0deg)`,
          transition: 'none',
        }
      }

      if (selectedCardIndex.value === index) {
        // Selected card moves to center
        return {
          transform: `translateX(0) translateY(0) rotate(0deg) scale(1.2)`,
          zIndex: 10,
        }
      }

      if (selectedCardIndex.value !== null) {
        // Other cards move away when one is selected
        const direction = index < selectedCardIndex.value ? -1 : 1
        return {
          transform: `translateX(${direction * 60}vw) translateY(20vh) rotate(${direction * 20}deg)`,
          opacity: 0.3,
        }
      }

      // Spread out the cards in a fan pattern
      const totalCards = cards.value.length
      const spreadWidth = 600 // percentage of container width
      const angle = 20 // maximum rotation angle

      // Calculate position in the fan
      const position = index - (totalCards - 1) / 2
      const xPercent = (position / (totalCards - 1)) * spreadWidth
      const rotation = position * (angle / ((totalCards - 1) / 2))

      // Apply hover effect - only move up on Y axis when hovered
      const yOffset = hoveredCardIndex.value === index ? -20 : 0

      return {
        transform: `translateX(${xPercent}%) translateY(${yOffset}px) rotate(${rotation}deg)`,
        boxShadow:
          hoveredCardIndex.value === index
            ? '0 20px 40px rgba(0, 0, 0, 0.6), 0 0 20px rgba(255, 215, 0, 0.5)'
            : '0 10px 30px rgba(0, 0, 0, 0.5)',
        zIndex: index,
      }
    }

    return {
      cards,
      cardsDealt,
      selectedCardIndex,
      selectedCard,
      shuffling,
      hoveredCardIndex,
      shuffleAndDeal,
      selectCard,
      resetCards,
      onCardHover,
      getCardStyle,
    }
  },
}
</script>

<style scoped lang="scss">

@keyframes appearAnimation {
  0% {
    opacity: 0;
    transform: translateY(15px) scale(0.2);
    filter: blur(10px);
  }
  60% {
    opacity: 1;
    transform: translateY(0) scale(1.05);
    filter: blur(0);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

.magic-text span {
  display: inline-block;
  opacity: 0;
  animation: appearAnimation 0.5s forwards
}

.mystical-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  position: relative;
  overflow: hidden;
  color: #f0e6ff;
}

.mystical-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #1a0b2e 0%, #2a1a4a 50%, #3b2a6a 100%);
  z-index: -1;
  overflow: hidden;
}

.mystical-background::before {
  content: '';
  position: absolute;
  width: 200%;
  height: 200%;
  top: -50%;
  left: -50%;
  background: radial-gradient(
    ellipse at center,
    rgba(255, 215, 0, 0.1) 0%,
    rgba(255, 215, 0, 0) 70%
  );
  animation: rotate 60s linear infinite;
}

@keyframes rotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(180deg);
  }
}

.title {
  font-size: 2.5rem;
  margin-bottom: 2rem;
  text-align: center;
  color: #ffd700;
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.5);
  letter-spacing: 2px;
}

.intro-message {
  text-align: center;
  max-width: 600px;
  margin-bottom: 2rem;
}

.intro-message p {
  font-size: 1.2rem;
  margin-bottom: 2rem;
  line-height: 1.6;
}

.mystical-button {
  background: linear-gradient(135deg, #4a2b7e 0%, #7b4ed0 100%);
  border: 2px solid #ffd700;
  color: #ffffff;
  padding: 0.8rem 2rem;
  font-size: 1.1rem;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  z-index: 1;
  box-shadow: 0 0 15px rgba(123, 78, 208, 0.5);
}

.mystical-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: all 0.6s ease;
  z-index: -1;
}

.mystical-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 7px 20px rgba(123, 78, 208, 0.7);
}

.mystical-button:hover::before {
  left: 100%;
}

.button-text {
  position: relative;
  z-index: 2;
}

.reset-button {
  margin-top: 2rem;
}

.cards-container.shuffling .card {
  transition: all 0.3s ease-in-out;
}

.shuffle-card {
  animation: shuffle 4s ease-in-out;
}

@keyframes shuffle {
  0%, 100% {
    transform: translateX(0) translateY(0) rotate(0deg);
  }
  10% {
    transform: translateX(-100px) translateY(-50px) rotate(-10deg);
  }
  20% {
    transform: translateX(150px) translateY(30px) rotate(5deg);
  }
  30% {
    transform: translateX(-80px) translateY(80px) rotate(15deg);
  }
  40% {
    transform: translateX(120px) translateY(-100px) rotate(-20deg);
  }
  50% {
    transform: translateX(-150px) translateY(-30px) rotate(10deg);
  }
  60% {
    transform: translateX(100px) translateY(90px) rotate(-15deg);
  }
  70% {
    transform: translateX(-120px) translateY(-70px) rotate(20deg);
  }
  80% {
    transform: translateX(80px) translateY(50px) rotate(-5deg);
  }
  90% {
    transform: translateX(-50px) translateY(-120px) rotate(15deg);
  }
}

@for $i from 1 through 30 {
  .shuffle-card:nth-child(#{$i}) {
    animation-delay: #{($i - 1) * 0.05}s;
  }
}

.cards-container {
  width: 100%;
  height: 60vh;
  position: relative;
  perspective: 1000px;
  margin: 2rem 0;
}

.cards-layout {
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.card {
  position: absolute;
  width: 220px;
  height: 340px;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  transform-style: preserve-3d;
  border-radius: 15px;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transform-style: preserve-3d ;
  transition: transform 1s ease-in-out;
}

.card.selected .card-inner {
  transform: rotateY(180deg);
}

.card-back-face,
.card-front-face {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 1.5rem;
  text-align: center;
}

.card-back-face {
  background: linear-gradient(135deg, #2a1a4a 0%, #4a2b7e 100%);
  border: 3px solid #ffd700;
}

.card-symbol {
  width: 80%;
  height: 80%;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'%3E%3Ccircle cx='50' cy='50' r='45' fill='none' stroke='%23ffd700' stroke-width='2'/%3E%3Cpath d='M50 5 L50 95 M5 50 L95 50 M26 26 L74 74 M26 74 L74 26' stroke='%23ffd700' stroke-width='1'/%3E%3Ccircle cx='50' cy='50' r='20' fill='none' stroke='%23ffd700' stroke-width='1'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.8;
}

.card-front-face {
  background: linear-gradient(135deg, #7b4ed0 0%, #4a2b7e 100%);
  border: 3px solid #ffd700;
  transform: rotateY(180deg);
}

.card-front-face h3 {
  color: #ffd700;
  font-size: 1.5rem;
  margin-bottom: 1rem;
  text-shadow: 0 0 5px rgba(255, 215, 0, 0.5);
}

.card-front-face p {
  color: #f0e6ff;
  font-size: 1rem;
  line-height: 1.5;
}

/* Transitions */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.card-move {
  transition: transform 0.8s;
}

.card-enter-active,
.card-leave-active {
  transition: all 0.8s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.card-enter-from {
  opacity: 0;
  transform: translateY(50px) scale(0.8);
}

.card-leave-to {
  opacity: 0;
  transform: translateY(-50px) scale(0.8);
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .title {
    font-size: 2rem;
  }

  .card {
    width: 180px;
    height: 280px;
  }

  .card-front-face h3 {
    font-size: 1.2rem;
  }

  .card-front-face p {
    font-size: 0.9rem;
  }
}

@media (max-width: 480px) {
  .title {
    font-size: 1.8rem;
  }

  .card {
    width: 150px;
    height: 230px;
  }

  .card-front-face h3 {
    font-size: 1rem;
  }

  .card-front-face p {
    font-size: 0.8rem;
  }
}
</style>
