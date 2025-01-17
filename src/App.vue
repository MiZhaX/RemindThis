<script setup>
  import { useRouter } from 'vue-router';
  import { getCurrentUser } from 'vuefire';
  import Login from './components/Login.vue';

  const router = useRouter();

  router.beforeEach(async (to, from) => {
    if (to.meta.requiresAuth){
      const user = await getCurrentUser();
      if(!user){
        return false;
      } else return true;
    } else return true;
  });
</script>
<template>
  <div class="cabecera">
    <nav class="navegacion">
      <RouterLink to="/">Inicio</RouterLink>
      <RouterLink to="/recordatorios">Recordatorios</RouterLink>
    </nav>
    <Login></Login>
  </div>   
  <RouterView></RouterView>
</template>



<style scoped>
.cabecera {
  background-color: #efb514;
  color: black;
  position: fixed;
  z-index: 100; 
  width: 100%;
  height: 7rem;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.6);
  font-size: x-large;
  font-weight: bold;
  align-items: center;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  padding-left: 8rem;
  padding-right: 8rem;
}

.navegacion {
  display: flex;
  flex-direction: row;
  gap: 2rem;
}


</style>