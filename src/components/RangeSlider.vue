<template>
    <v-range-slider
      v-model="range"
      :max="videoDuration"
      :min="0"
      :step="0.1"
      hide-details
      class="align-center"
      strict
      thumb-label="always"
      @mouseup="setStartTime"
      @touchend="setStartTime"
    >
      <template v-slot:thumb-label="props">
        {{ convertToHHMMSS(props.modelValue) }}
      </template>
    </v-range-slider>
  </template>
  
  <script>
  export default {
    props: ["videoDuration", "convertToHHMMSS","trimStartPoint"],
    data() {
      return {
        range: [0, this.videoDuration],
        previousStart: 0, // Initialize previousStart to track the initial start value
      };
    },
    methods: {
      setStartTime() {
        if (this.range[0] !== this.previousStart) {
          const startTime = this.range[0];
          this.$emit("start-time", startTime);
          this.previousStart = startTime; // Update previousStart when the value changes
        }
        else{
          const endTrim=this.range[1]
          this.$emit("end-time", endTrim);
        }
      },
    },
  };
  </script>
  
  <style scoped>
  /* Add your scoped styles here if needed */
  </style>
  