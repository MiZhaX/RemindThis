<script setup>
import { GoogleAuthProvider } from 'firebase/auth'
import { 
    signInWithPopup, 
    signOut,
} from 'firebase/auth'
import { useRouter } from 'vue-router';
import { useCurrentUser, useFirebaseAuth } from 'vuefire'

const emit = defineEmits(['establecerUsuario', 'cerrarSesion']);
const googleAuthProvider = new GoogleAuthProvider()
const user = useCurrentUser()
const auth = useFirebaseAuth()
const router = useRouter();

function iniciarSesion(){
    signInWithPopup(auth, googleAuthProvider).then(
        (result) => {
            console.log(result.user)
            localStorage.setItem('user', JSON.stringify(result.user))
            console.log(JSON.parse(localStorage.getItem('user')));
            router.push('/recordatorios');
            emit("establecerUsuario", result.user.email)
        }
    ).catch((reason) => {
        console.error('Failed sign', reason)
    })
}

function cerrarSesion(){
    signOut(auth).then(
        () => {
            console.log("Se ha cerrado la sesión");
            router.push('/');
            localStorage.removeItem('user');
            
            emit("cerrarSesion");
        }
    ).catch(
        (e) => console.log(e)
    );
}

</script>
<template>
    <div class="login">
        <p v-if="user">Bienvenido {{ user.displayName }}</p>
        <button v-if="user" @click='cerrarSesion'>Cerrar Sesión</button>
        <button v-else @click="iniciarSesion">Iniciar Sesión</button>
    </div>
</template>

<style scoped>
.login{
    display: flex;
    flex-direction: row;
    gap: 2rem;
}
</style>

