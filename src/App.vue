<template>
  <main>
    <header>
      <h1>My Tasks</h1>
      <div class="userName-logout" v-if="this.isAuthenticated">
        <p class="userName"><span class="initiale"> {{ currentUser.displayName[0] }}</span> {{ currentUser.displayName
        }} <span title="Se déconnecter" class="material-symbols-rounded logout" @click="handleLogout">logout</span>
        </p>
      </div>
    </header>

    <transition name="fadeModal">
      <modal_login v-if="!this.isAuthenticated" @authenticated="authentification"></modal_login>
    </transition>
    <input type="text" placeholder="Ajouter une liste de tâches" class="newTodoList" v-model="newTodoList"
      @keyup.enter="addTodoList" v-if="this.isAuthenticated">

    <div class="todoList_container" v-if="this.isAuthenticated">
      <div class="todoList_item" v-for="list in allTodoList" :key="list.id">
        <h3 class="list_title"> {{ list.title }} <a href="#" title="Supprimer la liste"
            @click.prevent="deleteList(list)"><span class="material-symbols-rounded">delete</span></a> </h3>
        <todo_list :id="list.todoID"> </todo_list> <!-- props Id of the todolist to get the todos in the component  -->
      </div>

    </div>
  </main>
</template>


<script>
import modal_login from './components/modal_login.vue'
import { getAuth, app, onAuthStateChanged, signOut } from 'firebase/auth';
import { auth } from './firebase'
import { collection, query, getDocs, getFirestore, doc, setDoc, where, deleteDoc } from "firebase/firestore";
import todo_list from './components/todo_list.vue'

export default {
  name: 'app',
  data() {
    return {
      isAuthenticated: false, // true if a user is connected
      currentUser: {},
      allTodoList: [],
      newTodoList: "",
      db: getFirestore(app)
    }
  },
  methods: {

    // Get the current user after login or sign up (event 'authenticated')
    authentification: function () {
      onAuthStateChanged(auth, (user) => {
        if (user) {
          this.currentUser = user
          this.isAuthenticated = true
        }
      })
    },


    async addTodoList() {
      if (this.newTodoList !== "") {
        var uniqID = this.newTodoList + Date.now() // Generate a unique ID 
        await setDoc(doc(this.db, "todo-List", uniqID), { // register in database the new todo-list
          title: this.newTodoList,
          todoID: uniqID,
          userID: this.currentUser.uid
        });
        // add the new todo-list in the array for display it in real time
        this.allTodoList.push({
          title: this.newTodoList,
          todoID: uniqID,
          userID: this.currentUser.uid
        })
      }
      this.newTodoList = "";
    },

    // get the todo list of current user with the id
    async getLists() {
      var q = query(collection(this.db, "todo-List"), where("userID", "==", this.currentUser.uid))
      var querySnapshot = await getDocs(q);
      querySnapshot.forEach((doc) => {
        this.allTodoList.push(doc.data())
      });
    },

    // logout the current user and reload 
    async handleLogout() {
      await signOut(auth);
      window.location.href = "/";
    },

    // Delete a todo list with her task
    async deleteList(list) {
      // first delete all todos of the todo list in the data base
      var todos = query(collection(this.db, "todos"), where('todoListID', "==", list.todoID))
      var queryTodos = await getDocs(todos);
      queryTodos.forEach((task) => {
        deleteDoc(doc(this.db, "todos", task.data().taskID))
      })
      // delete the todo lit in db and filter the allTodoList array to not display it
      deleteDoc(doc(this.db, "todo-List", list.todoID));
      this.allTodoList = this.allTodoList.filter(a => a !== list);
    }
  },

  // Verify if a user is authenticated on mounted application 
  mounted: function () {
    var auth = getAuth();
    onAuthStateChanged(auth, (user) => {
      if (user) {
        this.currentUser = user
        this.getLists();
        this.isAuthenticated = true
      }
    })

  },

  components: {
    modal_login,
    todo_list
  },

}
</script>

<style>
:root {
  --blue: #00a2d4;
}

*,
::before,
::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body,
html {
  overflow-x: hidden;
}

body {
  background: #303030;
  color: #f1f1f1;
  font-family: Helvetica, sans-serif;
  padding-bottom: 100px;
}

header {
  width: 100%;
  margin: 50px 0 50px;
  display: grid;
  grid-template-columns: repeat(3 , 1fr);
}

h1 {
  text-align: center;
  width: 100%;
  grid-column: 2/3;
  font-size: clamp(45px, 7vw, 80px);
}

.userName-logout {
  padding-right: 10px;
  width: 100%;
  grid-column: 3/4;
}

.userName-logout p {
  width: 100%;
  line-height: 50px;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  align-items: center;
  font-size: 20px;
  text-transform: capitalize;
}

.userName .initiale {
  display: inline-block;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: var(--blue);
  line-height:40px;
  text-align: center;
  font-size: 30px;
  text-transform: uppercase;
}

.logout {
  cursor: pointer;

}

.list_title {
  position: relative;
  background-color: #242424;
  padding: 10px 10px 0 10px;
  border-radius: 10px 10px 0 0;
  font-size: 25px;
}

.list_title a {
  position: absolute;
  right: 10px;
  color: #808080;
}

.fadeModal-enter-active,
.fadeModal-leave-active {
  transition: opacity 0.3s ease;

}

.fadeModal-enter-from,
.fadeModal-leave-to {
  opacity: 0;
}

.todoList_container {
  width: 100%;
  display: flex;
  justify-content: space-evenly;
  gap: 25px;
  row-gap: 50px;
  flex-wrap: wrap;
}

.todoList_item {
  width: clamp(300px, 50%, 500px)
}

.newTodoList {
  display: block;
  margin: 50px auto;
  padding: 10px 10px 10px 0;
  font-size: 20px;
  outline: none;
  background: transparent;
  border: none;
  color: #F1F1F1;
  width: clamp(300px, 50%, 500px);
  border-bottom: 1px solid #808080;
}

@media screen and (max-width:730px) {
  

  h1, .logout {
    grid-column: 1/4;
    row-gap: 50px;
  }
}
</style>
