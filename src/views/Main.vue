<template>
  <div class="inputs">
    Кол. этажей
    <input type="number" v-model="newFloor">
    Кол. лифтов
    <input type="number" v-model="newShafts">
    <div v-if="message">Лифт движется</div>
    <button @click = "newSettings" style="margin-top: 10px;">Ок</button>
  </div>
  
  <div class="container">  
    <Shafts
      @reset-shaft="reset"
      :get-settings="settings"
      :get-shafts="shafts"
    />
    <Floors
      @call-elevator="callElevator"
      :get-settings="settings"
      :get-floor-buttons="floorsButtons"
    />
  </div>
</template>

<script>
import Shafts from '../components/Shafts.vue';
import Floors from '../components/Floors.vue';
export default{
  components:{
    Shafts,
    Floors,
  },
  data(){
    return{
      newFloor: null,
      newShafts: null,

      settings: {
        shafts: 5,
        floors: 17,
        floorBox: 20,
      },

      shafts: [],        
      floorsButtons: [],
      message: false,
    };
  },
  created(){
    if(localStorage.getItem("settings")){
      this.settings = JSON.parse(localStorage.getItem("settings")).data;
    }
  },
  mounted() {
    if(!localStorage.getItem("shafts")){
      this.createShafts();
    }else{
      const shaftsFromStore = JSON.parse(localStorage.getItem("shafts")).data;

      for(let i = 0; i < Object.keys(shaftsFromStore).length; i++){
        this.shafts.push(shaftsFromStore[i.toString()]);
      };

      for(let i = 0; i < this.shafts.length; i++){
        if(this.shafts[i].waitingFlag){
          setTimeout(()=>{
            this.floorsButtons[this.shafts[i].floor - 1] = true;
            this.shafts[i].loader = false;
            this.shafts[i].waitingFlag = false;
          }, 3000);
      
        }else if(this.shafts[i].arrow != ''){
          let newTop = this.settings.floorBox * (this.settings.floors - this.shafts[i].floor);
          let diff = Math.abs(this.shafts[i].floor - (this.settings.floors * this.settings.floorBox  - this.shafts[i].top) / this.settings.floorBox);
          this.elevatorMovement(i, newTop, diff, this.shafts[i].floor);

        }else if(this.shafts[i].loader){
          this.shafts[i].loader = false;
        };
      };
      this.floorsButtons = JSON.parse(localStorage.getItem("buttons")).data;
    }
  },
  watch:{
    shafts: {
      handler(){
        localStorage.setItem("shafts", JSON.stringify({data : this.shafts}));
      },
      deep: true
    },
    floorsButtons: {
      handler(){
        localStorage.setItem("buttons", JSON.stringify({data : this.floorsButtons}));
      },
      deep: true
    },
    settings: {
      handler(){
        localStorage.setItem("settings", JSON.stringify({data : this.settings}));
      },
      deep: true
    }
  },
  methods:{
    createShafts(){
      for (let i = 0; i < this.settings.shafts; i++) {
        const newFloor = Math.round(Math.random() * (this.settings.floors - 1)) + 1;
        this.shafts.push({
          floor: newFloor,                                                  // Текущий этаж на котором находится лифт
          top: this.settings.floorBox * (this.settings.floors - newFloor),  // Этаж на котором находится лифт в пикселях
          loader: false,                                                    // Занят ли в данный момент лифт
          waitingFlag:false,                                                // Флаг ожидания лифта по прибытию на необходимый этаж
          arrow: '',                                                        // Направление движения лифта
        });
      };
      for (let i = 0; i < this.settings.floors; i++) {
        this.floorsButtons[i] = true;
    };
    },

    async callElevator(targetFloor){
      if (this.shafts.values === 0) return;

      let closestShaft = -1;
      let diff = this.settings.floors + 1;
      let elevatorPosition = 0;

      for (let i = 0; i < this.shafts.length; i++) {
        if (this.shafts[i].floor === targetFloor) return;
        if (Math.abs(this.shafts[i].floor - targetFloor) < diff && !this.shafts[i].loader) {
            elevatorPosition = this.shafts[i].floor;
            closestShaft = i;
            diff = Math.abs(this.shafts[i].floor - targetFloor);
        };
      }
      if (closestShaft === -1) return;

      this.shafts[closestShaft].loader = true;
      this.floorsButtons[targetFloor - 1] = false;
      this.shafts[closestShaft].floor = targetFloor;

      let newTop = this.settings.floorBox * (this.settings.floors - targetFloor);

      this.shafts[closestShaft].arrow = `${targetFloor > elevatorPosition ? "up": "down"}`
      await this.elevatorMovement(closestShaft, newTop, diff, targetFloor);
    },

    async elevatorMovement(choosenShaft, newTop, diff, targetFloor){
      const step = (this.shafts[choosenShaft].top - newTop) / Math.round(20 * diff);
      
      for (let i = 0; i < Math.round(20 * diff); i++) {
        await new Promise(resolve => setTimeout(resolve, 50));
        this.shafts[choosenShaft].top -= step;
      };

      this.shafts[choosenShaft].waitingFlag = true;
      this.shafts[choosenShaft].arrow = "";

      await new Promise(resolve => setTimeout(resolve, 3000));

      this.floorsButtons[targetFloor - 1] = true;
      this.shafts[choosenShaft].loader = false;
      this.shafts[choosenShaft].waitingFlag = false;
      
    },

    async reset(shaftIndex){
      if (this.shafts[shaftIndex].loader) return;

      const tempFloor = this.shafts[shaftIndex].floor - 1;
      const newTop = (this.settings.floors - 1) * this.settings.floorBox

      this.shafts[shaftIndex].loader = true;
      this.shafts[shaftIndex].floor = 1;
      this.shafts[shaftIndex].arrow = "down";
      
      await this.elevatorMovement(shaftIndex, newTop, tempFloor , 1); 
    },

    newSettings(){
      if(Number.isInteger(this.newFloor) && Number.isInteger(this.newShafts)){
        for(let i = 0; i < this.shafts.length; i++){
          if(this.shafts[i].loader){
            this.message = true
            setTimeout(()=>{this.message = false}, 3000);
            return
          }
        }
        this.settings.floors = this.newFloor;
        this.settings.shafts = this.newShafts;
        this.shafts = []

        this.createShafts()
      }
    },
  }
}
</script>

<style>
.container {
  height: fit-content;
  margin-left: 50px;
  display: flex;
  flex-direction: row;
  align-self: center;
}
.inputs{
  display: flex;
  flex-direction: column;
  margin: 20px;
}
</style>
