<template>
    <div class="container_form">
        <h2 class="title_login">Inscription</h2>
        <form method="post" @submit.prevent="register">
            <div class="form_row">
                <label for="username">Nom d'utilisateur</label>
                <input type="text" name="username" v-model="newUser.username" required>
            </div>
            <div class="form_row">
                <label for="email">Email</label>
                <input type="email" name="email" v-model="newUser.mail" required>
            </div>
            <div class="form_row">
                <label for="password">Mot de passe</label>
                <input type="password" name="password" v-model="newUser.password" required>
            </div>
            <div class="form_row">
                <label for="password_confirm">Confirmation du mot de passe</label>
                <input type="password" name="password_confirm" v-model="confirmPassword">
            </div>
            <div class="form_row">
                <button type="submit">S'inscrire</button>
            </div>
        </form>
        <div class="error">
            <p v-for="error in errors" v-bind:key="error.id"> {{ error }}</p>
        </div>
    </div>

</template>

<script>
import { auth } from '../firebase'
import { createUserWithEmailAndPassword, getAuth, updateProfile } from "firebase/auth";


export default {


    data() {
        return {
            newUser: {
                username: '',
                mail: '',
                password: ''
            },
            confirmPassword: '',
            errors: [],
        }
    },
    methods: {
        register: function () {
            this.errors = [] // reset errors array
            if (this.newUser.password !== this.confirmPassword) { //verify id password == confirm password
                this.errors.push('Le mot de passe et la confirmation doivent être identiques')
            }
            if (this.newUser.password.length < 6) { // verify the length of the password
                this.errors.push('Le mot de passe doit faire au moins 6 caractères')
            }

            if (this.errors.length === 0) { // if not errors ; register the user
                try {
                    createUserWithEmailAndPassword(auth, this.newUser.mail, this.newUser.password)
                        .then(async () => {
                            await updateProfile(getAuth().currentUser, { // set the displayName (username) in firebase of the created user
                                displayName: this.newUser.username
                            })
                            window.location.href = '/'
                            this.$emit("authenticated")
                        }
                        )
                } catch (error) {
                    console.log(error)
                }



            }
        }
    },
}

</script>

<style>

</style>