<template>
  <v-col class='newstoryfiles overflow-y-auto' align-self="center" >
    <h2 class='newstoryfiles_title mb-8'>Report sighting</h2>
    <v-row class='newstoryfiles_body' align="center" justify="center" align-content="center">
      <v-form
        class='newstoryfiles_body_form mx-4'
        ref="form"
        v-model="valid"
        :lazy-validation="lazy"
      >

        <v-file-input accept="image/*"
          :loading="getShowProgress"
          full-width
          multiple
          label="Upload images of the sighting"
          @change="onPhotoFilePick"
          prepend-icon="mdi-camera"
          @click:clear="onClear">
        </v-file-input>


        <v-spacer>
        </v-spacer>

        <v-file-input
          :loading="getShowProgress"
          @change="onAudioFilePick"
          full-width
          accept="audio/*"
          label="Upload sounds of the sighting"
          prepend-icon="mdi-file-music"
          multiple
        >
        </v-file-input>

       <v-spacer>
        </v-spacer>

        <v-file-input
          @change="onVideoFilePick"
          :loading="getShowProgress"
          full-width
          accept="video/*"
          label="Upload videos of the sighting"
          prepend-icon="mdi-file-video"
          multiple
        >
        </v-file-input>

        <v-spacer>
        </v-spacer>

        <v-progress-linear
          v-model="getUploadProgress"
        ></v-progress-linear>

        <v-spacer>
        </v-spacer>
        <v-spacer>
        </v-spacer>
        <v-btn
          tile
          color="white"
          class="mt-4 mx-4"
          @click="$router.go(-1)"

        >
          Previous
        </v-btn>
        <v-btn
          :loading="getShowProgress"
          tile
          :disabled="!finished || getUploadProgress < 100"
          color="white"
          class="mt-4 mx-4"
          @click="submit"

        >
          Next
        </v-btn>
        <v-alert class='mt-6' v-if="error !== ''" type="error" tile>
        {{ error }}
        </v-alert>

      </v-form>
    </v-row>
  </v-col>
</template>
<script>
import { mapActions, mapGetters } from 'vuex'

export default {
  name: 'newstory',
  components: {
  },
  computed: {
    ...mapGetters('auth', [
      'getUser'
    ]),
    ...mapGetters('storage', [
      'getUploadProgress'
    ]),
    getShowProgress () {
      console.log(this.getUploadProgress)
      return this.getUploadProgress > 1 && this.getUploadProgress < 100 && !this.finished
    }
  },
  data: () => ({
    valid: true,
    lazy: false,
    error: '',
    finished: false,
    imageFiles: [],
    audioFiles: [],
    videoFiles: []
  }),
  methods: {
    ...mapActions('auth', [
      'login'
    ]),
    ...mapActions('story', [
      'storeCreatingStory'
    ]),
    ...mapActions('storage', [
      'uploadFile'
    ]),
    onClear () {
      this.finished = false
    },
    onAudioFilePick (audioFiles) {
      if (!audioFiles || audioFiles.length === 0) {
        this.finished = false
        return
      }
      this.finished = false
      this.audioFiles = []
      let timestamp = Date.now()
      audioFiles.forEach((file) => {
        let url = this.getUser.uid + '/audios/' + timestamp + '/' + file.name
        this.uploadFile({
          file: file,
          url: url
        }).then((downloadUrl) => {
          console.log('uploaded!' + downloadUrl)
          console.log(file)
          let dataToSave = {
            name: file.name,
            url: url,
            downloadUrl: downloadUrl
          }
          this.audioFiles.push(dataToSave)
          this.finished = true
        })
      })
    },
    onVideoFilePick (videoFiles) {
      if (!videoFiles || videoFiles.length === 0) {
        this.finished = false
        return
      }
      this.finished = false
      this.videoFiles = []
      let timestamp = Date.now()
      videoFiles.forEach((file) => {
        let url = this.getUser.uid + '/videos/' + timestamp + '/' + file.name
        this.uploadFile({
          file: file,
          url: url
        }).then((downloadUrl) => {
          console.log('uploaded!' + downloadUrl)
          console.log(file)
          let dataToSave = {
            name: file.name,
            url: url,
            downloadUrl: downloadUrl
          }
          this.videoFiles.push(dataToSave)
          this.finished = true
        })
      })
    },
    onPhotoFilePick (imageFiles) {
      if (!imageFiles || imageFiles.length === 0) {
        this.finished = false
        return
      }
      this.finished = false
      this.imageFiles = []
      let timestamp = Date.now()
      imageFiles.forEach((file) => {
        let url = this.getUser.uid + '/images/' + timestamp + '/' + file.name
        this.uploadFile({
          file: file,
          url: url
        }).then((downloadUrl) => {
          console.log('uploaded!' + downloadUrl)
          console.log(file)
          let dataToSave = {
            name: file.name,
            url: url,
            downloadUrl: downloadUrl
          }
          this.imageFiles.push(dataToSave)
          this.finished = true
        })
      })
    },
    submit () {
      let entity = {
        images: this.imageFiles,
        videos: this.videoFiles,
        audios: this.audioFiles,
        thumbnail: this.imageFiles[0]
      }
      this.storeCreatingStory(entity)

      this.$router.push('/create/3')
    },
    reset () {
      this.$refs.form.reset()
    },
    resetValidation () {
      this.$refs.form.resetValidation()
    },
  }
}
</script>
<style lang='sass'>

</style>
<style lang='sass' scoped>
.newstoryfiles
  max-height: 80vh

  &_title
    margin-top: 40px
  &_body
    &_form
      width: 100%
      max-width: 400px
</style>