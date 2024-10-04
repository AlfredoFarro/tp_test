<template>
  <div class="cpt-task">
    <!-- Pantalla de inicio -->
    <div v-if="!taskStarted" class="instructions-screen">
      <h1>Instrucciones</h1>
      <p>
        El objetivo de esta evaluación es medir tu <strong>atención</strong> y <strong>velocidad de respuesta</strong>.
        Durante la prueba, verás letras en pantalla y deberás hacer clic en la letra que corresponde en el momento en que aparezca.
      </p>
      <h2>Ejemplo:</h2>
      <p>Presione la letra: A</p>
      <p>Deberás hacer clic en la letra A en el momento que aparezca.</p>
      <button @click="startTask">Iniciar Tarea</button>
    </div>

    <!-- Tarea principal -->
    <div v-if="taskStarted">
      <h2 v-if="!timeUp">Presiona espacio cuando aparezca la letra "{{ targetLetter }}"</h2>
      <h1 v-if="!timeUp">{{ currentLetter }}</h1>

      <!-- Temporizador -->
      <div class="timer" v-if="!timeUp">
        Tiempo restante: {{ timer }} segundos
      </div>

      <!-- Pantalla de error -->
      <div v-if="showErrorScreen" class="error-screen">
        <h1>¡Te has equivocado!</h1>
        <p>Intenta de nuevo presionando la tecla de espacio cuando aparezca la letra "{{ targetLetter }}".</p>
      </div>

      <!-- Resultado final -->
      <div v-if="timeUp" class="result-screen">
        <h1>Resultados</h1>
        <p>Aciertos: {{ correctAnswers }}</p>
        <p>Errores: {{ incorrectAnswers }}</p>
        <p v-if="reactionTimes.length > 0">Tiempo de reacción promedio: {{ averageReactionTime }} ms</p>
        <button @click="resetGame">Reiniciar juego</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      taskStarted: false,  // Controla si la tarea ha comenzado
      letters: 'ABCDEFGHIJ'.split(''),  // Lista de letras de A a J
      targetLetter: '',  // Letra objetivo
      currentLetter: '',  // Letra actual que se muestra
      correctAnswers: 0,  // Contador de respuestas correctas
      incorrectAnswers: 0,  // Contador de errores
      reactionTimes: [],  // Arreglo de tiempos de reacción
      averageReactionTime: null,  // Tiempo de reacción promedio
      timer: 60,  // Temporizador de 60 segundos
      timeUp: false,  // Controla si el tiempo ha terminado
      intervalId: null,  // Intervalo para cambiar letras
      timerInterval: null,  // Intervalo para el temporizador
      showErrorScreen: false,  // Controla la pantalla de error
      letterStartTime: null,  // Tiempo en que la letra actual apareció
    };
  },
  methods: {
    // Iniciar la tarea
    startTask() {
      this.taskStarted = true;  // Marca que la tarea ha comenzado
      this.startTimer();
      this.startLetterChange();
      this.generateTargetLetter(); // Genera la letra objetivo inicial
      window.addEventListener('keydown', this.handleKeyPress);
    },
    startLetterChange() {
      this.intervalId = setInterval(() => {
        this.generateRandomLetter();
      }, 1000);  // Cambia la letra cada segundo
    },
    generateRandomLetter() {
      const randomIndex = Math.floor(Math.random() * this.letters.length);
      this.currentLetter = this.letters[randomIndex];
      this.letterStartTime = Date.now();  // Registra el momento en que aparece la letra
    },
    generateTargetLetter() {
      const randomIndex = Math.floor(Math.random() * this.letters.length);
      this.targetLetter = this.letters[randomIndex]; // Cambia la letra objetivo
    },
    handleKeyPress(event) {
      if (event.code === 'Space' && !this.timeUp) {
        this.checkAnswer();
      }
    },
    checkAnswer() {
      const currentTime = Date.now();
      const reactionTime = currentTime - this.letterStartTime;  // Calcula el tiempo de reacción

      if (this.currentLetter === this.targetLetter) {
        this.correctAnswers++;  // Incrementa el contador de aciertos
        this.reactionTimes.push(reactionTime);  // Agrega el tiempo de reacción al arreglo
        this.generateTargetLetter(); // Cambia la letra objetivo al acertar
      } else {
        this.incorrectAnswers++;  // Incrementa el contador de errores
        this.showErrorScreen = true;  // Muestra la pantalla de error
        setTimeout(() => {
          this.showErrorScreen = false;
        }, 1000);  // La pantalla de error se muestra durante 1 segundo
      }
    },
    startTimer() {
      this.timerInterval = setInterval(() => {
        if (this.timer > 0) {
          this.timer--;
        } else {
          this.endTask();  // Finaliza la tarea cuando el tiempo se agote
        }
      }, 1000);
    },
    endTask() {
      this.timeUp = true;  // Marca que el tiempo ha terminado
      clearInterval(this.intervalId);  // Detiene el cambio de letras
      clearInterval(this.timerInterval);  // Detiene el temporizador

      // Calcular tiempo de reacción promedio
      if (this.reactionTimes.length > 0) {
        const totalReactionTime = this.reactionTimes.reduce((acc, time) => acc + time, 0);
        this.averageReactionTime = (totalReactionTime / this.reactionTimes.length).toFixed(2);
      }
    },
    resetGame() {
      this.taskStarted = false;  // Reinicia el estado de inicio
      this.correctAnswers = 0;  // Reinicia el contador de aciertos
      this.incorrectAnswers = 0;  // Reinicia el contador de errores
      this.reactionTimes = [];  // Reinicia los tiempos de reacción
      this.averageReactionTime = null;  // Reinicia el promedio de tiempos de reacción
      this.timer = 60;  // Reinicia el temporizador a 60 segundos
      this.timeUp = false;  // Reinicia el estado de tiempo
      clearInterval(this.intervalId);  // Detiene el cambio de letras
      clearInterval(this.timerInterval);  // Detiene el temporizador
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
}

.error-screen h1 {
  font-size: 48px;
}

.error-screen p {
  font-size: 24px;
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
