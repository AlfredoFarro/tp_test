<template>
  <div>
    <AppNavbar></AppNavbar>

    <!-- Pantalla de introducción: Pregunta sobre TDAH -->
    <div v-if="introScreen">
      <h2>¿Tienes TDAH?</h2>
      <button @click="handleTdahResponse(true)">Sí</button>
      <button @click="handleTdahResponse(false)">No</button>
    </div>

    <!-- Pantalla de selección de tareas -->
    <div v-else-if="!selectedTask">
      <h2>Seleccione una tarea</h2>
      <button @click="selectTask('stroop')">Stroop Task</button>
      <button @click="selectTask('cpt')">Continuous Performance Task</button>
      <button @click="selectTask('stop')">Stop Signal Task</button>
    </div>

    <!-- Mostrar la tarea seleccionada -->
    <div v-else>
      <!-- Botón para regresar a la selección de tareas -->
      <button @click="selectedTask = null">Volver</button>

      <!-- Mostrar la tarea correspondiente -->
      <StroopTask v-if="selectedTask === 'stroop'" @backToTaskSelection="selectedTask = null" />
      <ContinuousPerformanceTask 
        v-if="selectedTask === 'cpt'" 
        @backToTaskSelection="selectedTask = null"  
      />
      <StopSignalTask v-if="selectedTask === 'stop'" @backToTaskSelection="selectedTask = null" />
    </div>
  </div>
</template>

<script>
import StroopTask from './components/StroopTask.vue';
import ContinuousPerformanceTask from './components/ContinuousPerformanceTask.vue';
import StopSignalTask from './components/StopSignalTask.vue';
import AppNavbar from './components/AppNavbar.vue';

export default {
  data() {
    return {
      introScreen: true, // Controla si se muestra la pantalla inicial sobre TDAH
      selectedTask: null // Controla qué tarea está seleccionada
    };
  },
  components: {
    StroopTask,
    ContinuousPerformanceTask,
    StopSignalTask,
    AppNavbar
  },
  methods: {
    handleTdahResponse(response) {
      this.introScreen = false; // Oculta la pantalla de TDAH después de seleccionar una opción
      // Puedes guardar la respuesta del usuario aquí si la necesitas
      console.log("Respuesta sobre TDAH:", response);
    },
    selectTask(task) {
      this.selectedTask = task; // Asigna la tarea seleccionada
    }
  }
};
</script>

<style scoped>
body {
  margin: 0px !important;
}

h2 {
  text-align: center;
}

button {
  display: block;
  margin: 10px auto;
  padding: 10px 20px;
  font-size: 18px;
  cursor: pointer;
}

button:hover {
  background-color: #f0f0f0;
}
</style>
