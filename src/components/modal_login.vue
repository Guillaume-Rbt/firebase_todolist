<template>
    <div class="login_container">
        <div class="login_container_nav">
            <ul class="nav_login">
                <li :class="this.filterActive ? 'active' : ''" @click="clickLogIn">Se connecter</li>
                <li :class="this.filterActive ? '' : 'active'" @click="clickSignIn">S'inscrire</li>
            </ul>
        </div>
        <div class="form_container">
            <transition name="fade">
                <log-in v-if="this.filterNav==='logIn'" @authenticated="authentification"></log-in>
            </transition>
            <transition name="fade">
                <sign-up v-if="this.filterNav==='signUp'" @authenticated="authentification"></sign-up>
            </transition>
        </div>


    </div>
</template>

<script>
import logIn from './logIn.vue'
import signUp from './signUp.vue'

export default {
    data() {
        return {
            filterNav: 'logIn',
            filterActive: true,
        }
    },

    components: {
        logIn,
        signUp
    },

    methods: {
        clickSignIn: function () {
            this.filterNav = 'signUp';
            this.filterActive = false
        },

        clickLogIn: function () {
            this.filterNav = 'logIn';
            this.filterActive = true;

        }, 

        authentification: function() {
            this.$emit("authenticated") 
        }
    }, 
    
}
</script>



<style>

.login_container {
    width:80%;
    margin: auto;
    min-width: 300px;
}
.nav_login {
    list-style: none;
    display: flex;
}

.nav_login li {
    cursor: pointer;
    padding: 10px;
    background-color: #191919;
}

.nav_login li:nth-child(1) {
    border-radius: 10px 0 0 0;
}
.nav_login li:nth-child(2) {
    border-radius: 0 10px 0 0;
}

.nav_login li.active {
    background-color: #535353;
}

.form_container {
    background-color: #242424;
    border-radius: 0 10px 10px 10px;
    padding: 10px;
    height: 400px ;
    width:100%;
    margin: auto;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
  position: absolute;

}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.container_login, .container_signUp {
  width: 50%;
  min-width: 250px;
  height: calc(100% - 20px);
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.form_row input {
    padding: 5px ;
    border-radius: 5px;
    outline: none;
    border: none;
}
.title_login {
    margin-bottom: 10px;
}

.form_row button {
    padding: 5px ;
    border-radius: 5px;
    outline: none;
    border: none;
    background-color: var(--blue);
    font-size: 16px;
    width:50%;
    cursor:pointer;
    margin-top: 10px;
    margin-bottom: 10px;
}

</style>