<template>
  <div class="stroop-task">
    <!-- Pantalla de inicio -->
    <div v-if="!taskStarted" class="instructions-screen">
      <h1>Instrucciones</h1>
      <p>
        El objetivo de esta evaluación es identificar el <strong>color de la tinta</strong> de la palabra mostrada, no el <strong>significado</strong> de la palabra.
      </p>
      <h2>Ejemplo:</h2>
      <p>AZUL</p>
      <p>Tendrás que presionar el color azul, y luego continuar con el color que te salga.</p>
      <button @click="startTask">Iniciar Tarea</button>
    </div>

    <!-- Tarea principal -->
    <div v-if="taskStarted">
      <h2>Seleccione el color</h2>
      <h1 :style="{ color: displayedColor.code }">{{ displayedColor.name }}</h1>

      <div class="color-buttons">
        <button
          v-for="color in colors"
          :key="color.name"
          :style="{ backgroundColor: color.code }"
          @click="checkAnswer(color)"
        ></button>
      </div>

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
        <p>Intenta de nuevo seleccionando el color correcto.</p>
      </div>

      <!-- Resultado final -->
      <div v-if="timeUp" class="result-screen">
        <h1>Tiempo agotado</h1>
        <p>Acertaste: {{ correctAnswers }} veces</p>
        <p>Errores: {{ wrongAnswers }} veces</p>
        <p>Tiempo de respuesta promedio: {{ averageResponseTime }} ms</p>
        <button @click="goToInstructions">Volver a la pantalla inicial</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      taskStarted: false, // Controla si la tarea ha comenzado
      colors: [
        { name: 'ROJO', code: '#ff0000' },
        { name: 'NEGRO', code: '#000000' },
        { name: 'NARANJA', code: '#ff9900' },
        { name: 'VERDE', code: '#00cc99' },
        { name: 'AZUL', code: '#0000ff' },
        { name: 'AMARILLO', code: '#ffff00' },
        { name: 'MORADO', code: '#800080' },
        { name: 'ROSADO', code: '#ff69b4' }
      ],
      displayedColor: {},
      result: null,
      showErrorScreen: false,
      timer: 60, // Temporizador de 60 segundos
      timerInterval: null,
      correctAnswers: 0, // Contador de respuestas correctas
      wrongAnswers: 0, // Contador de respuestas incorrectas
      timeUp: false, // Controla si el tiempo ha terminado
      responseTimes: [], // Almacena los tiempos de respuesta
      startTime: null, // Guarda el tiempo en que comienza cada pregunta
    };
  },
  methods: {
    // Iniciar la tarea
    startTask() {
      this.taskStarted = true;  // Marca que la tarea ha comenzado
      this.resetGame(); // Reinicia el juego y genera el primer color
      this.startTimer(); // Inicia el temporizador
    },
    generateRandomColor() {
      const randomColor = this.colors[Math.floor(Math.random() * this.colors.length)];
      const randomText = this.colors[Math.floor(Math.random() * this.colors.length)];
      this.displayedColor = { name: randomText.name, code: randomColor.code };
      this.startTime = new Date(); // Inicia el temporizador para la respuesta
    },
    checkAnswer(color) {
      const responseTime = new Date() - this.startTime; // Calcula el tiempo en milisegundos
      this.responseTimes.push(responseTime); // Almacena el tiempo de respuesta

      if (color.code === this.displayedColor.code) {
        this.result = true;
        this.correctAnswers++; // Incrementa el contador de aciertos
        setTimeout(() => {
          this.result = null;
          this.generateRandomColor(); // Genera un nuevo color
        }, 1000);
      } else {
        this.result = false;
        this.wrongAnswers++; // Incrementa el contador de errores
        this.showErrorScreen = true;

        setTimeout(() => {
          this.showErrorScreen = false;
          this.result = null;
          this.generateRandomColor(); // Genera un nuevo color
        }, 1000); // Mantiene la pantalla de error durante 1 segundo
      }
    },
    startTimer() {
      this.timer = 60; // Reinicia el temporizador a 60 segundos
      this.timerInterval = setInterval(() => {
        if (this.timer > 0) {
          this.timer--;
        } else {
          this.timeUp = true; // Marca que el tiempo ha terminado
          clearInterval(this.timerInterval); // Detiene el temporizador
        }
      }, 1000);
    },
    resetGame() {
      this.correctAnswers = 0; // Reinicia el contador de aciertos
      this.wrongAnswers = 0; // Reinicia el contador de errores
      this.responseTimes = []; // Reinicia los tiempos de respuesta
      this.timer = 60; // Reinicia el temporizador a 60 segundos
      this.timeUp = false; // Reinicia el estado de tiempo
      this.generateRandomColor(); // Genera un nuevo color
    },
    goToInstructions() {
      this.$emit('backToTaskSelection'); // Regresa a la pantalla de instrucciones
    }
  },
  computed: {
    averageResponseTime() {
      const totalResponseTime = this.responseTimes.reduce((acc, time) => acc + time, 0);
      return this.responseTimes.length ? (totalResponseTime / this.responseTimes.length).toFixed(0) : 0;
    }
  },
  beforeUnmount() {
    clearInterval(this.timerInterval); // Limpia el temporizador al destruir el componente
  }
};
</script>


<style scoped>
.stroop-task {
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
  width: auto;
  height: auto;
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

/* Estilos para los colores */
h1 {
  font-size: 48px;
}

.color-buttons {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

button {
  width: 50px;
  height: 50px;
  margin: 0 10px;
  border: none;
  cursor: pointer;
}

/* Resultados */
.correct {
  color: green;
}

.wrong {
  color: red;
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
</style>
