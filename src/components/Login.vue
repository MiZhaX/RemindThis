<script setup>
import { GoogleAuthProvider, createUserWithEmailAndPassword, signInWithEmailAndPassword } from 'firebase/auth'
import { signInWithPopup, signOut } from 'firebase/auth'
import { useRouter } from 'vue-router';
import { useCurrentUser, useFirebaseAuth } from 'vuefire'
import { ref } from 'vue';
import { GithubAuthProvider } from 'firebase/auth'

const googleAuthProvider = new GoogleAuthProvider()
const githubAuthProvider = new GithubAuthProvider()
const user = useCurrentUser()
const auth = useFirebaseAuth()
const router = useRouter();
const showModal = ref(false);
const email = ref('');
const password = ref('');
const repeatPassword = ref('');
const registrando = ref(false);
var contraseñasIguales = ref(true);
var correoInvalido = ref(false);
var contraseñaInvalida = ref(false);
var contraseñaDebil = ref(false);
var correoEnUso = ref(false);

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

function iniciarSesionConGithub() {
    signInWithPopup(auth, githubAuthProvider).then(
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
    contraseñasIguales.value = true;
    correoInvalido.value = false;
    contraseñaInvalida.value = false;
    contraseñaDebil.value = false;
    correoEnUso.value = false;
    
    signInWithEmailAndPassword(auth, email.value, password.value).then(
        (result) => {
            console.log(result.user)
            router.push('/recordatorios');
            email.value = '';
            password.value = '';
            repeatPassword.value = '';
            showModal.value = false;
            contraseñasIguales.value = true;
            correoInvalido.value = false;
            contraseñaInvalida.value = false;
        }
    ).catch((reason) => {
        if (reason.code === 'auth/invalid-email') {
            console.error('El correo no está registrado');
            correoInvalido.value = true;
        } else if (reason.code === 'auth/invalid-credential') {
            console.error('Contraseña incorrecta');
            contraseñaInvalida.value = true;
        } else {
            console.error('Error', reason);
        }
    })
}

function registrarConCorreo() {
    contraseñasIguales.value = true;
    correoInvalido.value = false;
    contraseñaInvalida.value = false;
    contraseñaDebil.value = false;
    correoEnUso.value = false;

    if (password.value !== repeatPassword.value) {
        console.error('Las contraseñas no coinciden')
        contraseñasIguales.value = false;
        return;
    }
    createUserWithEmailAndPassword(auth, email.value, password.value).then(
        (result) => {
            console.log(result.user)
            router.push('/recordatorios');
            email.value = '';
            password.value = '';
            repeatPassword.value = '';
            showModal.value = false;
            contraseñasIguales.value = true;
            correoInvalido.value = false;
        }
    ).catch((reason) => {
        if (reason.code === 'auth/invalid-email') {
            console.error('El correo no es válido');
            correoInvalido.value = true;
        } else if (reason.code === 'auth/weak-password') {
            console.error('La contraseña es débil');
            contraseñaDebil.value = true;
        } else if (reason.code === 'auth/email-already-in-use') {
            console.error('El correo ya está registrado');
            correoEnUso.value = true;
        } else {
            console.error('Error', reason);
        }
    })
}

function cerrarSesion() {
    signOut(auth).then(
        () => {
            console.log("Se ha cerrado la sesión");
            router.push('/');
            localStorage.removeItem('user');
            registrando.value = false;
        }
    ).catch(
        (e) => console.log(e)
    );
}

function mostrarFormulario() {
    email.value = '';
    password.value = '';
    repeatPassword.value = '';
    contraseñasIguales.value = true;
    correoInvalido.value = false;
    contraseñaInvalida.value = false;
    contraseñaDebil.value = false;
    correoEnUso.value = false;
    showModal.value = true;
    registrando.value = !registrando.value;
}

function cerrarFormulario() {
    email.value = '';
    password.value = '';
    repeatPassword.value = '';
    contraseñasIguales.value = true;
    correoInvalido.value = false;
    contraseñaInvalida.value = false;
    contraseñaDebil.value = false;
    correoEnUso.value = false;
    showModal.value = false;
    registrando.value = false;
}

</script>

<template>
    <div class="login">
        <div class="userInfo">
            <p v-if="user">{{ user.displayName }}</p>
        </div>
        <button v-if="user" @click='cerrarSesion'>Cerrar Sesión</button>
        <button v-else @click="showModal = true" class="logout">Iniciar Sesión</button>
    </div>

    <div v-if="showModal" class="modal">
        <div class="modal-content">
            <span class="closeModal" @click="cerrarFormulario">&times;</span>
            <h2>{{ registrando ? 'Registrarse' : 'Iniciar Sesión' }}</h2>

            <!-- ERRRORES -->
            <span v-if="contraseñasIguales == false" class="error">Las contraseñas no coinciden</span>
            <span v-if="correoInvalido == true" class="error">El correo no es válido</span>
            <span v-if="contraseñaInvalida == true" class="error">La contraseña no es correcta</span>
            <span v-if="contraseñaDebil == true" class="error">La contraseña debe tener mínimo 6 caracteres</span>
            <span v-if="correoEnUso == true" class="error">El correo ya está en uso</span>

            <input type="email" v-model="email" placeholder="Correo electrónico">
            <input type="password" v-model="password" placeholder="Contraseña">
            <input v-if="registrando" type="password" v-model="repeatPassword" placeholder="Repite la contraseña">
            <button @click="registrando ? registrarConCorreo() : iniciarSesionConCorreo()">
                {{ registrando ? 'Registrarse' : 'Iniciar Sesión' }}
            </button>
            <div class="opcionesExternas">
                <button @click="iniciarSesion"><i class="fa-brands fa-google fa-2xl"></i></button>
                <button @click="iniciarSesionConGithub"><i class="fa-brands fa-github fa-2xl"></i></button>
            </div>
            <p>
                {{ registrando ? '¿Ya tienes una cuenta? ' : '¿No tienes una cuenta? ' }}
                <span @click="mostrarFormulario" class="register">
                    {{ registrando ? 'Inicia sesión' : 'Regístrate' }}
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
    gap: 10px;
}

.userInfo {
    display: flex;
    flex-direction: row;
    align-items: center;
}

.userPhoto {
    border-radius: 50%;
    width: 60px;
}

.opcionesExternas {
    display: flex;
    flex-direction: row;
    gap: 20px;
}

button {
    background-color: #efb514;
    color: black;
    font-weight: bold;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
}

button:hover {
    background-color: #bd9012;
    color: white;
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
    width: 480px;
    height: 400px;
    display: flex;
    flex-direction: column;
    gap: 15px;
    align-items: center;
    background-color: white;
    color: black;
    padding: 20px;
    border-radius: 5px;
    text-align: center;
}

.modal-content input {
    text-align: center;
    width: 80%;
    height: 25px;
}

.closeModal {
    position: relative;
    top: -5px;
    right: -190px;
    font-size: 20px;
    cursor: pointer;
    padding: 5px;
    border-radius: 30px;
}

.closeModal:hover {
    background-color: #efb514;
}

.register {
    cursor: pointer;
    color: #efb514;
}

.error {
    color: red;
    font-size: medium;
}
</style>
