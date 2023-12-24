<template>
    <div>
        <div class="card ">
      <div class="card-body d-flex column-gap-3  justify-content-center align-items-center">
        
        <a  :class="{ 'disabled': !isImageUploaded || inEditMode , }"  @click="toggleInEditMode" class="btn btn-outline-secondary" href="#" role="button">Edit</a>
        <a class="btn btn-outline-success" href="#" role="button">View</a>
      </div>
    </div>
    </div>
</template>
<script>
        /* eslint-disable */

export default {
  props:["isImageUploaded","inEditMode"],
  data() {
    return {
      
    }
  },
  methods: {
    toggleInEditMode(){
      this.$emit("toggle-edit-mode");
    }
  },
    
}
</script>
<style scoped lang="css">
    .card{
        border-bottom: none;
        border-bottom-left-radius: 0%;
        border-bottom-right-radius: 0%;
    }
</style>