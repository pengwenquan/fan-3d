<template>
  <Fanji></Fanji>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Fanji from "./components/fan-ji.vue";

export default defineComponent({
  components: {
    Fanji,
  },
});
</script>

<style lang="scss">
#app {
  width: 100vw;
  height: 100vh;
  background: url("./assets/bg.png") center center no-repeat;
}
</style>
