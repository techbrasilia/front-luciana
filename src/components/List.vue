<template>
  <div class="list">
      <div class="navbar-brand">
        <button class=" navbar-item button" v-on:click="createTask()">Adicionar tarefa</button>
      </div>

    <div class="box" v-for="(task, index) in orderedTasks" >
      <article class="media" >
        <div class="media-content">
          <div class="content">
            <p>
              <span class="tag" v-bind:class="checkClass(task)">{{task.stage.name}}</span> <strong>{{task.title}}</strong>
              <br>
              {{task.description}}
            </p>
          </div>
          <nav class="level is-mobile" v-if="getNextStep(task)">
            <div class="level">
              <a class="level-item">
                <a class="button is-small" v-on:click="moveTo(task, getNextStep(task), index)">move to {{getNextStep(task).name}}</a>
              </a>
            </div>
          </nav>
        </div>
      </article>
    </div>
    </ul>
    <div class="modal"  v-bind:class="{ 'is-active': modalActive }">
      <div class="modal-background"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">Nova tarefa</p>
        </header>
        <section class="modal-card-body">
          <label class="label">Titulo</label>
          <div class="control">
            <input class="input" type="text" placeholder="" v-model="newTask.title">
          </div>
          <div class="field">
            <label class="label">Descrição</label>
            <div class="control">
              <textarea class="textarea" placeholder="" v-model="newTask.description"></textarea>
            </div>
          </div>
        </section>
        <footer class="modal-card-foot">
          <button class="button is-success" v-on:click="saveTask()">Adicionar</button>
          <button class="button" v-on:click="cancelCreation()">Cancelar</button>
        </footer>
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash';
export default {
  name: 'list',
  created () {
    this.fetchData();
  },
  computed: {
    orderedTasks: function () {
      return _.orderBy(this.tasks, ['stage', 'id'])
    }
  },
  data () {
    return {
      tasks: [],
      stages: [],
      newTask: {
        title: "",
        description: ""
      },
      modalActive: false,

    }
  },
  methods: {
    checkClass: function(task) {
      return {
        'is-light': task.stage.name == "To do",
        'is-info': task.stage.name == "Doing",
        'is-success': task.stage.name == "Done",
      }
    },
    getNextStep(task) {
      if (task.stage.id == this.stages[this.stages.length - 1].id) {
        return false;
      }

      let next = this.stages.find( (o, i) => {
          if (o.id == (task.stage.id + 1)){
          return o;
        }
      })
      return next;
    },
    moveTo(task, stage, index) {
      this.axios.put(`${process.env.API_URL}/tasks/${task.id}`, {
        stage_id: stage.id
      })
      .then((response) => {
        this.fetchData();
      })
      .catch( (error) => {});

    },
    createTask() {
      this.modalActive = true;
    },
    saveTask() {
      let task = {
        title:this.newTask.title,
        description:this.newTask.description,
        stage_id: 1
      }
      this.axios.post(`${process.env.API_URL}/tasks`,{task: task}).then( (response) => {
        this.fetchData();
        this.modalActive = false;
      })
    },
    cancelCreation() {
      this.newTask.title = "";
      this.newTask.description = "";
      this.modalActive = false;
    },
    fetchData () {
        this.axios.get(`${process.env.API_URL}/tasks`).then((response) => {
          this.tasks = response.data
        })
        this.axios.get(`${process.env.API_URL}/stages`).then((response) => {
          this.stages = response.data
        })
    }
  }
}
</script>

<style scoped>
</style>
