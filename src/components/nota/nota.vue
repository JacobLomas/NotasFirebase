<template>
  <article class="nota">
    <div  class="nota">
      <div class="cuerpo"> 
        <h5 class="tituloNota" :class="{completado: nota.completado}">
          <label class="check">
          <input type="checkbox" :checked=nota.completado @click="cambiarEstado"/>
          <div class="box"></div>
        </label>
          {{nota.titulo}}</h5>
      </div>
        
      <div class="delete">
        <i @click="borrarNota" class="fas fa-minus-circle"></i> 
      </div>
      <div class="prioridades">
        Prioridad: <button @click="cambiarPrioridad(0)" class="btn-pri" :class="{low: nota.prioridad==0}">Low</button>
        <button @click="cambiarPrioridad(1)" :class="{normal: nota.prioridad==1}" class="btn-pri">Normal</button>
         <button @click="cambiarPrioridad(2)" :class="{high: nota.prioridad==2}" class="btn-pri">High</button>
      </div>
      <div>
        <i class="fas fa-clock"></i> Added <timeago :datetime="nota.fechaCreacion.toDate()" :auto-update="60" :includeSeconds="true"></timeago>
      </div>
    </div>
  </article>
</template>
<script>
import {db} from '../../db.js'
export default {
  name: 'nota',
  data () {
    return {}
  },
  methods:{
    cambiarPrioridad(prioridad){
      var notaCopia= Object.assign({}, this.nota)
      if(prioridad==0)
        notaCopia.prioridad=0;
      if(prioridad==1)
        notaCopia.prioridad=1;
      if(prioridad==2)
        notaCopia.prioridad=2;
      db.collection('recordatorios').doc(this.nota.id).update(notaCopia)
        .then(()=>{
          this.$emit('actualizarQuery');
          this.$notify({
            title:'Prioridad',
            type:'info',
            text:'Prioridad actualizada'
          });
        }).catch((error)=>{
          console.log(error)
        })
    },
    cambiarEstado(){
      var notaCopia= Object.assign({}, this.nota)
      console.log(notaCopia)
      notaCopia.completado=!notaCopia.completado;
      db.collection('recordatorios').doc(this.nota.id).update(notaCopia)
        .then(()=>{
          this.$emit('actualizarQuery');
          this.$notify({
            title:'Prioridad',
            type:'info',
            text:'Prioridad actualizada'
          });
        }).catch((error)=>{
          console.log(error)
        })
    },
    borrarNota(){
      db.collection('recordatorios').doc(this.nota.id).delete()
      .then(()=>{
        this.$notify("Recordatorio eliminado");
        this.$emit('actualizarQuery');
      }).catch(()=>{
        this.$notify({
          type:'error',
          text:'No se ha podido eliminar el recordatorio'
        })
      });
    }
  },
  props:['nota']
}
</script>

<style>
@import "https://use.fontawesome.com/releases/v5.6.1/css/all.css";
.completado{
  text-decoration: line-through;
  color: #00bc8c;
}
.nota{
  width: 100%;
  background-color: #303030;
  color: wheat;
  margin: auto;
  display: flex;
  flex-flow: row wrap;
  align-items: center;
  border-bottom: 3px solid wheat;
}
.cuerpo{
  display: flex;
  align-items: center;
  flex-flow: column;
  width: 80%;
}
.nota:last-of-type{
  border: none;
}
.tituloNota{
  display: flex;
  font-size: calc(0.8rem + 1vw);
  position: relative;
  margin-left: 5%;
  width: 100%;
  transition: all 1.1s cubic-bezier(0.19, 1, 0.22, 1);

}
.delete{
  width: 18%;
  height: 100%;
  text-align: right;
}
.delete i{
  font-size: 4vh;
  margin-right: 20%;
  transition: all 1.1s cubic-bezier(0.19, 1, 0.22, 1);
  cursor: pointer;
}
.delete i:hover{
  color: lightcoral;
}
.prioridades{
  width: 300px;
  text-align: left;
  padding-left: 1vw;
  padding-bottom: 1vw;
}
.btn-pri{
  margin-left: 1% ;
  background-color: rgba(128, 128, 128, 0.253);
  border: 1px solid rgba(245, 222, 179, 0.623);
  border-radius: 15% 15%;
  color: rgba(245, 222, 179, 0.623);
  cursor: pointer;
}
.low{
  background-color: #00bc8c;
  border: 1px solid #00bc8d79;
  border-radius: 15% 15%;
  font-weight: bolder;
  color: black
}
.normal{
  background-color: wheat;
  border: 1px solid rgba(236, 206, 75, 0.739);
  border-radius: 15% 15%;
  font-weight: bolder;
  color: black
}
.high{
  background-color: lightcoral;
  border: 1px solid red;
  border-radius: 15% 15%;
  font-weight: bolder;
  color: black
}


.check {
  width: 3.4vh;
  height: 3.4vh;
  position: relative;
  margin-left: 4%;
  margin-right: 4%;
}
.check input {
  display: none;
}
.check input:checked + .box {
  background-color: #b3ffb7;
}
.check input:checked + .box:after {
  top: 0;
}
.check .box {
  width: 3.4vh;
  height: 100%;
  transition: all 1.1s cubic-bezier(0.19, 1, 0.22, 1);
  border: 2px solid transparent;
  border-radius: 100%;
  background-color: white;
  position: relative;
  overflow: hidden;
  cursor: pointer;
  box-shadow: 0 5px rgba(0, 0, 0, 0.2);
}
.check .box:after {
  width: 50%;
  height: 20%;
  content: '';
  position: absolute;
  border-left: 7.5px solid;
  border-bottom: 7.5px solid;
  border-color: #038665;
  transform: rotate(-45deg) translate3d(0, 0, 0);
  transform-origin: center center;
  transition: all 1.1s cubic-bezier(0.19, 1, 0.22, 1);
  left: 0;
  right: 0;
  top: 200%;
  bottom: 5%;
  margin: auto;
}

</style>
