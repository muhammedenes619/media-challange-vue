<template >
    <div>
    <div v-if="videoList.length === 0" class="row justify-content-center align-items-center">
      <div class="col text-center">
        <div class="alert alert-warning" role="alert">
          No Videos in the list
        </div>
      </div>
    </div>

    <div v-else class="row">
        <div class="col-12 text-center">
            <h2>Merge List</h2>
        </div>
      <div v-for="(video, index) in videoList" :key="index" class="col-3 video-container">
        <video :src="video.src" controls></video>
      </div>
      <div class="col-12 text-center">
        <a class="btn btn-outline-secondary mb-2" @click="mergeVideosTrigger" role="button">Merge</a>
      </div>
    </div>
  </div>
</template>
<script>
export default {
    props:["videoList"],
    methods: {
        mergeVideosTrigger(){
            this.$emit("merge-videos")
        },
    },
}
</script>
<style scoped>

    .video-container video{
        max-width: 100%;
        padding: 10px;
    }
</style>