<script>
const STORAGE_KEY = 'vue-todomvc'

const filters = {
    all: (todos) => todos,
    active: (todos) => todos.filter((todo) => !todo.completed),
    completed: (todos) => todos.filter((todo) => todo.completed)
}

export default {
    // app initial state
    data: () => ({
        todos: JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'),
        editedTodo: null,
        visibility: 'all'
    }),

    // watch todos change for localStorage persistence
    watch: {
        todos: {
            handler(todos) {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
            },
            deep: true
        }
    },

    mounted() {
        window.addEventListener('hashchange', this.onHashChange)
        this.onHashChange()
    },

    computed: {
        filteredTodos() {
            return filters[this.visibility](this.todos)
        },
        remaining() {
            return filters.active(this.todos).length
        }
    },

    // methods that implement data logic.
    // note there's no DOM manipulation here at all.
    methods: {
        toggleAll(e) {
            this.todos.forEach((todo) => (todo.completed = e.target.checked))
        },

        addTodo(e) {
            const value = e.target.value.trim()
            if (!value) {
                return
            }
            this.todos.push({
                id: Date.now(),
                title: value,
                completed: false
            })
            e.target.value = ''
        },

        removeTodo(todo) {
            this.todos.splice(this.todos.indexOf(todo), 1)
        },

        editTodo(todo) {
            this.beforeEditCache = todo.title
            this.editedTodo = todo
        },

        doneEdit(todo) {
            if (!this.editedTodo) {
                return
            }
            this.editedTodo = null
            todo.title = todo.title.trim()
            if (!todo.title) {
                this.removeTodo(todo)
            }
        },

        cancelEdit(todo) {
            this.editedTodo = null
            todo.title = this.beforeEditCache
        },

        removeCompleted() {
            this.todos = filters.active(this.todos)
        },

        onHashChange() {
            var visibility = window.location.hash.replace(/#\/?/, '')
            if (filters[visibility]) {
                this.visibility = visibility
            } else {
                window.location.hash = ''
                this.visibility = 'all'
            }
        }
    }
}
</script>

<template>
    <div class="container mt-5">
        <div class="col-6">
            <form class="d-flex justify-content-center align-items-center mb-4">
                <div class="form-outline flex-fill">
                    <input type="text" class="form-control" placeholder="Add a new task and press Enter"
                        @keyup.enter="addTodo" />
                </div>
            </form>
            <div class="card text-center">
                <div class="card-header">
                    <ul class="nav nav-tabs card-header-tabs">
                        <li class="nav-item">
                            <a href="#/all" class="nav-link active" aria-current="true"
                                :class="{ selected: visibility === 'all' }">All</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#/active" :class="{ selected: visibility === 'active' }">Active</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#/completed"
                                :class="{ selected: visibility === 'completed' }">Completed</a>
                        </li>
                    </ul>
                </div>
                <div class="card-body">
                    <div class="tab-content" id="ex1-content">
                        <div class="tab-pane fade show active" id="ex1-tabs-1" role="tabpanel" aria-labelledby="ex1-tab-1">
                            <ul class="list-group mb-0" v-for="todo in filteredTodos" :key="todo.id">
                                <li class="list-group-item d-flex align-items-center border-0 mb-2 rounded"
                                    style="background-color: #f4f6f7;">
                                    
                                    <div class="input-group">
                                        <input class="form-check-input me-2" type="checkbox" v-model="todo.completed" />
                                        <p>{{ todo.title }}</p>
                                    </div>
                                    <button v-if="todo.completed" class="btn-sm btn-outline-danger" @click="removeTodo(todo)"><i class="bi bi-trash"></i></button>
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

