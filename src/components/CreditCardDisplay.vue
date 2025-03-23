<template>
  <div class="flex items-center bg-white border border-gray-200 rounded-2xl px-3 py-2 w-[300px] shadow-sm">
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
  animation: border-countdown 4s linear forwards;
  transform-origin: center;
}
</style>