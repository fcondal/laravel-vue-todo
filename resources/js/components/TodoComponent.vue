<template>
    <div class="w-50">
        <form @submit.prevent="saveData">
            <div class="input-group mb-3 w-100">
                <input
                    v-model="form.title"
                    :class="{'is-invalid' : form.errors.has('title')}"
                    type="text"
                    class="form-control form-control-lg"
                    @keydown="form.errors.clear('title')"
                    aria-label="Recipient's username"
                    aria-describedby="button-addon2">
                <div class="input-group-append">
                    <button class="btn btn-success" type="submit" id="button-addon2">Add this to your list</button>
                </div>
            </div>
            <div class="text-center">
                <span class="text-danger pt-3 pb-3 text-center" style="font-size:20px;" v-if="form.errors.has('title')" v-text="form.errors.get('title')"></span>
            </div>
        </form>
        <div class="w-100">
                <table class="table w-100 text-center">
                    <thead>
                        <tr>
                            <th>Status</th>
                            <th>Todo</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="todo in this.todos" :key="todo.id">
                            <td v-if="todo.completed">Completed</td>
                            <td v-else>Pending</td>

                            <td v-if="editMode == false || editMode != todo.id">{{ todo.title }}</td>
                            <td v-if="editMode == todo.id">
                                <input type="text" v-model="todo.title">
                            </td>
                            <td>
                                <button v-on:click="toggleTodo(todo)" v-if="!todo.completed" class="btn btn-sm btn-success">Mark as completed</button>
                                <button v-on:click="toggleTodo(todo)" v-else class="btn btn-sm btn-primary">Mark as pending</button>

                                <button v-on:click="updateTodo(todo)" class="btn btn-sm btn-warning">{{ (editMode == todo.id) ? 'Save' : 'Edit' }}</button>

                                <button v-on:click="deleteTodo(todo)" class="btn btn-sm btn-danger">Remove</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                editMode: false,
                todos: '',
                form: new Form({
                    title: '',
                })
            }
        },
        methods: {
            getTodos() {
                axios.get('api/todo').then((res) => {
                    this.todos = res.data;
                }).catch((error) => {
                    console.log(error);
                })
            },
            saveData() {
                let data = new FormData();
                data.append('title', this.form.title)
                axios.post('/api/todo', data).then((res) =>{
                    this.form.reset()
                    this.getTodos()
                }).catch((error) => {
                    console.log(error);
                    this.form.errors.record(error.response.data.errors)
                })
            },
            toggleTodo(todo){
                todo.completed = !todo.completed;
                let data = new FormData();
                data.append('_method', 'PATCH');
                if(todo.completed) {
                    data.append('completed', 1);
                } else {
                    data.append('completed', 0);
                }
                axios.post('/api/todo/' + todo.id, data);
            },
            updateTodo(todo){
                if(this.editMode != todo.id){
                    this.editMode = todo.id;
                } else {
                    this.editMode = false;
                    let data = new FormData();
                    data.append('_method', 'PATCH');
                    data.append('title', todo.title)
                    axios.post('/api/todo/' + todo.id, data)
                    .then(() =>{})
                    .catch((error) => {
                        console.log(error);
                        this.form.errors.record(error.response.data.errors)
                    })
                }
            },
            deleteTodo(todo){
                let data = new FormData();
                data.append('_method', 'DELETE');
                axios.post('/api/todo/' + todo.id, data)
                    .then((res) =>{
                        this.todos = res.data
                    })
                    .catch((error) => {
                        console.log(error);
                        this.form.errors.record(error.response.data.errors)
                    })
            }
        },
        mounted() {
            console.log('Component mounted.')
            this.getTodos();
        }
    }
</script>
