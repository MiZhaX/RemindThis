<script setup>
import Cabecera from './Cabecera.vue';
import ResumenTareas from './ResumenTareas.vue';
import Tarea from './Tarea.vue';
import Pie from './Pie.vue'; 
import { useCollection, useFirestore } from 'vuefire'
import { collection, addDoc, orderBy, query, setDoc, doc, deleteDoc, where } from 'firebase/firestore'
import { ref, watch } from 'vue';
import { useCurrentUser } from 'vuefire';

const db = useFirestore();
var usuarioActual = useCurrentUser()
var listaTareas;

if(usuarioActual.value.email === 'admin@admin.com') {
  listaTareas = useCollection(query(collection(db, 'tasks'), orderBy('prioridad', 'desc')));
} else {
  listaTareas = useCollection(query(collection(db, 'tasks'), where('usuario', '==', usuarioActual.value.email), orderBy('prioridad', 'desc')));
}

var pendingTasks = ref(listaTareas.value.filter(task => !task.completado).length);

function updateList(){
  pendingTasks.value = listaTareas.value.filter(task => !task.completado).length;
  listaTareas.value.sort((a, b) => b.prioridad - a.prioridad);
}

function nuevaTarea(tarea) {
  var nuevaTarea = {
    titulo: tarea,
    prioridad: 0,
    completado: false,
    creado: new Date(),
    usuario: usuarioActual.value.email
  }

  // listaTareas.value.push(nuevaTarea);

  addDoc(collection(db, 'tasks'), nuevaTarea
  ).then((docRef) =>{

  }).catch((error) => {
    console.log(error);
  });
}

function eliminarTarea(index) {
  const tareaRef = doc(db, 'tasks', listaTareas.value[index].id);
  console.log(`Tarea ${listaTareas.value[index].titulo} eliminada`);
  deleteDoc(tareaRef)
    .then(() => {
    })
    .catch((error) => {
      console.error("Error al eliminar la tarea: ", error);
    });
}

watch(
  () => listaTareas.value,
  (newValue, oldValue) => {
    updateList();
    actualizarCreacion();
  },
  { deep: true }
);

function prioridadNueva(index, prioridad) {
  const tareaRef = doc(db, 'tasks', listaTareas.value[index].id);
  setDoc(tareaRef, { prioridad: prioridad }, { merge: true })
    .then(() => {})
    .catch((error) => {
      console.error("Error actualizando la prioridad: ", error);
    });
}

function completarTarea(index) {
  const tareaRef = doc(db, 'tasks', listaTareas.value[index].id);
  setDoc(tareaRef, { completado: !listaTareas.value[index].completado }, { merge: true })
    .then(() => {})
    .catch((error) => {
      console.error("Error al completar la tarea: ", error);
    });
}

function borrarCompletadas() {
  const completadas = listaTareas.value.filter(task => task.completado);
  completadas.forEach(task => {
    const tareaRef = doc(db, 'tasks', task.id);
    deleteDoc(tareaRef)
      .then(() => {
        console.log(`Tarea ${task.id} eliminada`);
      })
      .catch((error) => {
        console.error("Error al eliminar la tarea: ", error);
      });
  });
}

function actualizarCreacion() {
  if(listaTareas.value){
    listaTareas.value.forEach(task => {
      const minutos = Math.floor((Date.now() - task.creado.toDate()) / 60000);
      const horas = Math.floor(minutos / 60);
      const minutosRestantes = minutos % 60;
      if(minutosRestantes == 0 && horas == 0) task.tiempoTarea = `Creada recientemente`;
      else if(horas == 0) task.tiempoTarea = `Creada hace ${minutosRestantes}m`;
      else if(minutosRestantes == 0) task.tiempoTarea = `Creada hace ${horas}h`;
      else task.tiempoTarea = `Creada hace ${horas}h ${minutosRestantes}m`;
    });
  }
}

</script>

<template>
  <div class="container">
    <Cabecera @añadirTarea="nuevaTarea"></Cabecera>
    <ResumenTareas :tareasPendientes="pendingTasks" @borrarTareasCompletadas="borrarCompletadas"></ResumenTareas>
    <div class="tareas">
      <TransitionGroup name="fade" tag="div">
        <Tarea v-for="(tarea, index) in listaTareas" :key="index" :nombre="tarea.titulo" :complete="tarea.completado" :priority="tarea.prioridad" :tiempoTarea="tarea.tiempoTarea" @borrarTarea="eliminarTarea(index)" @cambioPrioridad="prioridadNueva(index, $event)" @marcarCompletada="completarTarea(index)"></Tarea>
      </TransitionGroup>
    </div>
    <Pie></Pie>
  </div>
</template>

<style scoped>
.container {
  background: #2d2b2b;
  border-radius: 8px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  padding: 2rem;
  width: 700px;
  max-height: 500px;
  text-align: center;
  margin: 0 auto; 
  position: absolute;
  top: 57%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.tareas {
  max-height: 210px; 
  overflow-y: auto;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>