<template>
  <div class="floors">
    <div v-for="index of getSettings.floors" 
      class="floor" 
      :style="`height: ${getSettings.floorBox}px; width: ${getSettings.floorBox}px`">
      <button 
        style="width: 100%;"
        :style="`height: ${getSettings.floorBox}px;`"
        :class="{'waitingButt' : !getFloorButtons[getSettings.floors - index]}"
        :disabled="!getFloorButtons[getSettings.floors - index]" 
        @click="$emit('callElevator', (getSettings.floors - index))">
          {{  getSettings.floors - index + 1 }}
      </button> 
    </div>
  </div>
</template>
<script>
export default{
  props: {
    getSettings:{
      type: Object,
      required: true,
      default(){
        return[]
      }
    },
    getFloorButtons:{
      type: Array,
      required: true,
      default(){
        return[]
      }
    }
  },
}
</script>
<style>
.floors {
  display: flex;
  flex-direction: column;
}
.floor {
  min-width: 50px;
}
.waitingButt{
  background-color: rgb(223, 118, 118);
}
button {
  cursor: pointer;
}
.waitingButt:disabled{
  color: black;
  cursor: default;
}
</style>