<template>
  <div class="flex items-center bg-white border border-gray-200 rounded-3xl px-4 py-3 max-w-[340px] shadow-sm">
    <div class="flex-1 text-base font-medium text-gray-800 tracking-wider font-sans">
      {{ displayNumber }}
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
      <div v-if="isVisible && !isCopied" class="absolute inset-0 border-2 border-green-500 rounded-xl" :class="{'animate-progress-decrease': isVisible}"></div>
      
      <!-- Icône œil -->
      <svg v-if="!isVisible && !isCopied" xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="M2 12s3-7 10-7 10 7 10 7-3 7-10 7-10-7-10-7Z" />
        <circle cx="12" cy="12" r="3" />
      </svg>
      
      <!-- Icône copier -->
      <svg v-if="isVisible && !isCopied" xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
        <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
      </svg>
      
      <!-- Icône checkmark -->
      <svg v-if="isCopied" xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="M20 6L9 17l-5-5"></path>
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
      isPartiallyVisible: false,
      isCopied: false,
      transitionTimeout: null,
      resetTimeout: null,
      progressValue: 100
    }
  },
  computed: {
    displayNumber() {
      const formattedFull = this.formatCardNumber(this.cardNumber);
      
      if (this.isCopied || this.isVisible) {
        return formattedFull;
      } else if (this.isPartiallyVisible) {
        const parts = formattedFull.split(' ');
        return `${parts[0]} 1xxx 2xxx ${parts[3]}`;
      } else {
        const parts = formattedFull.split(' ');
        return `${parts[0]} xxxx xxxx ${parts[3]}`;
      }
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
    handleButtonClick() {
      this.clearAllTimeouts();
      
      if (this.isCopied) {
        this.resetState();
      } else if (this.isVisible) {
        this.copyToClipboard();
        this.isCopied = true;
        
        this.resetTimeout = setTimeout(() => {
          this.resetState();
        }, 4000);
      } else {
        this.isPartiallyVisible = true;
        
        this.transitionTimeout = setTimeout(() => {
          this.isVisible = true;
          this.isPartiallyVisible = false;
          this.progressValue = 100;
          
          this.resetTimeout = setTimeout(() => {
            this.resetState();
          }, 4000);
        }, 300);
      }
    },
    copyToClipboard() {
      const textToCopy = this.formatCardNumber(this.cardNumber);
      navigator.clipboard.writeText(textToCopy).catch(err => {
        console.error('Erreur lors de la copie :', err);
      });
    },
    resetState() {
      this.isCopied = false;
      this.isVisible = false;
      this.isPartiallyVisible = false;
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
@keyframes progress-decrease {
  0% { clip-path: inset(0 0 0 0 round 0.75rem); }
  25% { clip-path: inset(0 0 25% 0 round 0.75rem); }
  50% { clip-path: inset(0 0 50% 0 round 0.75rem); }
  75% { clip-path: inset(0 0 75% 0 round 0.75rem); }
  100% { clip-path: inset(0 0 100% 0 round 0.75rem); }
}

.animate-progress-decrease {
  animation: progress-decrease 4s linear forwards;
}
</style>