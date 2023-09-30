<template>
  <div class="main">
    <Main/>
  </div>
</template>

<script>
import Main from './views/Main.vue';

export default{
  components:{
    Main,
  }
}
  
</script>

<style scoped>
.main{
  height: inherit;
  width: inherit;
  position: absolute;
  display: flex;
}
</style>
