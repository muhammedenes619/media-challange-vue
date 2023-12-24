<template >
  <div></div>
    <div class="card mb-3" style="max-width: 100%;" >
      <div v-if=" !isImageUploaded && !inEditMode" class="card-body d-flex flex-column justify-content-center align-items-center">
        <h5 class="card-title">There is no Image uploded yet  </h5>
        <p class="card-text"> if you wish You can upload one using upload button</p>
        <a href="#" class="btn btn-primary " >Upload
            <form class="form" @submit.prevent>
            <div  class="image-upload-form-drop" @change="onFileChange">
                <input type="file" 
                     class="image-upload-form__file"
                     accept="image/*">
            </div>
          </form>
         </a>
      </div>
      <div v-else-if=" isImageUploaded && inEditMode">
        <div class="row  g-0">
            <div class="col d-flex justify-content-center ">
              <div class="cropper-container">
                <img :src="imageSrc" class=" uploaded-img img-fluid rounded-start" ref="image" alt="...">
              </div>
            </div>
            <div class="col-auto ">
              <div class="card-body side-body d-flex flex-column justify-content-center align-items-center">
                <SideBar :cropTriggered="cropTriggered" @rotaite-image-right="rotaiteImage(90)" @rotaite-image-left="rotaiteImage(-90)" @crop-triggered="toggleCrop"/>
                <a class="btn mt-5 btn-outline-secondary" :class="{'disabled':!cropTriggered}" @click="submitCroppedImage" href="#" role="button">S</a>
              </div>
            </div>
        </div>
      </div>
      <div v-else >
        <div class="row g-0">
          <div class="col text-center">
            <img :src="imageSrc" class=" uploaded-img img-fluid rounded-start" alt="...">
          </div>
        </div>
      </div>
    </div>
</template>
<script>
    import SideBar from './SideBar.vue'
    import 'cropperjs/dist/cropper.css'; // Import the Cropper CSS file
import Cropper from 'cropperjs'; // Import the Cropper JS file

export default {
        /* eslint-disable */
    components: {
    SideBar
  },
  props:["inEditMode"],
    data() {
        return {
            isImageUploaded:false,
            imageSrc: '',
            cropTriggered:false,
            cropper:{},
            destination:{},
            image:{},
        }
    },
    methods: {
     onFileChange(event) {
      const file = event.target.files[0];
      if (file && file.type.startsWith('image/')) {
        this.isImageUploaded = true;
        const reader = new FileReader();
        reader.onload = e => {
          this.imageSrc = e.target.result;
        };
        reader.readAsDataURL(file);
        this.toggleIsImageUploaded
        this.$emit('image-uploaded', this.isImageUploaded);
      }
    },
    toggleCrop(){
      this.cropTriggered=!this.cropTriggered  
      console.log("triggered",this.cropTriggered);    
      this.cropImage()
    },
    cropImage(){
      this.image=this.$refs.image
            this.cropper=new Cropper(this.image,{
              viewMode: 1,
              aspectRatio: null ,
              guides: true,
              movable: true,
           
        })
    },
    rotaiteImage(degrees) {
      if (this.cropper) {
        this.cropper.rotate(degrees);
      }
    },
    
    submitCroppedImage() {
      if (this.cropper) {
        const canvas = this.cropper.getCroppedCanvas();
        const croppedImageData = canvas.toDataURL('image/jpeg'); // Change format if needed

        // Update the image source with the cropped image data
        this.imageSrc = croppedImageData;

        // Reset the cropTriggered flag and destroy the cropper
        this.cropTriggered = false;
        this.cropper.destroy();
        this.cropper = null;
      }
    },
    
  }
}
</script>
<style scoped lang="css">
    .card{
        border-top-left-radius: 0;
        border-top-right-radius: 0;
        height: 100%;
        width:100%;
    }
   
    .image-upload-form__file {
    position: absolute;
    top: 0;
    left: 0;

    width: 100%;
    height: 100%;
    opacity: 0;
    -moz-opacity: 0;
  }


  .uploaded-img {
    padding: 10px;
    max-height: 50vh;
    object-fit: contain;
}
.btn{
  position: relative;
  display: block;
}
.side-body{
  height: 100%;
  border-left: 1px solid #dee2e6;
}
.cropper-container{
  max-width: 600px;
  height: 100%;
}
</style>