<template>   
    <div></div>
</template>

<script lang="ts">
import { AbstractView } from '@/components/Lab1/AbstractView.vue';
import { createNamespacedHelpers } from 'vuex';
import { Component, Prop, Watch } from 'vue-property-decorator';
import { mixins } from 'vue-class-component';

const { mapActions, mapGetters } = createNamespacedHelpers('cubestore');

@Component({
    name: 'output-control',
    computed: {
        ...mapGetters(['color', 'objects']),
    },
})
export class OutputControl extends mixins(AbstractView) {
    private mounted() {
        this.init();
    }

    @Watch('color')
    private onColorChanged(val: string, oldVal: string) {
        this.updateColors(val);
    }

    @Watch('objects')
    private onObjectAdded() {
        this.renderScene();
    }
}

export default OutputControl;
</script>
