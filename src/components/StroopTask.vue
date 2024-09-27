<template>
    <div class="stroop-task">
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
        timeUp: false // Controla si el tiempo ha terminado
      };
    },
    created() {
      this.generateRandomColor();
      this.startTimer();
    },
    methods: {
      generateRandomColor() {
        const randomColor = this.colors[Math.floor(Math.random() * this.colors.length)];
        const randomText = this.colors[Math.floor(Math.random() * this.colors.length)];
        this.displayedColor = { name: randomText.name, code: randomColor.code };
      },
      checkAnswer(color) {
        if (color.code === this.displayedColor.code) {
          this.result = true;
          this.correctAnswers++; // Incrementa el contador de aciertos
          setTimeout(() => {
            this.result = null;
            this.generateRandomColor();
          }, 1000);
        } else {
          this.result = false;
          this.showErrorScreen = true;
          this.resetTimer(); // Reinicia el temporizador si se equivoca
        }
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
        this.result = null;
        this.generateRandomColor();
      },
      resetGame() {
        this.correctAnswers = 0; // Reinicia el contador de aciertos
        this.timer = 60; // Reinicia el temporizador a 60 segundos
        this.timeUp = false; // Reinicia el estado de tiempo
        this.generateRandomColor(); // Genera un nuevo color
        this.startTimer(); // Inicia el temporizador
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
  
  /* Temporizador */
  .timer {
    font-size: 24px; /* Tamaño del texto del temporizador */
    margin: 20px 0;
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
  