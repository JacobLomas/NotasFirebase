<template>
  <div id="app">    
    <notifications/>
    <div class="inputs">
      <div class="auth" >
        <h4 v-if="authenticated">Bienvenido {{firstName}}</h4>
        <button @click="login" v-if="!authenticated">Login <i class="fab fa-google"></i></button>
        <button v-if="authenticated" @click="logout">Logout <i class="fas fa-sign-out-alt"></i></button>
      </div>
      
      <input @keyup.enter="crearNota" v-model="texto" type="text" name="name" class="question" id="nta" required autocomplete="off" />
      <label for="nta"><span>Nuevo </span></label>
      
      <input v-model="filtro" type="text" name="name" class="question" id="fltr" required autocomplete="off" />
      <label for="fltr"><span>Filtrar</span></label>
      <h4>{{tareasPendientes}} pendientes de {{totalTareas}} || <span class="borrarCompletados" @click="borrarTareasCompletadas" title="Borra las tareas completadas"><i class="far fa-times-circle"></i> Borrar tareas completadas</span></h4>
    </div>
    <section class="notas">
      <nota v-for="(nota) in listaRecordatoriosFiltrada" :key="nota.id"
       @actualizarQuery="bindQuery"
       :nota="nota"/>
    </section>
    
  </div>
</template>

<script>
import nota from './components/nota/nota.vue'

import { db } from './db'
import Firebase from './db'
import firebase from 'firebase/app'
export default {
  name: 'App',
  mounted(){
    Firebase.auth.onAuthStateChanged( user => {
        if (user) {
          this.user.loggedIn = true;
          this.user.data = user;
          this.$bind('recordatorios', db.collection('recordatorios').where('uid','==',this.user.data.uid))
        }
        else {
          this.user.loggedIn = false;
          this.user.data = {};
          this.$bind('recordatorios', db.collection('recordatorios').where('uid','==',''))
        }
      })
  },
  firestore(){
    return{
      recordatorios: db.collection('recordatorios').where('uid','==',this.user.data.uid)
    }
  },
  data(){
    return{
      user: {
          loggedIn: false,
          data: {
            uid:""
          }
        },
      texto:"",
      filtro:"",
      recordatorios:[]
    }
  },
  methods:{
    bindQuery(){
      //Sin el bind no se me actualiza de forma correcta la vista
      this.$bind('recordatorios', db.collection('recordatorios').where('uid','==',this.user.data.uid))
    },
    crearNota(){
      db.collection('recordatorios').add({
        titulo: this.texto,
        fechaCreacion: firebase.firestore.FieldValue.serverTimestamp(),
        prioridad:0,
        completado:false,
        uid:this.user.data.uid
      })
      this.texto=""
    },
    borrarTareasCompletadas(){
      var lista = this.recordatorios.filter((recordatorio)=>{
        return recordatorio.completado
      });
      lista.forEach(recordatorio => {
        db.collection('recordatorios').doc(recordatorio.id).delete()
        .then(()=>{
          this.bindQuery()
        }).catch(()=>{
          this.$notify({
            type:'error',
            text:'No se ha podido eliminar el recordatorio'
          })
        });
      });
    },
    login(){
      Firebase.login();
    },
    logout(){
      Firebase.logout()
    }
  },
  computed:{
    authenticated(){
          return this.user.loggedIn;
        },
    firstName(){
      if (this.user.data.displayName) {
        return this.user.data.displayName.split(' ')[0]
      }
      return null
    },
    totalTareas(){
      return this.recordatorios.length
    },
    tareasPendientes(){
      let cont=0;
      for(let recordatorio of this.recordatorios){
        if(!recordatorio.completado)
            cont++
      }
      return cont;
    },
    listaRecordatoriosFiltrada(){
      var lista;
      if(this.filtro.length==0)
          lista= this.recordatorios;
      else{
          lista = this.recordatorios.filter((recordatorio)=>{
              return recordatorio.titulo.indexOf(this.filtro)>=0;
          });
      }      
      lista=lista.sort((recordatorio1, recordatorio2)=>{
          if(recordatorio1.prioridad<recordatorio2.prioridad)
              return 1;
          else
              return -1;
      })
      return lista 
    }
  },
  components: {
    nota
  },
  
}
</script>

<style>
body{
  background-color: rgb(82, 82, 82);
  color: wheat;
}
.auth{
  width: 40%;
  margin-left: auto;
  margin-right: auto;
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  align-items: center;
  color: wheat;
}
.auth button{
  padding: 8px;
  border-radius: 7%;
  background-color: wheat;
  border: 1px solid black;
  cursor: pointer;
  margin-left: 1vw;
}
.auth button:hover{
  background-color: rgb(223, 201, 160);
}
h4{
  text-align: left;
  color: wheat;
}
.borrarCompletados{
  color: lightcoral;
  cursor: pointer;
}
.notas, .inputs{
  width: 100%;
  margin: auto;
}
.notas{
  border: 3px solid rgba(128, 128, 128, 0.616);
}
input,
label {
  font-family: 'Ubuntu', sans-serif;
  display: block;
  margin: 10px;
  padding-top: 5px;
  padding-bottom: 5px;
  border: none;
  font-size: 1rem;
}

input:focus {
  outline: 0;
}
/* Question */

input.question {
  color: wheat;
  font-size: 48px;
  font-weight: 300;
  border-radius: 2px;
  margin: 0;
  border: none;
  width: 100%;
  background: rgba(0, 0, 0, 0);
  transition: padding-top 0.2s ease, margin-top 0.2s ease;
  overflow-x: hidden; /* Hack to make "rows" attribute apply in Firefox. */
}
/* Underline and Placeholder */

input.question + label {
  display: block;
  position: relative;
  white-space: nowrap;
  padding: 0;
  margin: 0;
  width: 10%;
  border-top: 1px solid wheat;
  color: wheat;
  -webkit-transition: width 0.4s ease;
  transition: width 0.4s ease;
  height: 0px;
}

input.question:focus + label {
  width: 80%;
}

input.question:focus,
input.question:valid {
  padding-top: 35px;
}


input.question:focus + label > span,
input.question:valid + label > span {
  top: -100px;
  font-size: 2rem;
  color: wheat;
}

textarea.question:focus + label > span {
  top: -150px;
  font-size: 2rem;
  color: wheat;
}


input.question:invalid {
  box-shadow: none;
}

input.question + label > span {
  font-weight: 300;
  margin: 0;
  position: absolute;
  color: wheat;
  font-size: 48px;
  top: -66px;
  left: 0px;
  z-index: -1;
  -webkit-transition: top 0.2s ease, font-size 0.2s ease, color 0.2s ease;
  transition: top 0.2s ease, font-size 0.2s ease, color 0.2s ease;
}


@-webkit-keyframes appear {
  100% {
    opacity: 1;
  }
}

@keyframes appear {
  100% {
    opacity: 1;
  }
}

@media(min-width: 578px){
 .notas, .inputs{
  width: 70vw;
  } 
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
