<script>
import Vue from 'vue';
import VueColorPickerSheldon from '@/vue-color-picker-sheldon.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueColorPickerSheldon
  },
  data(){
    return {
      colorValue:''
    }
  },
  methods:{
    receiveColor(colorValue){
      this.colorValue = colorValue
    }
  }
});
</script>

<template>
  <div id="app">
    {{colorValue}}
    <vue-color-picker-sheldon :draggable="true" @receiveColor="receiveColor"/>
  </div>
</template>
