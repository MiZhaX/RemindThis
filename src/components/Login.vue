<script setup>
import { GoogleAuthProvider, createUserWithEmailAndPassword, signInWithEmailAndPassword } from 'firebase/auth'
import { signInWithPopup, signOut } from 'firebase/auth'
import { useRouter } from 'vue-router';
import { useCurrentUser, useFirebaseAuth } from 'vuefire'
import { ref } from 'vue';

const googleAuthProvider = new GoogleAuthProvider()
const user = useCurrentUser()
const auth = useFirebaseAuth()
const router = useRouter();
const showModal = ref(false);
const email = ref('');
const password = ref('');
const isRegistering = ref(false);

function iniciarSesion() {
    signInWithPopup(auth, googleAuthProvider).then(
        (result) => {
            console.log(result.user)
            router.push('/recordatorios');
            showModal.value = false;
        }
    ).catch((reason) => {
        console.error('Error', reason)
    })
}

function iniciarSesionConCorreo() {
    signInWithEmailAndPassword(auth, email.value, password.value).then(
        (result) => {
            console.log(result.user)
            router.push('/recordatorios');
            email.value = '';
            password.value = '';
            showModal.value = false;
        }
    ).catch((reason) => {
        console.error('Error', reason)
    })
}

function registrarConCorreo() {
    createUserWithEmailAndPassword(auth, email.value, password.value).then(
        (result) => {
            console.log(result.user)
            router.push('/recordatorios');
            email.value = '';
            password.value = '';
            showModal.value = false;
        }
    ).catch((reason) => {
        console.error('Error', reason)
    })
}

function cerrarSesion() {
    signOut(auth).then(
        () => {
            console.log("Se ha cerrado la sesión");
            router.push('/');
            localStorage.removeItem('user');
        }
    ).catch(
        (e) => console.log(e)
    );
}

</script>

<template>
    <div class="login">
        <div class="userInfo">
            <img v-if="user" :src="user.photoURL" alt="userPhoto" class="userPhoto">
            <p v-if="user">{{ user.displayName }}</p>
        </div>
        <button v-if="user" @click='cerrarSesion'>Cerrar Sesión</button>
        <button v-else @click="showModal = true" class="logout">Iniciar Sesión</button>
    </div>

    <div v-if="showModal" class="modal">
        <div class="modal-content">
            <span class="close" @click="showModal = false">&times;</span>
            <h2>{{ isRegistering ? 'Registrarse' : 'Iniciar Sesión' }}</h2>
            <input type="email" v-model="email" placeholder="Correo electrónico">
            <input type="password" v-model="password" placeholder="Contraseña">
            <button @click="isRegistering ? registrarConCorreo() : iniciarSesionConCorreo()">
                {{ isRegistering ? 'Registrarse' : 'Iniciar Sesión' }}
            </button>
            <button @click="iniciarSesion">Iniciar Sesión con Google</button>
            <p>
                {{ isRegistering ? '¿Ya tienes una cuenta? ' : '¿No tienes una cuenta? ' }}
                <span @click="isRegistering = !isRegistering" class="register">
                    {{ isRegistering ? 'Inicia sesión' : 'Regístrate' }}
                </span>
            </p>
        </div>
    </div>
</template>

<style scoped>
.login {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
}

.userInfo {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 20px;
}

.userPhoto {
    border-radius: 50%;
    width: 60px;
}

button {
    background-color: #ff4b5c;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
}

button:hover {
    background-color: #ff1f3a;
}

.modal {
    display: flex;
    justify-content: center;
    align-items: center;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 5px;
    text-align: center;
}

.close {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 20px;
    cursor: pointer;
}

.register{
    cursor: pointer;
    color: #efb514;
}
</style>

