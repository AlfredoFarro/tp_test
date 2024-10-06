<template>
  <div class="stop-signal-task">
    <!-- Pantalla de instrucciones -->
    <div v-if="!taskStarted" class="instructions-screen">
      <h1>Instrucciones</h1>
      <p>
        El objetivo de este test es evaluar tu capacidad de respuesta y autocontrol.
      </p>
      <p>
        Durante la prueba, verás flechas en pantalla y deberás presionar las teclas de dirección correspondientes.
      </p>
      <h2>Ejemplo:</h2>
      <p>⬅️ Deberás presionar la tecla izquierda</p>
      <p>➡️ Deberás presionar la tecla derecha</p>
      <button @click="startTask">Empezar</button>
    </div>

    <!-- Tarea principal -->
    <div v-if="taskStarted">
      <h2>Presiona la tecla de flecha correspondiente</h2>
      <div class="game-container">
        <div class="arrow-container">
          <span v-if="showArrow && currentDirection === 'left'" class="arrow">⬅️</span>
          <span v-if="showArrow && currentDirection === 'right'" class="arrow">➡️</span>
          <!-- Mostrar el símbolo de espera en la misma posición que las flechas -->
          <span v-if="showWaitingSymbol" class="waiting-symbol">🔄</span>
        </div>

        <!-- Texto de resultado -->
        <p v-if="showResult" :class="result === 'correct' ? 'correct' : result === 'wrong' ? 'wrong' : 'missed'" class="result-text">
          {{ result === 'correct' ? 'Correcto' : result === 'wrong' ? 'Incorrecto' : 'No presionaste :(' }}
        </p>

        <!-- Temporizador -->
        <div class="timer">
          Tiempo restante: {{ timer }} segundos
        </div>
      </div>

      <!-- Resultado final -->
      <div v-if="timeUp" class="result-screen">
        <h1>Tiempo agotado</h1>
        <p>Acertaste: {{ correctAnswers }} veces</p>
        <p>Errores: {{ wrongAnswers }} veces</p>
        <p>Flechas omitidas: {{ missedArrows }} veces</p>
        <p>Tiempo de reacción promedio: {{ averageResponseTime }} ms</p>
        <button @click="goToTaskSelection">Volver a la selección de tareas</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      taskStarted: false,
      directions: ['left', 'right'],
      currentDirection: '',
      result: null,
      intervalId: null,
      timer: 60,
      timerInterval: null,
      correctAnswers: 0,
      wrongAnswers: 0,
      missedArrows: 0,
      timeUp: false,
      showArrow: false,
      showResult: false,
      hasAnswered: false,
      responseTimes: [],
      startTime: null,
      showWaitingSymbol: false, // Controla la visibilidad del símbolo de espera
    };
  },
  mounted() {
    window.addEventListener('keydown', this.handleKeyPress);
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeyPress);
    clearInterval(this.intervalId);
    clearInterval(this.timerInterval);
  },
  methods: {
    startTask() {
      this.resetGame();
      this.taskStarted = true;
      this.startTimer();
      this.showNextArrow();
    },

    showNextArrow() {
      if (this.timeUp) return; // Detener si el tiempo ha terminado

      this.generateRandomDirection();
      this.showArrow = true;
      this.startTime = new Date();
      this.result = null;
      this.hasAnswered = false;

      // Mostrar la flecha por 1 segundo, luego ocultarla y esperar 1 segundo más
      setTimeout(() => {
        this.showArrow = false;

        // Indicador de espera durante 1 segundo
        this.showWaitingSymbol = true;
        setTimeout(() => {
          this.showWaitingSymbol = false;

          if (!this.hasAnswered) {
            this.result = 'missed';
            this.missedArrows++;
          }
          this.showResult = true;

          // Esperar otro segundo para ocultar el resultado y pasar a la siguiente flecha
          setTimeout(() => {
            this.showResult = false;
            if (!this.timeUp) {
              this.showNextArrow(); // Repetir el ciclo
            }
          }, 1000);
        }, 1000); // Espera 1 segundo antes de mostrar el resultado
      }, 1000);
    },

    generateRandomDirection() {
      const randomIndex = Math.floor(Math.random() * this.directions.length);
      this.currentDirection = this.directions[randomIndex];
    },

    handleKeyPress(event) {
      if (!this.timeUp && (event.key === 'ArrowLeft' || event.key === 'ArrowRight')) {
        const responseTime = (new Date() - this.startTime); // Calcula el tiempo de respuesta en ms

        if (
          (event.key === 'ArrowLeft' && this.currentDirection === 'left') ||
          (event.key === 'ArrowRight' && this.currentDirection === 'right')
        ) {
          if (!this.hasAnswered) {
            this.result = 'correct';
            this.correctAnswers++;
            this.responseTimes.push(responseTime);
            this.hasAnswered = true;
          }
        } else {
          this.result = 'wrong';
          this.wrongAnswers++;
          this.hasAnswered = true;
        }
      }
    },

    startTimer() {
      this.timerInterval = setInterval(() => {
        if (this.timer > 0) {
          this.timer--;
        } else {
          this.timeUp = true;
          clearInterval(this.timerInterval);
        }
      }, 1000);
    },

    resetGame() {
      this.correctAnswers = 0;
      this.wrongAnswers = 0;
      this.missedArrows = 0;
      this.responseTimes = [];
      this.timer = 60;
      this.timeUp = false;
      this.showArrow = false;
      this.showResult = false;
      this.showWaitingSymbol = false;
    },

    goToTaskSelection() {
      this.$emit('backToTaskSelection'); // Regresa a la pantalla de selección de tareas
    },
  },
  computed: {
    averageResponseTime() {
      const totalResponseTime = this.responseTimes.reduce((acc, time) => acc + time, 0);
      return this.responseTimes.length ? (totalResponseTime / this.responseTimes.length).toFixed(0) : 0;
    },
  },
};
</script>

