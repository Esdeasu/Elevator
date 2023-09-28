<template>
  <div class="container">
      <div class="shafts">
          <div v-for="shaft, index1 of shafts" class="shaft">
              <div v-for="index of settings.floors" class="floor"
                  :style="`height: ${settings.floorBox}px; width: ${settings.floorBox}px`"
              >
              </div>
              <div class="highlighter" :style="`top: ${shaft.top}px; height: ${settings.floorBox}px; width: ${settings.floorBox}px`">
                  {{ Math.round((settings.floors * settings.floorBox  - shaft.top) / settings.floorBox) }}
              </div>
              <button class="reset" @click="reset(index1)" :style="`width: ${settings.floorBox}px`">
                  Reset
              </button>
          </div>
      </div>
      <div class="buttons">
          <div v-for="index of settings.floors" class="floor" :style="`height: ${settings.floorBox}px; width: ${settings.floorBox}px`">
            <button class="button">
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
    };
  },
  mounted() {
    for (let i = 0; i < this.settings.shafts; i++) {
      const newFloor =
        Math.round(Math.random() * (this.settings.floors - 1)) + 1;
      this.shafts.push({
        floor: newFloor,
        top: this.settings.floorBox * (this.settings.floors - newFloor),
        loader: false,
      });
    }
  },
  methods:{
    async shiftFloor(index, to, amount){
      let step = (this.shafts[index].top - to) / (20 * amount)
      if (step === 0)  {
          this.shafts[index].loader = false
          return
      }
      for (let i = 0; i < 20 * amount; i++) {
          await new Promise(resolve => setTimeout(resolve, 10));
          this.shafts[index].top -= step
      }
      this.shafts[index].loader = false
    },
    async reset(index){
      if (this.shafts[index].loader) return;
      this.shafts[index].loader = true
      await this.shiftFloor(index, (this.settings.floors - 1) * this.settings.floorBox, (this.shafts[index].floor - 1)); 
      this.shafts[index].floor = 1
      this.shafts[index].loader = false
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
    position: absolute;
    min-width: 50px;
    background-color: aliceblue;
    z-index: -1;
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
</style>
