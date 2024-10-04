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
      <div class="arrow-container">
        <span v-if="showArrow && currentDirection === 'left'" class="arrow">⬅️</span>
        <span v-if="showArrow && currentDirection === 'right'" class="arrow">➡️</span>
      </div>

      <p v-if="showResult" :class="result === 'correct' ? 'correct' : result === 'wrong' ? 'wrong' : 'missed'">
        {{ result === 'correct' ? 'Correcto' : result === 'wrong' ? 'Incorrecto' : 'No presionaste :(' }}
      </p>

      <!-- Temporizador -->
      <div class="timer">
        Tiempo restante: {{ timer }} segundos
      </div>

      <!-- Resultado final -->
      <div v-if="timeUp" class="result-screen">
        <h1>Tiempo agotado</h1>
        <p>Acertaste: {{ correctAnswers }} veces</p>
        <p>Errores: {{ wrongAnswers }} veces</p>
        <p>Flechas omitidas: {{ missedArrows }} veces</p>
        <p>Tiempo de reacción promedio: {{ averageResponseTime }} segundos</p>
        <button @click="resetGame">Reiniciar juego</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      taskStarted: false, // Controla si la tarea ha comenzado
      directions: ['left', 'right'], // Lista de direcciones
      currentDirection: '', // Dirección actual (flecha izquierda o derecha)
      result: null, // Resultado (correcto, incorrecto o omitido)
      intervalId: null,
      timer: 60, // Temporizador de 60 segundos
      timerInterval: null,
      correctAnswers: 0, // Contador de respuestas correctas
      wrongAnswers: 0, // Contador de respuestas incorrectas
      missedArrows: 0, // Contador de flechas omitidas
      timeUp: false, // Controla si el tiempo ha terminado
      showArrow: false, // Controla la visibilidad de la flecha
      showResult: false, // Controla la visibilidad del resultado
      hasAnswered: false, // Controla si se ha acertado la flecha actual
      responseTimes: [], // Almacena los tiempos de respuesta
      startTime: null, // Momento en que aparece una nueva flecha
    };
  },
  mounted() {
    window.addEventListener('keydown', this.handleKeyPress); // Añadir el evento de tecla al montar
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeyPress); // Limpiar el evento al desmontar
    clearInterval(this.intervalId); // Limpiar el intervalo de flechas
    clearInterval(this.timerInterval); // Limpiar el intervalo del temporizador
  },
  methods: {
    startTask() {
      this.resetGame(); // Reinicia el juego y variables
      this.taskStarted = true; // Marca que la tarea ha comenzado
      this.startTimer(); // Inicia el temporizador
      this.showNextArrow(); // Iniciar el ciclo de flechas de inmediato
    },

    showNextArrow() {
      // Mostrar la flecha inmediatamente
      this.generateRandomDirection();
      this.showArrow = true;
      this.startTime = new Date(); // Inicia el temporizador para la respuesta
      this.result = null; // Reiniciar el resultado de la respuesta
      this.hasAnswered = false; // Reiniciar el estado de la respuesta

      // Mostrar la flecha por 1 segundo, luego ocultarla y esperar 1 segundo más
      setTimeout(() => {
        this.showArrow = false; // Ocultar flecha después de 1 segundo
        setTimeout(() => {
          // Mostrar el resultado si no se ha presionado una tecla
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
          }, 1000); // Mostrar el resultado durante 1 segundo
        }, 1000); // Esperar 1 segundo con la flecha oculta antes de mostrar el resultado
      }, 1000); // Mostrar la flecha durante 1 segundo
    },

    generateRandomDirection() {
      const randomIndex = Math.floor(Math.random() * this.directions.length);
      this.currentDirection = this.directions[randomIndex];
    },

    handleKeyPress(event) {
      if (!this.timeUp && (event.key === 'ArrowLeft' || event.key === 'ArrowRight')) {
        const responseTime = (new Date() - this.startTime) / 1000; // Calcula el tiempo de respuesta

        if (
          (event.key === 'ArrowLeft' && this.currentDirection === 'left') ||
          (event.key === 'ArrowRight' && this.currentDirection === 'right')
        ) {
          if (!this.hasAnswered) {
            this.result = 'correct';
            this.correctAnswers++;
            this.responseTimes.push(responseTime);
            this.hasAnswered = true; // Marca que se ha respondido
          }
        } else {
          this.result = 'wrong';
          this.wrongAnswers++;
          this.hasAnswered = true; // Marca que se ha respondido
        }
      }
    },

    startTimer() {
      this.timerInterval = setInterval(() => {
        if (this.timer > 0) {
          this.timer--; // Decrementa el temporizador de 1 en 1
        } else {
          this.timeUp = true; // Marca que el tiempo ha terminado
          clearInterval(this.timerInterval); // Detiene el temporizador
          clearInterval(this.intervalId); // Detiene el intervalo de flechas
        }
      }, 1000);
    },

    resetGame() {
      this.correctAnswers = 0; // Reinicia el contador de aciertos
      this.wrongAnswers = 0; // Reinicia el contador de errores
      this.missedArrows = 0; // Reinicia el contador de flechas omitidas
      this.responseTimes = []; // Reinicia los tiempos de respuesta
      this.timer = 60; // Reinicia el temporizador a 60 segundos
      this.timeUp = false; // Reinicia el estado de tiempo
      this.showArrow = false; // Oculta las flechas al reiniciar
      this.showResult = false; // Oculta el resultado al reiniciar
    }
  },
  computed: {
    averageResponseTime() {
      const totalResponseTime = this.responseTimes.reduce((acc, time) => acc + time, 0);
      return this.responseTimes.length ? (totalResponseTime / this.responseTimes.length).toFixed(2) : 0;
    }
  }
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

/* Contenedor de flechas */
.arrow-container {
  font-size: 72px;
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
  margin: 20px 0;
}

/* Pantalla de resultado final */
.result-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0,
  0, 0, 0.8);
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
</style>
