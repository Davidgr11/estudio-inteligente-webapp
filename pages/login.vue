<script setup lang="ts">
import {
    signInWithEmailAndPassword, getAuth, signOut,
    sendPasswordResetEmail,
} from 'firebase/auth'

const loading = ref(false)

// Hardcodeando los valores de usuario y contraseña
const login = reactive({
    username: 'davidalejandro.gr11@gmail.com',
    password: 'Studially1-DG',
})

const resetPassword = reactive({
    email: '',
    sent: false,
    error: false,
})

definePageMeta({
    auth: {
        unauthenticatedOnly: true,
        navigateAuthenticatedTo: '/',
    }
})
const { signIn } = useAuth()

const { query: { error } } = useRoute()
const loginError = computed(() => error && error !== 'undefined')

const auth = getAuth()
async function submit() {
    try {
        const { user } = await signInWithEmailAndPassword(auth, login.username, login.password)
    } catch (error) {
        alert(error.message)
        return;
    }

    loading.value = true
    const response = await signIn('credentials', login)
    loading.value = false
}
</script>

<template>
    <div class="card text-center col-12 col-md-8 mx-auto">
        <div class="card-header">
            <h3 class="card-title">
                <BootstrapIconBoxArrowInRight />
                Demo
            </h3>
        </div>
        <div class="card-body">
            <form class="px-5 text-start" @submit.prevent="submit">
                <div class="d-grid">
                    <button class="btn btn-primary btn-lg" id="enter" :disabled="loading">
                        <span class="spinner-border spinner-border-sm" role="status" v-if="loading" aria-hidden="true">
                        </span>
                        <span v-else>
                            <BootstrapIconBoxArrowInRight />
                            Enter
                        </span>
                    </button>
                </div>
            </form>
        </div>
    </div>
    <ClientOnly>
        <BModal id="recuperar-contraseña" title="Recuperar contraseña" colour="primary">
            <template #body>
                <p class="text-center">
                    Ingresa tu correo electrónico y te enviaremos un enlace para restablecer tu contraseña.
                </p>
                <form class="px-5 text-start">
                    <div class="mb-3">
                        <div class="form-floating mb-3">
                            <input type="email" class="form-control" id="correo-electrónico" placeholder="tu@correo.com"
                                v-model="resetPassword.email">
                            <label for="correo-electrónico">
                                <BootstrapIconEnvelope /> Correo
                                electrónico
                            </label>
                        </div>
                    </div>
                    <div class="d-grid">
                        <button class="btn btn-outline-primary btn-lg" id="recuperar-contraseña" v-if="resetPassword.error"
                            @click.prevent="resetPassword.error = false; resetPassword.sent = false; resetPassword.email = ''"
                            :disabled="loading">
                            <BootstrapIconArrowCounterclockwise />
                            Volver a intentar
                        </button>

                        <button class="btn btn-outline-primary btn-lg" id="recuperar-contraseña"
                            v-else-if="!resetPassword.sent" @click.prevent="
                                loading = true;
                            sendPasswordResetEmail(auth, resetPassword.email).then(() => {
                                resetPassword.sent = true;
                                loading = false;
                            }).catch(error => { resetPassword.error = true; console.log(error) })" :disabled="loading">
                            <span v-if="loading" class="spinner-border spinner-border-sm" role="status"> </span>
                            <span v-else>
                                <BootstrapIconKeyFill />
                                Recuperar contraseña
                            </span>
                        </button>

                        <button class="btn btn-success btn-lg disabled" id="recuperar-contraseña" v-else disabled>
                            <BootstrapIconCheckCircle />
                            Enlace enviado
                        </button>
                    </div>
                </form>
            </template>
        </BModal>

        <BToast title="Error" message="Usuario o contraseña incorrectos" :success="false" colour="danger"
            v-if="loginError" />
    </ClientOnly>
</template>
