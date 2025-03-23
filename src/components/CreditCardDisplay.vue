<template>
  <div class="flex items-center bg-white border border-gray-200 rounded-2xl px-3 py-2 w-[300px] shadow-sm">
    <div class="flex-1 text-base font-bold text-gray-800 tracking-wider relative overflow-hidden" style="font-family: 'Google Sans', sans-serif;">
      <div class="flex space-x-1 relative">
        <div v-for="(group, groupIndex) in displayNumberGroups" :key="groupIndex" class="flex">
          <template v-for="(digit, digitIndex) in group.split('')" :key="`${groupIndex}-${digitIndex}`">
            <div class="number-slot relative overflow-hidden w-[12px] h-[20px]">
              <transition name="slide-digit" mode="out-in">
                <div v-if="shouldShowDigit(groupIndex, digitIndex)" 
                     :key="'digit'"
                     class="absolute w-full h-full flex justify-center items-center"
                     :style="{ 
                       transitionDelay: `${getTransitionDelay(groupIndex, digitIndex, true)}ms`,
                       transitionDuration: `${getTransitionDuration()}ms`
                     }">
                  {{ digit }}
                </div>
                <div v-else 
                     :key="'x'"
                     class="absolute w-full h-full flex justify-center items-center"
                     :style="{ 
                       transitionDelay: `${getTransitionDelay(groupIndex, digitIndex, false)}ms`,
                       transitionDuration: `${getTransitionDuration()}ms`
                     }">
                  x
                </div>
              </transition>
            </div>
            <span v-if="digitIndex === 3 && groupIndex < 3" class="mx-0.5"></span>
          </template>
        </div>
        <div v-if="isVisible && !isResetting" class="global-shine-effect"></div>
      </div>
    </div>
    <button 
      class="flex items-center justify-center w-10 h-10 ml-2 rounded-xl transition-all duration-300 relative overflow-hidden"
      :class="{
        'bg-indigo-100 text-indigo-600 hover:bg-indigo-200': !isVisible && !isCopied,
        'bg-green-100 text-green-600 hover:bg-green-200': isVisible && !isCopied,
        'bg-green-500 text-white hover:bg-green-600': isCopied
      }"
      @click="handleButtonClick"
    >
      <div v-if="isVisible && !isCopied" class="absolute inset-0">
        <svg class="absolute inset-0 w-full h-full" viewBox="0 0 40 40">
          <path 
            d="M20 2 L28 2 A10 10 0 0 1 38 12 V28 A10 10 0 0 1 28 38 H12 A10 10 0 0 1 2 28 V12 A10 10 0 0 1 12 2 L20 2"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            class="animate-border-countdown"
          />
        </svg>
      </div>
      
      <!-- Icône œil -->
      <svg v-if="!isVisible && !isCopied" xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 transition-all duration-300" viewBox="0 0 24 24" fill="currentColor" stroke="none">
        <path d="M12 15a3 3 0 100-6 3 3 0 000 6z" />
        <path fill-rule="evenodd" d="M1.323 11.447C2.811 6.976 7.028 3.75 12.001 3.75c4.97 0 9.185 3.223 10.675 7.69.12.362.12.752 0 1.113-1.487 4.471-5.705 7.697-10.677 7.697-4.97 0-9.186-3.223-10.675-7.69a1.762 1.762 0 010-1.113zM17.25 12a5.25 5.25 0 11-10.5 0 5.25 5.25 0 0110.5 0z" clip-rule="evenodd" />
      </svg>
      
      <!-- Icône copier -->
      <svg v-if="isVisible && !isCopied" xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 transition-all duration-300" viewBox="0 0 24 24" fill="currentColor" stroke="none">
        <path d="M7.5 3.375c0-1.036.84-1.875 1.875-1.875h.375a3.75 3.75 0 013.75 3.75v1.875C13.5 8.161 14.34 9 15.375 9h1.875A3.75 3.75 0 0121 12.75v3.375C21 17.16 20.16 18 19.125 18h-9.75A1.875 1.875 0 017.5 16.125V3.375z" />
        <path d="M15 5.25a5.23 5.23 0 00-1.279-3.434 9.768 9.768 0 016.963 6.963A5.23 5.23 0 0017.25 7.5h-1.875A.375.375 0 0115 7.125V5.25zM4.875 6H6v10.125A3.375 3.375 0 009.375 19.5H16.5v1.125c0 1.035-.84 1.875-1.875 1.875h-9.75A1.875 1.875 0 013 20.625V7.875C3 6.839 3.84 6 4.875 6z" />
      </svg>
      
      <!-- Icône checkmark -->
      <svg v-if="isCopied" xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 transition-all duration-300" viewBox="0 0 24 24" fill="currentColor">
        <path fill-rule="evenodd" d="M19.916 4.626a.75.75 0 01.208 1.04l-9 13.5a.75.75 0 01-1.154.114l-6-6a.75.75 0 011.06-1.06l5.353 5.353 8.493-12.739a.75.75 0 011.04-.208z" clip-rule="evenodd" stroke="currentColor" stroke-width="1"/>
      </svg>
    </button>
  </div>
