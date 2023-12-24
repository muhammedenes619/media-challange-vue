<template>
  <div class="container justify-items-center align-items-center ">
    <div class="row  align-items-center vh-100">
      <div class=" col-12">
        <TopBar :isImageUploaded="isImageUploaded" :inEditMode="inEditMode" @toggle-edit-mode="toggelInEditMode" />
        <MainArea @image-uploaded="receiveImageUploadStatus" :inEditMode="inEditMode"  v-show="!inEditMode"/>
      </div>
    </div>
  </div>
</template>

<script>
    /* eslint-disable */
    import MainArea from './components/MainArea.vue'
    import TopBar from './components/TopBar.vue'
    export default {
  name: 'App',
  components: {
    MainArea,
    TopBar,
  },
  data() {
    return {
      inEditMode:false,
      isImageUploaded: false,

    }
  },
  methods: {
    receiveImageUploadStatus(isUploaded) {
      this.isImageUploaded = isUploaded;

  },
  toggelInEditMode(){
    this.inEditMode = !this.inEditMode;
    console.log(this.inEditMode,this.isImageUploaded);

  }
   
  },
}
</script>

<style>
/* #app {
 
} */


</style>
