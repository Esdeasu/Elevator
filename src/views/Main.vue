<template>
  <div class="inputs">
    Кол. этажей
    <input type="number" min="1" max="28" v-model="newFloor">
    Кол. лифтов
    <input type="number" min="1" max="20" v-model="newShafts">
    Высота этажа
    <input type="number" min="1" max="40" v-model="newBox">
    <div v-if="msgFlag">{{ msg }}</div>
    <button @click = "newSettings" style="margin-top: 10px;">Ок</button>
  </div>
  
  <div class="container">  
    <Shafts
      @reset-shaft="reset"
      :get-settings="settings"
      :get-shafts="shafts"
    />
    <Floors
      @call-elevator="findElevator"
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
      newBox: null,

      settings: {
        shafts: 5,    // Количество шахт
        floors: 15,   // Количество этажей
        floorBox: 40, // Высота этажа
      },

      shafts: [],
      /*shafts : [{
        floor:  Текущий этаж на котором находится лифт
        top:  Этаж на котором находится лифт в пикселях
        waitingFlag: Флаг ожидания лифта по прибытию на необходимый этаж
        arrow: Направление движения лифта
      },...]*/
  
      floorsButtons: [],  // Список флагов для каждого этажа
      msgFlag: false,     
      msg: "",  
      callStack: [],      // Очередь вызовов
      elevatorLoaders: [] // Состояние лифтов
    };
  },
  watch:{
    shafts: {
      handler(){
        localStorage.setItem("shafts", JSON.stringify({data : this.shafts}));
      },
      deep: true
    },
    callStack: {
      handler(){
        localStorage.setItem("stack", JSON.stringify({data : this.callStack}));
      },
      deep: true
    },
    settings: {
      handler(){
        console.log("a")
        localStorage.setItem("settings", JSON.stringify({data : this.settings}));
      },
      deep: true
    },
    elevatorLoaders: {
      handler () {
        if (this.elevatorLoaders.length !== this.settings.shafts) return
        if (this.callStack.length === 0 ) return
        if (this.elevatorLoaders.every(item => item)) return
        const shaft = this.elevatorLoaders.findIndex(item => !item)
         
        if (shaft === -1) return
        const newFloor = this.callStack[0]
        this.callStack = this.callStack.slice(1,)
        this.callElevator(newFloor, this.shafts[shaft].floor, shaft)
      },
      deep: true
    }
  },
  methods:{
    // Создание шахт лифта
    createShafts(){
      const { settings } = this
      this.shafts = []
      for (let i = 0; i < settings.shafts; i++) {
        this.shafts.push({
          floor: 0,                                          // Текущий этаж на котором находится лифт
          top: settings.floorBox * (settings.floors - 1),    // Этаж на котором находится лифт в пикселях
          waitingFlag:false,                                 // Флаг ожидания лифта по прибытию на необходимый этаж
          arrow: '',                                         // Направление движения лифта
        });
      };
      this.floorsButtons = []
      this.elevatorLoaders = []
      for (let i = 0; i < settings.floors; i++) {
        this.floorsButtons.push(true)
      };
      for (let i = 0; i < settings.shafts; i++) {
        this.elevatorLoaders.push(false)
      }
    },
    // Вызов лифта на нужный этаж
    async callElevator (targetFloor, oldFloor, shaft) {
      const { settings } = this

      this.shafts[shaft].arrow = targetFloor > oldFloor ? "up": "down";
      let diff = Math.abs(oldFloor - targetFloor)

      this.elevatorLoaders[shaft] = true;
      this.floorsButtons[targetFloor] = false;
      this.shafts[shaft].floor = targetFloor;

      let newTop = settings.floorBox * (settings.floors - targetFloor - 1);

      await this.elevatorMovement(shaft, newTop, diff, targetFloor);

    },
    // Поиск ближайшего лифта
    async findElevator (targetFloor) {
      const { settings, elevatorLoaders, shafts } = this
      
      if (shafts.length === 0) return;
      if (this.callStack.includes(targetFloor)) return

      let closestShaft = -1;
      let diff = settings.floors;

      for (let i = 0; i < shafts.length; i++) {
        if (shafts[i].floor === targetFloor) return;
        if (Math.abs(shafts[i].floor - targetFloor) < diff && !elevatorLoaders[i]) {
            closestShaft = i;
            diff = Math.abs(shafts[i].floor - targetFloor);
        };
      }
      if (closestShaft === -1) {
        this.callStack.push(targetFloor)
        this.floorsButtons[targetFloor] = false
        return
      } else {
        await this.callElevator(targetFloor, shafts[closestShaft].floor, closestShaft)
      }
    },
    // Механика движения лифта на определённый этаж с определённым шагом
    async elevatorMovement(choosenShaft, newTop, diff, targetFloor){
      const step = (this.shafts[choosenShaft].top - newTop) / Math.round(20 * diff);
      
      for (let i = 0; i < Math.round(20 * diff); i++) {
        await new Promise(resolve => setTimeout(resolve, 40));
        this.shafts[choosenShaft].top -= step;
      };

      this.shafts[choosenShaft].waitingFlag = true;
      this.shafts[choosenShaft].arrow = "";

      await new Promise(resolve => setTimeout(resolve, 3000));

      this.floorsButtons[targetFloor] = true;
      this.elevatorLoaders[choosenShaft] = false;
      this.shafts[choosenShaft].waitingFlag = false;
      
    },
    // Функция отправки выбранного лифта на первый этаж
    async reset(shaftIndex){
      if (this.elevatorLoaders[shaftIndex] 
        || this.shafts[shaftIndex].waitingFlag 
        || this.shafts[shaftIndex].floor === 0) return;

      const tempFloor = this.shafts[shaftIndex].floor;
      const newTop = (this.settings.floors - 1) * this.settings.floorBox

      this.elevatorLoaders[shaftIndex] = true;
      this.shafts[shaftIndex].floor = 0;
      this.shafts[shaftIndex].arrow = "down";
      
      await this.elevatorMovement(shaftIndex, newTop, tempFloor , 0); 
    },

    newSettings(){
      // Проверка на целые числа
      if(!Number.isInteger(this.newFloor) || !Number.isInteger(this.newShafts) || !Number.isInteger(this.newBox)){
        this.msgFlag = true
        this.msg = "Введите целые числа"
        setTimeout(()=>{this.msgFlag = false}, 3000);
        return
      }
      // Проверка на удовлетворимый диапозон значений
      if(this.newFloor > 20 || this.newFloor < 2 ||
        this.newShafts > 20 || this.newShafts < 1 || 
        this.newBox < 20 || this.newBox > 40)
        {
        this.msgFlag = true
        this.msg = "Не те числа"
        setTimeout(()=>{this.msgFlag = false}, 3000);
        return
      }
      // Проверка на движущиеся лифты
      for(let i = 0; i < this.shafts.length; i++){
        if (this.elevatorLoaders[i]){
          this.msgFlag = true
          this.msg = "Лифт движется"
          setTimeout(()=>{this.msgFlag = false}, 3000);
          return
        }
      }
      // Изолирование входных параметров
      if (this.newShafts === this.settings.shafts && this.newFloor === this.settings.floors){
        if (this.newBox !== this.settings.floorBox){
          this.settings.floorBox = this.newBox
          for (let i = 0; i < this.shafts.length; i++){
            this.shafts[i].top = this.settings.floorBox * (this.settings.floors - this.shafts[i].floor - 1);
          }
        }
        return
      }
      this.settings.floors = this.newFloor;
      this.settings.shafts = this.newShafts;
      this.settings.floorBox = this.newBox;

      this.createShafts()
    },
  },
  
  created(){
    // Подтягивание настроек из стора
    if(localStorage.getItem("settings")){
      this.settings = JSON.parse(localStorage.getItem("settings")).data;
    }
    this.newFloor = this.settings.floors;
    this.newShafts = this.settings.shafts;
    this.newBox = this.settings.floorBox;
  },
  mounted() {
    // Если в сторе нет данных
    if(!localStorage.getItem("shafts")){
      this.createShafts();
    }
    // Если в сторе нет данных 
    else{
      this.callStack = JSON.parse(localStorage.getItem("stack"))?.data ?? [];

      const shaftsFromStore = JSON.parse(localStorage.getItem("shafts")).data;

      for (let i = 0; i < this.settings.floors; i++) {
        this.floorsButtons.push(true)
      };

      for(let i = 0; i < Object.keys(shaftsFromStore).length; i++){
        this.shafts.push(shaftsFromStore[i.toString()]);
      };
      // Флаги для продолжения анимации ожидания после обновления
      for(let i = 0; i < this.shafts.length; i++){
        if(this.shafts[i].waitingFlag){
          setTimeout(()=>{
            this.floorsButtons[this.shafts[i].floor] = true;
            this.elevatorLoaders[i] = false;
            this.shafts[i].waitingFlag = false;
          }, 3000);
      
        }
        // Продолжение движения лифта после обновления
        else if(this.shafts[i].arrow !== ''){
          const currFloor = ((this.settings.floors * this.settings.floorBox) - this.shafts[i].top) / this.settings.floorBox
          this.callElevator(this.shafts[i].floor, currFloor - 1, i)

        }else if(this.elevatorLoaders[i]){
          this.elevatorLoaders[i] = false;
        };
      };
      // Отключение занятых кнопок
      for (const shaft of this.shafts) {
        if (shaft.floor !== 0) {
          if (shaft.waitingFlag) {
            this.floorsButtons[shaft.floor] = false
          }
        }
      }
      // Отключение занятых кнопок в колстаке
      if (this.callStack.length !== 0) {
        for (const item of this.callStack) {
          this.floorsButtons[item] = false
        }
      }
    }
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
