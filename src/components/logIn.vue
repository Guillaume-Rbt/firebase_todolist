<template>
    <div class="container_form">
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
        <div class="error">
            <p v-for="error in errors" v-bind:key="error.id"> {{ error }}</p>
        </div>
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
            errors:[],
        }
    },

    methods: {
        handleLogin: function () {
            try {
                signInWithEmailAndPassword(auth, this.loginEmail, this.loginPassword).then(()=> {this.$emit("authenticated")})
                .catch((error)=>{ 
                    this.errors = [];
                    console.log(error)
                    this.errors.push('L\'identifiant ou le mot de passe sont incorrects')})
                

            }
            catch (error) {
                console.log('error')
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