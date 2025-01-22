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
    <RouterLink to="/"><img src="../public/logo.png" alt="Logo" width="50" height="50"></RouterLink>
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
  background-color: rgba(0, 0, 0, 0.26);
  color: white;
  position: fixed;
  z-index: 100; 
  width: 100%;
  height: 7rem;
  font-size: x-large;
  font-weight: bold;
  align-items: center;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  padding-left: 8rem;
  padding-right: 8rem;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.6);
}

.navegacion {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 2rem;
}

.navegacion a{
  text-decoration: none;
  color: white;
  padding: 10px;
  border-radius: 10px;
}

.navegacion a:hover{
  background-color: #efb514;
  color: rgb(8, 19, 25);
}
</style>