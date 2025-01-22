<script setup>
import { ref } from 'vue';

const emit = defineEmits(['añadirTarea']);

var nombre = ref("");

function añadirTarea(){
    if(nombre.value == "") return
    else {
        emit("añadirTarea", nombre.value);
        nombre.value = "";
    }
}

</script>

<template class="header">
    <h1>RemindThis</h1>
    <input type="text" id="taskList-input" placeholder="e.g. Acabar Proyecto" v-model="nombre" @keyup.enter="añadirTarea">
</template>

<style scoped>
.header {
    top: 0;
    position: fixed;
    z-index: 1000;
}

h1 {
    margin-bottom: 1rem;
    color: #efb514;
}

#taskList-input {
    width: 75%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    width: 100%;
}

</style>