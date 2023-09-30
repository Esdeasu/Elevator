<template>
  <div class="container">
    <Shafts
      @reset-shaft="reset"
      :get-settings="settings"
      :get-shafts="shafts"
    />
    <Floors
      @call="callFloor"
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
      //Настройки количества шахт и этажей
      settings:{
        shafts: 5,
        floors: 17,
        floorBox: 20,
      },
      shafts:[],        //Массив шахт лифта 
      floorsButtons:[], //Массив с флагами для кнопок вызова лифта
    };
  },
  mounted() {
    if(!localStorage.getItem("shafts")){
      this.createShafts();
    }else{
      let s = JSON.parse(localStorage.getItem("shafts")).data;

      for(let i = 0; i < Object.keys(s).length; i++){
        this.shafts.push(s[i.toString()]);
      };
      for(let i = 0; i < this.shafts.length; i++){
        // Работа с флагами после перезагрузки страницы
        if(this.shafts[i].waitingFlag){
          setTimeout(()=>{
            this.floorsButtons[this.shafts[i].floor - 1] = true;
            this.shafts[i].loader = false;
            this.shafts[i].waitingFlag = false;
          }, 3000);
        // Реализация продолжнения движения после перезагрузки страницы
        }else if(this.shafts[i].arrow != ''){
          let newTop = this.settings.floorBox * (this.settings.floors - this.shafts[i].floor);
          let diff = Math.abs(this.shafts[i].floor - (this.settings.floors * this.settings.floorBox  - this.shafts[i].top) / this.settings.floorBox);

          this.shiftFloor(i, newTop, diff, this.shafts[i].floor);
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
  },
  methods:{
    methodThatForcesUpdate() {
      this.$forceUpdate();  // Notice we have to use a $ here
    },
    // Формирование массива с шахтами стартовой информацией
    createShafts(){
      for (let i = 0; i < this.settings.shafts; i++) {
        const newFloor = Math.round(Math.random() * (this.settings.floors - 1)) + 1;
        this.shafts.push({
          floor: newFloor,                                                  // Текущий этаж на котором находится лифт
          top: this.settings.floorBox * (this.settings.floors - newFloor),  // Этаж на котором находится лифт в пикселях
          loader: false,                                                    // Занят ли в данный момент лифт
          waitingFlag:false,                                                // Флаг для анимации ожидания лифта по прибытию на необходимый этаж
          arrow: '',                                                        // Направление движения лифта
        });
      };
      for (let i = 0; i < this.settings.floors; i++) {
        this.floorsButtons[i] = true;
    };
    },
    // Метод вызова лифта
    async callFloor(floor){
      if (this.shafts.values === 0) return;

      let closestIndex = -1;
      let diff = this.settings.floors + 1;
      let elevator = 0;

      //Поиск ближайшего лифта
      for (let i = 0; i < this.shafts.length; i++) {
        if (this.shafts[i].floor === floor) return;
        if (Math.abs(this.shafts[i].floor - floor) < diff && !this.shafts[i].loader) {
            elevator = this.shafts[i].floor;
            closestIndex = i;
            diff = Math.abs(this.shafts[i].floor - floor);
        };
      }
      if (closestIndex === -1) return;

      this.shafts[closestIndex].loader = true;
      this.floorsButtons[floor - 1] = false;
      this.shafts[closestIndex].floor = floor;

      let newTop = this.settings.floorBox * (this.settings.floors - floor);

      floor > elevator ? this.shafts[closestIndex].arrow = "up" :  this.shafts[closestIndex].arrow = "down";

      await this.shiftFloor(closestIndex, newTop, diff, floor);
    },

    // Механика движения лифта
    async shiftFloor(index, to, amount, floor){
      //Определение шага движения лифта
      let step = (this.shafts[index].top - to) / Math.round(20 * amount);
      
      if (step === 0)  {
        this.shafts[index].loader = false;
        return;
      };

      // Изменение позиции лифта на определённый шаг
      for (let i = 0; i < Math.round(20 * amount); i++) {
        await new Promise(resolve => setTimeout(resolve, 50));
        this.shafts[index].top -= step;
      };

      this.shafts[index].waitingFlag = true;
      this.shafts[index].arrow = "";

      await new Promise(resolve => setTimeout(resolve, 3000));

      this.floorsButtons[floor - 1] = true;
      this.shafts[index].loader = false;
      this.shafts[index].waitingFlag = false;
      
    },

    //Метод возврата лифта на 1 этаж
    async reset(index){
      if (this.shafts[index].loader) return;

      const tempFloor = this.shafts[index].floor - 1;

      this.shafts[index].loader = true;
      this.shafts[index].floor = 1;
      this.shafts[index].arrow = "down";
      
      await this.shiftFloor(index, (this.settings.floors - 1) * this.settings.floorBox, tempFloor , 1); 
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
</style>
