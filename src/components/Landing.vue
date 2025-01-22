<template>
    <section class="landing">
        <h1 class="titulo">RemindThis</h1>
        <h2 class="eslogan">Recuerda tareas y notas importantes con nosotros</h2>
    </section>
</template>

<script setup>

</script>

<style scoped>
.landing {
    padding-top: 3rem;
    display: flex;
    justify-content: center;
    flex-direction: column;
    background-color: rgb(8, 19, 25);
    align-items: center;
    min-height: 100vh;
    perspective: 1400px;
    position: relative;
    overflow: hidden; 
}

.landing::before {
    content: "";
    position: absolute;
    left: 5%; 
    width: 120%; 
    height: 120%; 
    background-image: url('../../public/fondo.png');
    background-size: cover; 
    background-position: center;
    transform: rotateY(35deg); 
    transform-origin: left;
    z-index: -1; 
}

.titulo{
    color: #efb514;
    font-size: 50px;
    padding: 15px;
    border-radius: 15px;
    background-color: rgba(0, 0, 0, 0.597);
}

.eslogan{
    color: #efb514;
    font-size: 30px;
    padding: 15px;
    border-radius: 15px;
    background-color: rgba(0, 0, 0, 0.597);
}
</style>