</template>

<script>
export default {
  name: 'CreditCardDisplay',
  data() {
    return {
      cardNumber: '4485199620577516',
      isVisible: false,
      isCopied: false,
      transitionTimeout: null,
      resetTimeout: null,
      progressValue: 100,
      isResetting: false
    }
  },
  computed: {
    displayNumberGroups() {
      const formattedFull = this.formatCardNumber(this.cardNumber);
      return formattedFull.split(' ');
    }
  },
  methods: {
    formatCardNumber(number) {
      const parts = [];
      for (let i = 0; i < number.length; i += 4) {
        parts.push(number.substring(i, i + 4));
      }
      return parts.join(' ');
    },
    shouldShowDigit(groupIndex, digitIndex) {
      if (this.isCopied || this.isVisible) return true;
      return groupIndex === 0 || groupIndex === 3;
    },
    getTransitionDelay(groupIndex, digitIndex, isShowing) {
      const totalIndex = (groupIndex * 4) + digitIndex;
      const baseDelay = 50;
      
      if (this.isResetting) {
        return totalIndex * baseDelay;
      }
      
      if (!this.isVisible) return 0;
      return totalIndex * baseDelay;
    },
    getTransitionDuration() {
      return 300; // 400ms pour la durée de transition
    },
    getLastDigitRevealTime() {
      // Calculer le temps total pour révéler tous les chiffres
      const lastIndex = 15; // 16 chiffres au total (0-15)
      const baseDelay = 50;
      const transitionDuration = this.getTransitionDuration();
      return (lastIndex * baseDelay) + transitionDuration;
    },
    handleButtonClick() {
      this.clearAllTimeouts();
      this.isResetting = false;
      
      if (this.isCopied) {
        this.resetState();
      } else if (this.isVisible) {
        this.copyToClipboard();
        this.isCopied = true;
        
        this.resetTimeout = setTimeout(() => {
          this.resetState();
        }, 3000);
      } else {
        this.isVisible = true;
        this.progressValue = 100;
        
        this.resetTimeout = setTimeout(() => {
          this.resetState();
        }, 3000);
      }
    },
    copyToClipboard() {
      const textToCopy = this.formatCardNumber(this.cardNumber);
      navigator.clipboard.writeText(textToCopy).catch(err => {
        console.error('Erreur lors de la copie :', err);
      });
    },
    resetState() {
      this.isResetting = true;
      this.isCopied = false;
      this.isVisible = false;
      
      // Réinitialiser isResetting après la fin de l'animation
      setTimeout(() => {
        this.isResetting = false;
      }, 1200); // Délai suffisant pour que toutes les animations soient terminées
    },
    clearAllTimeouts() {
      if (this.transitionTimeout) {
        clearTimeout(this.transitionTimeout);
        this.transitionTimeout = null;
      }
      if (this.resetTimeout) {
        clearTimeout(this.resetTimeout);
        this.resetTimeout = null;
      }
    }
  },
  beforeUnmount() {
    this.clearAllTimeouts();
  }
}
</script>

<style>
@keyframes border-countdown {
  0% {
    stroke-dasharray: 120;
    stroke-dashoffset: 0;
  }
  100% {
    stroke-dasharray: 120;
    stroke-dashoffset: 120;
  }
}

.animate-border-countdown {
  animation: border-countdown 3s linear forwards;
  transform-origin: center;
}

.number-slot {
  perspective: 1000px;
  font-feature-settings: "tnum" on, "lnum" on;
  font-variant-numeric: tabular-nums;
}

.slide-digit-enter-active,
.slide-digit-leave-active {
  transition-property: transform, opacity;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-digit-enter-from {
  transform: translateY(-100%) rotateX(90deg);
  opacity: 0;
}

.slide-digit-leave-to {
  transform: translateY(-100%) rotateX(90deg);
  opacity: 0;
}

.slide-digit-enter-to,
.slide-digit-leave-from {
  transform: translateY(0) rotateX(0);
  opacity: 1;
}

/* Ajout d'une animation plus douce pour la retombée */
.slide-digit-enter-active {
  transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1);
}

.digit-wrapper {
  position: relative;
}

.global-shine-effect {
  position: absolute;
  top: 0;
  left: 0;
  width: 70%;
  height: 100%;
  background: linear-gradient(
    to right,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.6) 50%,
    rgba(255, 255, 255, 0) 100%
  );
  transform: skewX(-20deg) translateX(-100%);
  animation: global-shine 2s ease-in-out forwards;
  /* On attend que tous les chiffres soient révélés (environ 1150ms) avant de commencer l'effet */
  animation-delay: 1150ms;
  pointer-events: none;
  z-index: 10;
}

@keyframes global-shine {
  0% {
    transform: skewX(-20deg) translateX(-100%);
  }
  100% {
    transform: skewX(-20deg) translateX(300%);
  }
}
</style>