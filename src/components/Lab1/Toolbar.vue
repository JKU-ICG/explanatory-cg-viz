<template>
  <div>
    <div id = "toolbar" style="height: 50px; width: 500px"></div>
    <slider-picker :value="color" @input="setColor" ></slider-picker>
    <button @click="displayObject('cube')">Cube</button>
    <button @click="displayObject('square')">Square</button>
    <button @click="displayObject('sphere')">Sphere</button>
    <!-- Directly updates the color value -->
  </div>  
</template>

<script lang="ts">
import Vue from 'vue';
import slider from 'vue-color/src/components/Slider.vue';
import { createNamespacedHelpers } from 'vuex';
import { Component, Prop } from 'vue-property-decorator';
import { mixins } from 'vue-class-component';

const { mapGetters, mapActions } = createNamespacedHelpers('cubestore');

@Component({
  name: 'toolbar',
  components: {
    'slider-picker': slider,
  },
  computed: {
    ...mapGetters(['color']),
  },
  methods: {
    ...mapActions(['setColor', 'displayObject']),
  },
})
export default class Toolbar extends Vue { }
</script>
