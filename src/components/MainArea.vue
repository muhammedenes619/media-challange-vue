<template >
  <div></div>
    <div class="card mb-3" style="max-width: 100%;" >
      <videoList @merge-videos="mergeVideos" :videoList="videoList"/>
      <div v-if="isVideoUploaded">
        <div class="row">
          <div class="col-12 d-flex justify-content-center align-items-center">
             <div class="video-container p-2">
               <video ref="videoElement" :src="videoSrc" @loadedmetadata="getVideoDuration" controls></video>
               <RangeSlider class="py-2" @end-time="endTimeModify" @start-time="startTimeModify" :trimStartPoint="trimStartPoint" :videoDuration="videoDuration" :convertToHHMMSS="convertToHHMMSS"/>
               <a class="btn btn-outline-secondary mb-2" @click="trimVideo" role="button">Trim & Save</a>
               <a class="btn btn-outline-secondary mb-2" @click="addToMergeList"  role="button">Add To Merge List</a>
             </div>
          </div>
        </div>
      </div>
      <div v-else-if=" !isImageUploaded && !inEditMode && !isVideoUploaded" class="card-body d-flex flex-column justify-content-center align-items-center">
        <h5 class="card-title">There is no Image Or Video uploded yet  </h5>
        <p class="card-text"> if you wish You can upload one using upload button</p>
        <a href="#" class="btn btn-primary " >Upload
            <form class="form" @submit.prevent>
            <div  class="image-upload-form-drop" @change="onFileChange">
                <input ref="uploadedFile" type="file" 
                     class="image-upload-form__file"
                     accept="image/*,video/*">
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
      <div v-else-if=" isImageUploaded && !inEditMode" >
        <div class="row g-0">
          <div class="col text-center">
            <img :src="imageSrc" class=" uploaded-img img-fluid rounded-start" alt="...">
          </div>
        </div>
      </div>
    </div>
</template>
<script>
        /* eslint-disable */
    import SideBar from './SideBar.vue';
    import RangeSlider from './RangeSlider.vue'
    import VideoList from './VideoList.vue'
    import 'cropperjs/dist/cropper.css'; // Import the Cropper CSS file
    import Cropper from 'cropperjs'; // Import the Cropper JS file
    import { FFmpeg } from '@ffmpeg/ffmpeg';
    import { fetchFile, toBlobURL } from '@ffmpeg/util';
    import { createFFmpeg} from "@ffmpeg/ffmpeg";


