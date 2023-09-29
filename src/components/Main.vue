<template>
  <div class="container">
      <div class="shafts">
          <div v-for="shaft, index1 of shafts" class="shaft">
              <div v-for="index of settings.floors" class="floor"
                  :style="`height: ${settings.floorBox}px; width: ${settings.floorBox}px`"
              >
              </div>
              <div class="highlighter"
                :class="{ 'waiting' : shafts[index1].waitingFlag}"
                :style="`top: ${shaft.top}px; height: ${settings.floorBox}px; width: ${settings.floorBox}px`">
                  {{shafts[index1].floor}}
                  <div v-if="shafts[index1].loader">
                    <div v-if = "shafts[index1].arrow === 'up'">
                      &#8679
                    </div>
                    <div v-if = "shafts[index1].arrow === 'down'">
                      &#8681
                    </div>
                  </div>
              </div>
              <button class="reset" @click="reset(index1)" :style="`width: ${settings.floorBox}px`">
                  Reset
              </button>
          </div>
      </div>
      <div class="buttons">
          <div v-for="index of settings.floors" 
            class="floor" 
            :style="`height: ${settings.floorBox}px; width: ${settings.floorBox}px`">
            <button 
              class="button"
              :class="{'waitingButt' : !floorsButtons[settings.floors - index]}" 
              @click="callFloor(settings.floors - index + 1)">
                {{  settings.floors - index + 1 }}
            </button> 
          </div>
      </div>
  </div>
</template>

<script>
export default{
  data(){
    return{
      settings:{
        shafts: 5,
        floors: 15,
        floorBox: 20,
      },
      shafts:[],
      floorsButtons:[],
    };
  },
  mounted() {
    if(!localStorage.getItem("shafts")){
        for (let i = 0; i < this.settings.shafts; i++) {
        const newFloor = Math.round(Math.random() * (this.settings.floors - 1)) + 1;
        this.shafts.push({
          floor: newFloor,
          top: this.settings.floorBox * (this.settings.floors - newFloor),
          loader: false,
          waitingFlag:false,
          arrow: '',
        });
      }
      console.log(Object.keys(this.shafts))
      for (let i = 0; i < this.settings.floors; i++) {
        this.floorsButtons[i] = true;
      }
    }else{
      let s = JSON.parse(localStorage.getItem("shafts"))
      for(let i = 0; i < Object.keys(s).length; i++){
        this.shafts.push(s[i.toString()])
        console.log(s[i.toString()])
      }
      for(let i = 0; i < this.shafts.length; i++){
        if(this.shafts[i].waitingFlag){
          setTimeout(()=>{
            this.floorsButtons[this.shafts[i].floor - 1] = true
            this.shafts[i].loader = false;
            this.shafts[i].waitingFlag = false;
          }, 3000)
        }else if(this.shafts[i].arrow != ''){
          let newTop = this.settings.floorBox * (this.settings.floors - this.shafts[i].floor)
          let diff = Math.abs(this.shafts[i].floor - (this.settings.floors * this.settings.floorBox  - this.shafts[i].top) / this.settings.floorBox)
          this.shiftFloor(i, newTop, diff, this.shafts[i].floor)
        }else if(this.shafts[i].loader){
          this.shafts[i].loader = false;
        }
      }
      this.floorsButtons = JSON.parse(localStorage.getItem("buttons")).data;
      console.log(this.floorsButtons)
    }
    
  },
  watch:{
    shafts: {
      handler(){
        let a = {...this.shafts}
        localStorage.setItem("shafts", JSON.stringify(a));
      },
      deep: true
    },
    floorsButtons: {
      handler(){
        localStorage.setItem("buttons", JSON.stringify({data:this.floorsButtons}));
      },
      deep: true
    },
  },
  methods:{
    async callFloor(floor){
      if (this.shafts.values === 0) return

      let closestIndex = -1
      let diff = this.settings.floors + 1
      let elevator = 0

      for (let i = 0; i < this.shafts.length; i++) {
          if (this.shafts[i].floor === floor) return
          if (Math.abs(this.shafts[i].floor - floor) < diff && !this.shafts[i].loader) {
              elevator = this.shafts[i].floor
              closestIndex = i
              diff = Math.abs(this.shafts[i].floor - floor)
          }
      }
      if (closestIndex === -1) return
      this.shafts[closestIndex].loader = true
      this.floorsButtons[floor - 1] = false
      this.shafts[closestIndex].floor = floor

      let newTop = this.settings.floorBox * (this.settings.floors - floor)

      floor > elevator ? this.shafts[closestIndex].arrow = "up" :  this.shafts[closestIndex].arrow = "down"

      await this.shiftFloor(closestIndex, newTop, diff, floor)
    },

    async shiftFloor(index, to, amount, floor){
      let step = (this.shafts[index].top - to) / Math.round(20 * amount)
      

      if (step === 0)  {
          this.shafts[index].loader = false
          return
      }
      for (let i = 0; i < Math.round(20 * amount); i++) {
          await new Promise(resolve => setTimeout(resolve, 50));
          this.shafts[index].top -= step
      }
      this.shafts[index].waitingFlag = true
      this.shafts[index].arrow = ""

      await new Promise(resolve => setTimeout(resolve, 3000));

      this.floorsButtons[floor - 1] = true
      this.shafts[index].loader = false;
      this.shafts[index].waitingFlag = false;
      
    },
    async reset(index){
      if (this.shafts[index].loader) return;
      const tempFloor = this.shafts[index].floor - 1
      this.shafts[index].loader = true
      this.shafts[index].floor = 1
      this.shafts[index].arrow = "down"
      await this.shiftFloor(index, (this.settings.floors - 1) * this.settings.floorBox, tempFloor , 1); 
    },
  }
}
</script>

<style>
* {
  color: black
}
.container {
  height: fit-content;
  margin-left: 50px;
  display: flex;
  flex-direction: row;
  align-self: center;
}

.shafts {
  display: flex;
  flex-direction: row;
}
.shaft {
  display: flex;
  flex-direction: column;
  position: relative;
}
.reset {
  position: absolute;
  bottom: -50px;
  height: 40px;
  min-width: 50px;
}
.highlighter {
  display: flex;
  flex-direction: row;
  justify-content: center;
  text-align: center;
  position: absolute;
  min-width: 50px;
  background-color: aliceblue;
  z-index: -1;
}
.waiting{
  animation: waiting 1s linear infinite;
}
.waitingButt{
  background-color: rgb(223, 118, 118);
}
.floors {
  display: flex;
  flex-direction: column;
}
.floor {
  min-width: 50px;
}
.button{
  width: 100%;
}
@keyframes waiting {
  from{
    
  }
  to{
    background-color: rgb(194, 147, 147);
  }
}
</style>
