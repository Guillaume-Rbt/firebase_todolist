<template>
    <div class="container_todo">
        <div class="todo-header">

            <div class="newTodo_container">
                <input class="newTodo_field" type="text" placeholder="Ajouter une nouvelle tâche" v-model="newTodo"
                    @keyup.enter="addNewTodo" />
            <button class="btn btn-1" @click="addNewTodo"><span class="material-symbols-rounded">add </span></button>

            </div>

            <div class="field_group"><input type="checkbox" v-model="allCompleted" @change="handleAllCompleted" />
                <label>Cocher/décocher toutes les tâches</label>
            </div>
            <div class="filters">
                <a href="#" :class="{ active: filter == 'all' }" @click.prevent="filter = 'all'">Toutes les tâches</a>
                <a href="#" :class="{ active: filter == 'todo' }" @click.prevent="filter = 'todo'">Tâches à faire</a>
                <a href="#" :class="{ active: filter == 'todoCompleted' }"
                    @click.prevent="filter = 'todoCompleted'">Tâches
                    complétées</a>
            </div>
        </div>


        <ul class="todos_list">
            <li v-for="todo in filterTodoList" v-bind:key="todo.id" :class="{editing: todo === editTodo}"> <input
                    type="checkbox" v-model="todo.completed" @change="toggleCompleted(todo)"><span
                    @dblclick="editingTodo(todo)" :class="{ completed: todo.completed }">{{ todo.title }}</span><a
                    title="Supprimer la tâche" href="#" class="delete_todo" @click.prevent="deleteTodo(todo)"><span
                        class="material-symbols-rounded">delete</span></a>
                <input v-model="todo.title" type="text" @keyup.enter="doneEdit(todo)" class="edit" />
            </li>
        </ul>
        <div class="todo-footer">
            <p>{{ taskRemaining }} tâche(s) à faire</p>
        </div>
    </div>



</template>


<script>
import app from '../firebase'
import { collection, query, where, getDocs, getFirestore, doc, setDoc, deleteDoc, updateDoc } from "firebase/firestore";


export default {

    props: ['id'],
    data: function () {

        return {
            todosList: [],
            test: "test",
            newTodo: "",
            allCompleted: false,
            filter: 'all',
            editTodo: null,
            db: getFirestore(app),
        }
    },

    methods: {

        // Add new todo
        addNewTodo: function () {

            if (this.newTodo !== "") {
                var uniqID = this.newTodo + Date.now()
                // add in database
                setDoc(doc(this.db, "todos", uniqID), {
                    title: this.newTodo,
                    completed: false,
                    todoListID: this.id,
                    taskID: uniqID,
                });
                // add in array for display
                this.todosList.push({
                    title: this.newTodo,
                    completed: false,
                    todoListID: this.id,
                    taskID: uniqID,
                })
            }
            this.newTodo = "";
        },

        // update a todo 
        async doneEdit(todo) {
            await updateDoc(doc(this.db, "todos", todo.taskID), {
                title: todo.title
            })
            this.editTodo = null

        },

        // handle double click to edit todo. Add class to the todo to display the editing fieldd
        editingTodo: function (todo) {
            this.editTodo = todo;
        },

        // delete a todo (database and filter todos array)
        async deleteTodo(todo) {
            this.todosList = this.todosList.filter(a => a !== todo);
            await deleteDoc(doc(this.db, "todos", todo.taskID))
        }
        ,

        // handle all completed (or not)
        async handleAllCompleted() {
            if (this.allCompleted) {
                this.todosList.forEach(todo => {
                    todo.completed = true;
                    updateDoc(doc(this.db, "todos", todo.taskID), {
                        completed: true
                    })
                });
            } else {
                this.todosList.forEach(todo => {
                    todo.completed = false;
                    updateDoc(doc(this.db, "todos", todo.taskID), {
                        completed: false
                    })
                });
            }
        },

        // Update completed status of a todos
        async toggleCompleted(todo) {
            await updateDoc(doc(this.db, "todos", todo.taskID), {
                completed: todo.completed
            })
        }
    },

    computed: {

        // count the remaining tasks in the todi list
        taskRemaining() {
            var TodoRemaining = this.todosList.filter(function (todo) {
                return !todo.completed
            })
            return TodoRemaining.length
        },

        // filter todo list
        filterTodoList() {
            if (this.filter == 'all') {
                return this.todosList
            } else if (this.filter == 'todo') {
                return this.todosList.filter(todo => !todo.completed);
            } else {
                return this.todosList.filter(todo => todo.completed);
            }
        }
    },

    // get the todos when the component is mounted
    async mounted() {
        var q = query(collection(this.db, "todos"), where("todoListID", "==", this.id))
        const querySnapshot = await getDocs(q);
        querySnapshot.forEach((doc) => {
            this.todosList.push(doc.data())
        });
    }
}

