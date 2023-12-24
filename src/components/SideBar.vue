<template >
    <div>
        <aside class="  d-flex flex-column row-gap-3"> 
        <a class="btn btn-outline-primary" :class="{'disabled':cropTriggered}" href="#" @click="toggleCrop" role="button">C
        </a>
        <a class="btn btn-outline-secondary" :class="{'disabled':!cropTriggered}" href="#" @click="rotaitImageRight" role="button">R</a>
        <a class="btn btn-outline-secondary" :class="{'disabled':!cropTriggered}" href="#" @click="rotaitImageLeft" role="button">L</a>
        </aside>
    </div>
</template>
<script>
/* eslint-disable */
export default {
    props:['cropTriggered'],
    data() {
        return {
        }
    },
    methods: {
        toggleCrop(){
            this.$emit('crop-triggered', this.cropTriggered);        
        },
        rotaitImageLeft(){
                this.$emit('rotaite-image-left');
        },
        rotaitImageRight(){
            this.$emit('rotaite-image-right');
        }
    },
}
</script>
<style scoped lang="css">
    
</style>