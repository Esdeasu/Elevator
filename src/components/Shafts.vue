<template>
  <div class="shafts">
    <div v-for="shaft, index1 of getShafts" class="shaft">
      <div v-for="index of getSettings.floors" class="floor"
        :style="`height: ${getSettings.floorBox}px; width: ${getSettings.floorBox}px`"
      >
      </div>
      <div class="highlighter"
        :class="{ 'waiting' : getShafts[index1].waitingFlag}"
        :style="`top: ${shaft.top}px; height: ${getSettings.floorBox}px; width: ${getSettings.floorBox}px`">
          {{getShafts[index1].floor}}
          <div v-if="getShafts[index1].loader">
            <div v-if = "getShafts[index1].arrow === 'up'">
              &#8679
            </div>
            <div v-if = "getShafts[index1].arrow === 'down'">
              &#8681
            </div>
          </div>
      </div>
      <button class="reset" @click = "$emit('resetShaft', index1)" :style="`width: ${getSettings.floorBox}px`">
        Reset
      </button>
    </div>
  </div>
</template>

<script>
export default{
  props: {
    getShafts:{
      type: Object,
      required: true,
      default(){
        return{}
      }
    },
    getSettings:{
      type: Object,
      required: true,
      default(){
        return[]
      }
    }
  },
}
</script>

<style>
.shafts {
  display: flex;
  flex-direction: row;
}
.shaft {
  display: flex;
  flex-direction: column;
  position: relative;
}
.floor {
  min-width: 50px;
}
.highlighter {
  display: flex;
  flex-direction: row;
  justify-content: center;
  text-align: center;
  align-items: center;
  position: absolute;
  min-width: 50px;
  background-color: rgb(168, 206, 240);
  border-radius: 5px;
  z-index: -1;
}
.reset {
  position: absolute;
  bottom: -40px;
  height: 40px;
  min-width: 50px;
}
.waiting{
  animation: waiting 1s linear infinite;
}
@keyframes waiting {
  from{
    background-color: rgb(255, 255, 255);
  }
  to{
    background-color: rgb(194, 147, 147);
  }
}
</style>