</script>


<style >
.container_todo {
    width: 100%;
    background-color: #242424;
    padding: 10px;
    border-radius: 0 0 10px 10px;
}

.newTodo_container {
    display: flex;
}

.newTodo_field {
    padding: 10px 10px 10px 0;
    font-size: 20px;
    outline: none;
    background: transparent;
    border: none;
    color: #F1F1F1;
    width: calc(100% - 34px);
}

.field_group {
    display: flex;
    align-items: center;
    margin: 10px 0 10px 0;
}

.field_group input {
    margin-right: 10px;
}

.field_group input:checked {
    margin-right: 10px !important;
}

.filters {
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-content: center;
    margin: 20px 0 0;
}

.filters a {
    color: #F1F1F1;
    background-color: #191919;
    text-decoration: none;
    display: inline-block;
    transition: all 0.3s;
    padding: 10px;
    flex: 1 1 0;
    text-align: center;
}

.filters a.active {
    background-color: #535353;
}

.filters a:nth-child(1) {
    border-top-left-radius: 10px;
}

.filters a:nth-child(3) {
    border-top-right-radius: 10px;
}

.todos_list {
    list-style: none;
    padding: 20px 0 0;
    background-color: #191919;
    border-radius: 0 0 10px 10px;
    margin-bottom: 10px;
}

.todos_list li {
    padding: 10px;
    display: flex;
    align-items: center;
    gap: 10px;
    position: relative;
}

.delete_todo {
    position: absolute;
    right: 10px;
    color: #808080
}

.todos_list li span {
    display: inline-block;
    width: -webkit-max-content;
    width: -moz-max-content;
    width: max-content;
    position: relative;
}

.todos_list li span::before {
    content: "";
    width: 100%;
    height: 4px;
    position: absolute;
    top: calc(50% - 1px);
    background-color: #191919;
    transform: scaleX(0);
    transform-origin: left;
    transition: transform 0.3s;
}

.todos_list li span.completed {
    opacity: 0.5;
}

.todos_list li span.completed::before {
    transform: scaleX(1);
    transition: transform 0.3s;
}

input[type=checkbox] {
    -webkit-appearance: none;
    -moz-appearance: none;
    -ms-appearance: none;
}

input[type=checkbox] {
    border-radius: 4px;
    height: 24px;
    width: 24px;
    background: transparent;
    border: 1px solid #808080;
    transition: all 0.3s;
    position: relative;
    z-index: 12;
    overflow: hidden;
}

input[type=checkbox]:before {
    font-family: FontAwesome;
    content: "\e876";
    display: block;
    color: #808080;
    font-size: 25px;
    position: absolute;
    top: -4px;
    width: auto;
    overflow: hidden;
    transition: all 0.2s ease;
    z-index: 9;
}

input[type=checkbox]:after {
    content: "";
    position: absolute;
    display: block;
    width: 100%;
    height: 100%;
    background: #242424;
    z-index: 10;
    transition: all 0.1s;
}

.todos_list input[type=checkbox]:after {
    background-color: #191919;
}

input[type=checkbox]:checked {
    background: transparent;
    margin: 0;
    position: relative;
    overflow: hidden;
}

input[type=checkbox]:checked:before {
    font-family: FontAwesome;
    content: "\f00c";
    display: block;
    color: grey;
    font-size: 25px;
    position: absolute;
    top: -4px;
    width: auto;
    overflow: hidden;
    z-index: 9;
}

input[type=checkbox]:checked:after {
    content: "";
    position: absolute;
    display: block;
    width: 100%;
    height: 100%;
    background: #333;
    z-index: 10;
    transform: translateX(110%);
}

.edit {
    width: calc(100% - 80px);
    position: absolute;
    padding: 5px;
    margin-left: 34px;
    background: #202020;
    border: none;
    outline: none;
    color: #F1F1F1;
    font-size: 25px;
    display: none;
    border: 1px solid #f1f1f1;
    transition: all 0.3s;
}

.editing .edit {
    display: block;
}
</style>