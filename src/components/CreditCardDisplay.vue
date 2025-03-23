<template>
  <div class="card-number-display">
    <div class="number-text">
      {{ displayNumber }}
    </div>
    <button 
      class="visibility-toggle" 
      @click="handleButtonClick" 
      :class="{ 
        'copied': isCopied, 
        'visible': isVisible && !isCopied 
      }"
    >
      <div v-if="isVisible && !isCopied" class="progress-border"></div>
      
      <!-- Icône œil -->
      <svg v-if="!isVisible && !isCopied" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="icon">
        <path d="M2 12s3-7 10-7 10 7 10 7-3 7-10 7-10-7-10-7Z" />
        <circle cx="12" cy="12" r="3" />
      </svg>
      
      <!-- Icône copier -->
      <svg v-if="isVisible && !isCopied" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="icon">
        <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
        <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
      </svg>
      
      <!-- Icône checkmark -->
      <svg v-if="isCopied" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="icon">
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
        // État de transition - montrer partiellement les chiffres
        const parts = formattedFull.split(' ');
        return `${parts[0]} 1xxx 2xxx ${parts[3]}`;
      } else {
        // État initial - masquer les 8 chiffres du milieu
        const parts = formattedFull.split(' ');
        return `${parts[0]} xxxx xxxx ${parts[3]}`;
      }
    }
  },
  methods: {
    formatCardNumber(number) {
      // Formater avec des espaces tous les 4 chiffres
      const parts = [];
      for (let i = 0; i < number.length; i += 4) {
        parts.push(number.substring(i, i + 4));
      }
      return parts.join(' ');
    },
    handleButtonClick() {
      // Annuler tout timeout existant
      this.clearAllTimeouts();
      
      if (this.isCopied) {
        // Si on a déjà copié, revenir à l'état initial
        this.resetState();
      } else if (this.isVisible) {
        // Si les chiffres sont visibles, copier le numéro
        this.copyToClipboard();
        this.isCopied = true;
        
        // Réinitialiser après 4 secondes
        this.resetTimeout = setTimeout(() => {
          this.resetState();
        }, 4000);
      } else {
        // Si les chiffres sont masqués, commencer la transition pour les révéler
        this.isPartiallyVisible = true;
        
        // Après un court délai, afficher complètement les chiffres
        this.transitionTimeout = setTimeout(() => {
          this.isVisible = true;
          this.isPartiallyVisible = false;
          this.progressValue = 100; // Réinitialiser la progression
          
          // Démarrer l'animation de la bordure qui diminue
          this.startProgressAnimation();
          
          // Réinitialiser après 4 secondes si l'utilisateur ne fait rien
          this.resetTimeout = setTimeout(() => {
            this.resetState();
          }, 4000);
        }, 300);
      }
    },
    startProgressAnimation() {
      // Créer une animation CSS pour la bordure qui diminue
      const progressBorder = document.querySelector('.progress-border');
      if (progressBorder) {
        progressBorder.style.animation = 'none';
        setTimeout(() => {
          progressBorder.style.animation = 'progress-decrease 4s linear forwards';
        }, 10);
      }
    },
    copyToClipboard() {
      // Copier le numéro de carte dans le presse-papier
      const textToCopy = this.formatCardNumber(this.cardNumber);
      navigator.clipboard.writeText(textToCopy).catch(err => {
        console.error('Erreur lors de la copie :', err);
      });
    },
    resetState() {
      // Réinitialiser à l'état initial
      this.isCopied = false;
      this.isVisible = false;
      this.isPartiallyVisible = false;
    },
    clearAllTimeouts() {
      // Nettoyer tous les timeouts
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
    // Nettoyer les timeouts si le composant est détruit
    this.clearAllTimeouts();
  }
}
</script>

<style scoped>
.card-number-display {
  display: flex;
  align-items: center;
  background-color: white;
  border: 1px solid #e6e6e6;
  border-radius: 24px;
  padding: 12px 16px;
  max-width: 340px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.number-text {
  flex: 1;
  font-size: 16px;
  font-weight: 500;
  color: #333;
  letter-spacing: 0.5px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.visibility-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  background: #eaecff; /* Fond lavande clair pour l'état initial */
  border: none;
  border-radius: 12px; /* Carré arrondi au lieu d'un cercle */
  width: 40px;
  height: 40px;
  cursor: pointer;
  color: #5c6bc0; /* Couleur bleue pour l'icône d'œil */
  transition: all 0.3s ease;
  margin-left: 8px;
  position: relative;
  overflow: hidden;
}

.visibility-toggle.visible {
  background: #e1f7e1; /* Fond vert clair pour l'état visible */
  color: #4caf50; /* Couleur verte pour l'icône de copie */
}

.visibility-toggle.copied {
  background: #4caf50; /* Fond vert pour l'état copié */
  color: white; /* Icône blanche pour l'état copié */
}

.progress-border {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border: 2px solid #4caf50;
  border-radius: 12px;
  pointer-events: none;
}

@keyframes progress-decrease {
  0% {
    clip-path: inset(0 0 0 0 round 12px);
  }
  25% {
    clip-path: inset(0 0 25% 0 round 12px);
  }
  50% {
    clip-path: inset(0 0 50% 0 round 12px);
  }
  75% {
    clip-path: inset(0 0 75% 0 round 12px);
  }
  100% {
    clip-path: inset(0 0 100% 0 round 12px);
  }
}

.icon {
  width: 20px;
  height: 20px;
  position: relative;
  z-index: 1;
}

.visibility-toggle:hover {
  background-color: #dfe3ff; /* Couleur de survol pour l'état initial */
}

.visibility-toggle.visible:hover {
  background-color: #d0f0d0; /* Couleur de survol pour l'état visible */
}

.visibility-toggle.copied:hover {
  background-color: #43a047; /* Couleur de survol pour l'état copié */
}
</style>