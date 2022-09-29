<template>
    <div class="container_login">
        <h2 class="title_login">Connexion</h2>
        <form @submit.prevent="handleLogin">
            <div class="form_row">
                <label for="email">Email</label>
                <input type="email" name="email" v-model="loginEmail">
            </div>
            <div class="form_row">
                <label for="password">Mot de passe</label>
                <input type="password" name="password" v-model="loginPassword">
            </div>
            <div class="form_row">
                <button type="submit">Se connecter</button>
            </div>

        </form>

    </div>

</template>

<script>

import { auth } from '../firebase'
import { signInWithEmailAndPassword } from '@firebase/auth';





export default {
    data() {
        return {
            loginEmail: "",
            loginPassword: "",
        }
    },

    methods: {
        handleLogin: function () {
            try {
                signInWithEmailAndPassword(auth, this.loginEmail, this.loginPassword)
                this.$emit("authenticated")

            }
            catch (error) {
                console.log(error)
            }
        }
    },
}
</script>

<style>
.form_row {
    display: flex;
    flex-direction: column;
    margin-top: 10px;
    gap: 5px
}
</style>