<template>
  <div>
<!-- {{socket}} -->
  <!-- <h1>{{ message }}</h1> -->

  <loginPage
  :page="page" 
  v-on:change-Page="changePage"
  v-on:fetch-Project="panggil"
  ></loginPage>

  <registerPage
  :page="page" 
  v-on:change-Page="changePage"
  ></registerPage>
  <navbarHome
  :page="page" 
  v-on:change-Page="changePage"
  ></navbarHome>

  <homePage
  :page="page" 
  :socket="socket"
  :projects="projects"
  v-on:change-Page="changePage"
  v-on:project-Page="projectPage"
  ></homePage>

  <projectPage
  :page="page" 
  :kanbanTasks="kanbanTasks"
  :currentProject="currentProject"
  v-on:change-Page="changePage"
  v-on:project-Page="projectPage"
  ></projectPage>

  <!-- <testDrag/> -->
  <!-- <form @submit.prevent="addTask">
      <h3>Add Task</h3>
      <label for>masukan title</label>
      <br/>
      <input type="text" v-model="title" />
      <br/>
      <label for>masukan assignee</label>
      <br />
      <input type="text" v-model="assignee" />
      <br/>
      <label for>masukan desc</label>
      <br />
      <input type="text" v-model="desc" />
      <button type="submit">submit</button>
      <br />
    </form>
    {{tasks}} 
    <br/>
   ini data baru {{baru}} -->
  </div>
</template>

<script>
import testDrag from './components/testDrag'
import loginPage from './components/loginPage'
import registerPage from './components/registerPage'
import navbarHome from './components/navbarHome'
import homePage from './components/homePage'
import projectPage from './components/projectPage'
import io from 'socket.io-client'
import axios from './api/axiosInstance'
export default {
  data() {
    return {
      message: 'Hello world',
      socket: io.connect('http://localhost:3000', {reconnect: true}),
      tasks:[],
      title:'',
      assignee:'',
      desc:'',
      status:'backlog',
      newTask: {},
      baru:'',
      page:'login',
      currentProject: {},
      kanbanTasks:[],
      projects:[]
    };
  },
  components:{
    loginPage,
    registerPage,
    homePage,
    navbarHome,
    projectPage,
    testDrag
  },
  methods:{
    panggil(laman){
      console.log('kepanggil fetch dari login', laman);
      
    },
    changePage(laman){
      if(laman == 'home'){
        // console.log('dari root', laman)
        this.fetchProject()
      }
      this.page = laman
      // this.getArticle()
    },
    projectPage(payload){
      // console.log('ini masuk');
      
      // console.log(id, 'ini dari app');
      
      this.currentProject = payload
    },
    addTask(){
      let obj = {
        title: this.title,
        assignee: this.assignee,
        desc: this.desc,
        status: this.status
      }
      // console.log(obj);
      
      axios({
        method:'post',
        url:'/task/',
        data: obj
      })
      .then(({data})=>{
        // console.log(data);
        // this.fetch() 
      })
      .catch(err =>{
        console.log(err);
        
      })
    },
    fetchTask(){
      axios({
        method:'get',
        url:'/task',
      })
      .then(({data})=>{
        this.kanbanTasks = data
      })
      .catch(err =>{
        console.log(err);
      })
    },
    fetchProject() {
      axios({
        method: "get",
        url: "user/project",
        headers:{
          token: localStorage.token
        }
      })
        .then(({ data }) => {
          this.projects = data;
          // console.log('fetch baru nih');
          
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
 created: function () {
  //  console.log(this.socket);
  if(localStorage.token){
    this.fetchTask()
    this.fetchProject();

    this.page='home'

    this.socket.on("project", data => {
        // console.log(data, 'ini dari io');
        this.projects.push(data);
      });
    this.socket.on("fetchProject", data => {
        // console.log('kepanggil');
        this.fetchProject();
      });

    this.socket.on('news', (data) => {
      // console.log(data, 'ini dari io');
      this.kanbanTasks.push(data)
      });
      this.socket.on('fetchTask', (data) => {
      // console.log(data, 'ini dari io');
      this.fetchTask()
      });
    }
  }
};
</script>

<style scoped>
</style>