<style scoped>
.stop-signal-task {
  text-align: center;
}

/* Pantalla de instrucciones */
.instructions-screen {
  text-align: center;
  margin: 20px;
}

.instructions-screen h1 {
  font-size: 36px;
  margin-bottom: 20px;
}

.instructions-screen p {
  font-size: 18px;
  margin-bottom: 20px;
}

.instructions-screen button {
  padding: 10px 20px;
  font-size: 18px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.instructions-screen button:hover {
  background-color: #45a049;
}

/* Contenedor de juego */
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative; /* Permite el uso de posiciones absolutas dentro */
  height: 200px; /* Altura fija para evitar el movimiento del temporizador */
}

/* Contenedor de flechas */
.arrow-container {
  font-size: 72px;
  position: absolute; /* Permite el posicionamiento absoluto de las flechas y el símbolo */
  top: 0; /* Ajusta esto según sea necesario para centrarlo verticalmente */
  left: 50%; /* Centrado horizontalmente */
  transform: translateX(-50%); /* Ajuste para centrar */
}

/* Texto de resultado */
.result-text {
  font-size: 24px;
  margin-bottom: 20px; /* Espacio fijo para evitar el movimiento */
}

/* Resultados */
.correct {
  color: green;
}

.wrong {
  color: red;
}

.missed {
  color: orange;
}

/* Temporizador */
.timer {
  font-size: 24px;
  position: absolute; /* Mantiene el temporizador en la misma posición */
  bottom: 20px; /* Espacio desde el fondo */
  left: 50%; /* Centrado horizontalmente */
  transform: translateX(-50%); /* Ajuste para centrar */
}

/* Pantalla de resultado final */
.result-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.result-screen h1 {
  font-size: 48px;
}

.result-screen p {
  font-size: 24px;
}

.result-screen button {
  background-color: white;
  color: black;
  font-size: 15px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.result-screen button:hover {
  background-color: #f0f0f0;
}

/* Símbolo de espera */
.waiting-symbol {
  font-size: 72px;
  color: #aaa;
  position: absolute; /* Para que esté en la misma posición que las flechas */
  top: 0; /* Ajusta esto según sea necesario para centrarlo verticalmente */
  left: 50%; /* Centrado horizontalmente */
  transform: translateX(-50%); /* Ajuste para centrar */
}
</style>
