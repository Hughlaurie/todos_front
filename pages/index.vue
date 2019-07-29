<template>
  <div>
    <h2>Todos:</h2>
    <input v-model="texttask" Placeholder="Новая задача">
    <ol>
      <li v-for="(todo,idx) in todos">
        <label>
          <input type="checkbox"
           @change="toggle(todo)"
           :checked="todo.done">
          <input
            v-if="edittext[todo.id] && edittext[todo.id].isEdit"
            v-model="edittext[todo.id].text"
            placeholder="Что нужно сделать?"
            >
          <span v-else :class="{done : todo.done}">{{ todo.text }}</span>
          <div v-if="!(edittext[todo.id] && edittext[todo.id].isEdit)">
            <button class="edittask" @click="editTask(todo)">Edit Tesk</button>
            <button class="deltask" @click="deleteTaskFromServer(todo)">Delete Task</button>
          </div> 
          <template v-else>
            <button class="canceltask" @click.prevent="cancelEdit(todo)">Cancel</button>
            <button class="ok" @click.prevent="saveEditText(edittext[todo.id])">Ok</button>
          </template>
        </label>
      </li>
    </ol>
    <button class="addtask" @click="addTask()">Add Task</button>
    <button class="delalltask" @click="deleteAllDoneTaskFromServer(todos)">Delete All Task</button>
  </div>
</template>

<script>
  const axios = require('axios');
  export default {

    data() {
      return {
        texttask: '',
        edittext: {},
        todos: [],
      }
    },

    watch: {
      texttask(temptexttask) {
        localStorage.texttask = temptexttask;
      }
    },
  
    methods: {
      async toggle(todo) {
        todo.done = !todo.done
        await axios.put(`http://localhost:3001/todos/${todo.id}`, todo);
        this.loadDataServer();
      },

      async addTask() {
        if (!this.texttask) {return};
        var text = {id: 0, text: this.texttask, done: false};
        this.addDataServer(text);
      },

      async deleteTaskFromServer(todo) {
        if (!todo.done) return;
        await axios.delete(`http://localhost:3001/todos/${todo.id}`);
        this.loadDataServer();
        
      },

      async deleteAllDoneTaskFromServer(todos) {
        await axios.delete(`http://localhost:3001/todos/`);
        this.loadDataServer();
        //this.todos = this.todos.filter(todo => !todo.text)
      },

      editTask(todo) {
        this.$set(this.edittext, todo.id, Object.assign({isEdit: true}, todo));

      },

      cancelEdit(todo) {
        this.edittext[todo.id].isEdit = false;

      },

      async saveEditText(todo) {
        if (!todo.text){return;}
        delete todo.isEdit;
        await axios.put(`http://localhost:3001/todos/${todo.id}`, todo)
        this.loadDataServer();
        //this.edittext[todo.id].isEdit = false;
        //todo.text = this.edittext[todo.id].text;
      },

      async loadDataServer() {
        let getData = await axios.get('http://localhost:3001/todos');
        this.todos = getData.data;
      },

      async addDataServer(todo) {
        await axios.post('http://localhost:3001/todos', todo);
        this.loadDataServer();
      }

    },

    mounted() {
      this.loadDataServer();
      console.log (this.todos);
      if (localStorage.texttask) {
        this.texttask = localStorage.texttask;
      }      
    }
  }

</script>

<style scoped>
  body {
    background: #20262E;
    padding: 20px;
    font-family: Helvetica;
  }

  #app {
    background: #fff;
    border-radius: 4px;
    padding: 20px;
    transition: all 0.2s;
  }

  li {
    margin: 8px 0;
  }

  h2 {
    font-weight: bold;
    margin-bottom: 15px;
  }

  del {
    color: rgba(0, 0, 0, 0.3);
  }

  .done {
    color: rgba(0, 0, 0, 0.3);
    text-decoration: line-through;
  }

  .edittask,
  .ok,
  .canceltask {
    float: right;
    margin-right: 500px;
  }

  .ok {
    margin-right: 5px;
  }

</style>