export default {
    components: {
    SideBar,
    RangeSlider,
    VideoList
  },
  props:["inEditMode"],
    data() {
        return {
            isImageUploaded:false,
            cropTriggered:false,
            isVideoUploaded:false,
            imageSrc: '',
            videoSrc:'',
            cropper:{},
            destination:{},
            image:{},
            videoDuration:0,
            currentTime:0,
            trimStartPoint:0,
            trimEndPoint:0,
            videoFile : {},
            videoList:[]
        }
    },
    methods: {
     onFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        if (file.type.startsWith('image/')) {
          this.isImageUploaded = true;
          const reader = new FileReader();
          reader.onload = e => {
            this.imageSrc = e.target.result;
          };
          reader.readAsDataURL(file);
          this.$emit('image-uploaded', this.isImageUploaded);
        }
        else if (file.type.startsWith('video/')) {
          // Handling when a video is uploaded
          this.isImageUploaded = false; // Reset the image flag
          this.isVideoUploaded = true; // Set a flag to indicate video upload
          this.videoSrc = URL.createObjectURL(file); // Store video source for displaying in the UI
          this.videoFile=file
          this.$emit('video-uploaded', this.isVideoUploaded);

        } else {
          // Handling other file types (if needed)
          console.log('Unsupported file type');
        }
      }
    },

    getVideoDuration() {
      const video = this.$refs.videoElement; // Reference to the video element
      this.videoDuration = video.duration; // Assign video duration
      this.currentTime = video.currentTime; // Assign current time
     
      // console.log(this.videoDuration,this.currentTime);
      // video.currentTime=30;
      // video.play();
    },
    startTimeModify(startTime){
      this.trimStartPoint=startTime
      const video = this.$refs.videoElement;
      video.currentTime=startTime;
      video.play();
      console.log(this.trimStartPoint);

    },
    endTimeModify(endTime){
      this.trimEndPoint=endTime
      console.log(this.trimEndPoint);
    },
    convertToHHMMSS(seconds) {
      const hours = Math.floor(seconds / 3600);
      const minutes = Math.floor((seconds % 3600) / 60);
      const remainingSeconds =Math.floor( seconds % 60);

      // Add leading zeros if necessary
      const formattedHours = hours.toString().padStart(2, '0');
      const formattedMinutes = minutes.toString().padStart(2, '0');
      const formattedSeconds = remainingSeconds.toString().padStart(2, '0');

      return `${formattedHours}:${formattedMinutes}:${formattedSeconds}`;
    },



    async mergeVideos() {
      const ffmpeg = createFFmpeg({ log: true });
      const outputFile = 'mergedvideo.mp4'; // Replace with your desired output file name

      try {
        console.log('Loading FFmpeg...');
        await ffmpeg.load();
        console.log('FFmpeg loaded successfully!');

        // Convert blob URLs to array buffers and write to FFmpeg FS
        for (let i = 0; i < this.videoList.length; i++) {
          const response = await fetch(this.videoList[i].src);
          const blob = await response.blob();
          const arrayBuffer = await blob.arrayBuffer();
          const videoFileName = `input${i}.mp4`; // Create a unique name for each video
          ffmpeg.FS('writeFile', videoFileName, new Uint8Array(arrayBuffer));
          this.videoList[i].src = videoFileName; // Update the src to the new path
        }

        // Update the file inputs for the FFmpeg command
        const fileInputs = this.videoList.map(video => `-i ${video.src}`).join(' ');

        console.log('Merging videos...');
        const scaleFilter = this.videoList.map((_, index) => `[${index}:v]scale=1920:1080[scaled${index}]`).join(';');
        const filterComplex = `${scaleFilter};[scaled0][0:a][scaled1][1:a]concat=n=${this.videoList.length}:v=1:a=1[v][a]`;

        await ffmpeg.run(
          '-y',
          ...fileInputs.split(' '),
          '-filter_complex', filterComplex,
          '-map', '[v]', '-map', '[a]',
          outputFile
        );


        // Read the resulting file
        const data = ffmpeg.FS('readFile', outputFile);
        // Use the data as needed, e.g., set it to a video element or create a download link
        const videoSrc = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }));
          this.$refs.videoElement.src=videoSrc
          this.videoSrc=videoSrc
          this.trimStartPoint=0;
          this.trimEndPoint=data.duration;

      } catch (error) {
        console.error('Error merging videos:', error);
        throw error;
      }
    },

    async trimVideo() {
        console.log(this.videoFile.URL);
      await this.ffmpegTrimVideo(this.videoFile,this.trimStartPoint,this.trimEndPoint);
    },
    async ffmpegTrimVideo(inputFile, startTime, endTime) {
      const ffmpeg = createFFmpeg({ log: true });
      const outputFile = 'trimmed_video.mp4'; // Replace 'trimmed_video.mp4' with your desired output file name
      const duration = endTime -startTime;
      try {
        console.log('Loading FFmpeg...');
        await ffmpeg.load();
        console.log('FFmpeg loaded successfully!');

        console.log('Reading input file...');
        await ffmpeg.FS('writeFile', inputFile.name, await fetchFile(inputFile));

        console.log('Trimming video...');
        await ffmpeg.run(
          '-i', inputFile.name,
          '-ss', this.convertToHHMMSS(startTime),
          '-t', this.convertToHHMMSS(duration),
          '-c', 'copy', outputFile
        );        
        console.log(startTime,endTime,duration);

        console.log('Writing output file...');
        const data = ffmpeg.FS('readFile', outputFile);
          // Set the trimmed video data to the <video> element
          // this.$refs.uploadedFile.files[0] = data;
          
          const videoSrc = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }));
          this.$refs.videoElement.src=videoSrc
          this.videoSrc=videoSrc
          this.trimStartPoint=0;
          this.trimEndPoint=data.duration;
            
      } catch (error) {
        console.error('Error during video trimming:', error);
        throw error;
      } 
    },

    addToMergeList() {
      // Check if the video file exists
      if (this.$refs.videoElement) {
        // Add the video file to the list (you may have a separate list defined)
        
        this.videoFile.src=this.videoSrc
        this.videoList.push(this.videoFile);
        // Reset the input file
        this.resetInput();
      } else {
        console.error('No video file available to add to the list.');
      }
    },
    resetInput() {
      // Reset the input file
      const input = this.$refs.uploadedFile;
      if (input) {
        input.value = ''; // Resetting the input value to clear the selected file
      }

      // Reset other necessary variables or flags if needed
      this.videoFile = null;
      this.isVideoUploaded = false;
      this.videoSrc = '';
      this.trimStartPoint = 0;
      this.trimEndPoint = 0;
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
.video-container{
  width: fit-content;
}

video{
  max-width: 100%;
  max-height: 600px;
  padding-top: 20px;
  padding-bottom: 20px;
  
}
</style>