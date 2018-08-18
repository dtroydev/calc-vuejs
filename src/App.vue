<template>
  <div id="app">
    <Calculator/>
  </div>
</template>

<script>
import Calculator from './components/Calculator.vue';

export default {
  name: 'app',
  components: {
    Calculator,
  },
};
</script>

<style>
/* center the calculator */
html,
body {
  height: 100%;
}
body {
  display: grid;
  justify-items: center;
  align-items: center;
}
</style>
