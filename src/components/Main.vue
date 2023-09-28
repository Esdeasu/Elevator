<template>
  <div class="container">
      <div class="shafts">
          <div v-for="shaft, index1 of shafts" class="shaft">
              <div v-for="index of floors" class="floor"
                  :style="`height: ${floorBox}px; width: ${floorBox}px`"
              >
              </div>
              <div class="highlighter" :style="`top: ${shaft.top}px; height: ${floorBox}px; width: ${floorBox}px`">
                  {{ Math.round((floors * floorBox  - shaft.top) / floorBox) }}
              </div>
              <button class="reset"
                  :style="`width: ${floorBox}px`">
                  Reset
              </button>
          </div>
      </div>
      <div class="floors">
          <div v-for="index of floors" class="floor" 
              :style="`height: ${floorBox}px; width: ${floorBox}px`"
          >
            <button class="button">
            {{ floors - index + 1}}
            </button> 
          </div>
      </div>
  </div>
</template>

<script>
export default{
  data(){
    return{
      shafts: 7,
      floors: 15,
      floorBox: 20
    }
  }
}
</script>

<style>
* {
    color: black
}
.container {
    height: fit-content;
    display: flex;
    flex-direction: row;
    gap: 1rem;
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
