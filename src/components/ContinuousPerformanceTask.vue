<template>
  <div class="cpt-task">
    <h2>Presiona espacio cuando aparezca la letra "{{ targetLetter }}"</h2>
    <h1>{{ currentLetter }}</h1>

    <p v-if="result !== null" :class="result ? 'correct' : 'wrong'">
      {{ result ? 'Correcto' : 'Incorrecto' }}
    </p>

    <!-- Temporizador -->
    <div class="timer">
      Tiempo restante: {{ timer }} segundos
    </div>

    <!-- Pantalla de error -->
    <div v-if="showErrorScreen" class="error-screen">
      <h1>¡Te has equivocado!</h1>
      <p>Intenta de nuevo presionando la tecla de espacio cuando aparezca la letra "{{ targetLetter }}".</p>
      <button @click="closeErrorScreen">Intentar de nuevo</button>
    </div>

    <!-- Resultado final -->
    <div v-if="timeUp" class="result-screen">
      <h1>Tiempo agotado</h1>
      <p>Acertaste: {{ correctAnswers }} veces</p>
      <button @click="resetGame">Reiniciar juego</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      letters: 'ABCDEFGHIJ'.split(''),  // Lista de letras de A a J
      targetLetter: '',  // Letra objetivo
      currentLetter: '',  // Letra actual que se muestra
      result: null,  // Resultado
      intervalId: null,
      timer: 60,  // Temporizador de 60 segundos
      timerInterval: null,
      correctAnswers: 0,  // Contador de respuestas correctas
      timeUp: false,  // Controla si el tiempo ha terminado
      showErrorScreen: false,  // Controla la pantalla de error
    };
  },
  mounted() {
    this.startTask();
    this.startTimer();
    this.generateTargetLetter(); // Genera la letra objetivo inicial
    window.addEventListener('keydown', this.handleKeyPress);
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeyPress);
    clearInterval(this.intervalId);
    clearInterval(this.timerInterval);
  },
  methods: {
    startTask() {
      this.intervalId = setInterval(() => {
        this.generateRandomLetter();
      }, 1000);  // Cambia la letra cada segundo
    },
    generateRandomLetter() {
      const randomIndex = Math.floor(Math.random() * this.letters.length);
      this.currentLetter = this.letters[randomIndex];
    },
    generateTargetLetter() {
      const randomIndex = Math.floor(Math.random() * this.letters.length);
      this.targetLetter = this.letters[randomIndex]; // Cambia la letra objetivo
    },
    handleKeyPress(event) {
      if (event.code === 'Space') {
        this.checkAnswer();
      }
    },
    checkAnswer() {
      if (this.currentLetter === this.targetLetter) {
        this.result = true;
        this.correctAnswers++;  // Incrementa el contador de aciertos
        this.generateTargetLetter(); // Cambia la letra objetivo al acertar
      } else {
        this.result = false;
        this.showErrorScreen = true;  // Muestra la pantalla de error
        this.resetTimer();  // Reinicia el temporizador si se equivoca
      }

      setTimeout(() => {
        this.result = null;  // Restablece el resultado después de un segundo
      }, 1000);
    },
    startTimer() {
      this.timerInterval = setInterval(() => {
        if (this.timer > 0) {
          this.timer--;
        } else {
          this.timeUp = true; // Marca que el tiempo ha terminado
          clearInterval(this.timerInterval); // Detiene el temporizador
        }
      }, 1000);
    },
    resetTimer() {
      this.timer = 60; // Reinicia el temporizador a 60 segundos
      clearInterval(this.timerInterval); // Detiene el temporizador actual
      this.startTimer(); // Inicia un nuevo temporizador
    },
    closeErrorScreen() {
      this.showErrorScreen = false;
      this.result = null;  // Restablece el resultado
      this.generateRandomLetter(); // Genera una nueva letra
    },
    resetGame() {
      this.correctAnswers = 0; // Reinicia el contador de aciertos
      this.timer = 60; // Reinicia el temporizador a 60 segundos
      this.timeUp = false; // Reinicia el estado de tiempo
      this.generateTargetLetter(); // Genera una nueva letra objetivo
      this.generateRandomLetter(); // Genera una nueva letra inicial
      this.startTimer(); // Inicia el temporizador
    }
  }
};
</script>

<style scoped>
.cpt-task {
  text-align: center;
}

h1 {
  font-size: 72px;
}

.correct {
  color: green;
}

.wrong {
  color: red;
}

/* Temporizador */
.timer {
  font-size: 24px; /* Tamaño del texto del temporizador */
  margin: 20px 0;
}

/* Pantalla de error */
.error-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(255, 0, 0, 0.8);
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px; /* Añadir padding */
}

.error-screen h1 {
  font-size: 48px;
}

.error-screen p {
  font-size: 24px; /* Tamaño del texto de la descripción */
  margin: 10px 0;
}

.error-screen button {
  background-color: white;
  color: black;
  font-size: 15px;
  border: none;
  cursor: pointer;
  border-radius: 5px; /* Bordes redondeados */
}

.error-screen button:hover {
  background-color: #f0f0f0;
}

/* Pantalla de resultado final */
.result-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8); /* Fondo oscuro */
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px; /* Añadir padding */
}

.result-screen h1 {
  font-size: 48px;
}

.result-screen p {
  font-size: 24px; /* Tamaño del texto de la descripción */
  margin: 10px 0;
}

.result-screen button {
  background-color: white;
  color: black;
  font-size: 15px;
  border: none;
  cursor: pointer;
  border-radius: 5px; /* Bordes redondeados */
}

.result-screen button:hover {
  background-color: #f0f0f0;
}
</style>
