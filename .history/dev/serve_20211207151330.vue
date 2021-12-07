<script>
import Vue from 'vue';
import VueColorPickerSheldon from '@/vue-color-picker-sheldon.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueColorPickerSheldon
  }
});
</script>

<template>
  <div id="app">
    <vue-color-picker-sheldon />
  </div>
</template>
