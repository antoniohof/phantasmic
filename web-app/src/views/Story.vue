<template>
  <v-col
    v-if="story"
    class="story fill-height fill-width"
    align="center"
    justify="center"
    align-content="center"
  >
    <v-app-bar class="story_header" color="black accent-4" dense dark>
      <v-toolbar-title>{{ story.title }}</v-toolbar-title>
      <v-chip label>
        {{ getStoryDate(story) }}
      </v-chip>
      <v-chip label>
        {{ story.time }}
      </v-chip>
      <v-spacer></v-spacer>
      <v-menu
        v-if="showOptions"
        :offset-y="true"
        right
        bottom
        class="story_header_menu"
      >
        <template v-slot:activator="{ on, attrs }">
          <v-btn
            icon
            v-bind="attrs"
            v-on="on"
          >
            <v-icon>mdi-dots-vertical</v-icon>
          </v-btn>
        </template>

        <v-list>
          <v-list-item @click="onClickDelete" class="story_header_menu_item">
            <v-list-item-title>DELETE</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>


    </v-app-bar>
    <v-col class="story_body">
      <MapBox
        class="map"
        :stories="[story]"
        locked />
      <p class="story_body_description">{{ story.description }}</p>
      <v-carousel v-if="displayImages" cycle hide-delimiter-background show-arrows-on-hover>
        <v-carousel-item v-for="(image, i) in story.images" :src="image.downloadUrl" :key="i" contain max-height="80%"
        ></v-carousel-item>
      </v-carousel>
      <v-col v-if="displayAudio">
        <vuetify-audio
         v-for="(audio, i) in story.audios"
         :file="audio.downloadUrl"
         color="black"
         :key="i"
         downloadable>
         </vuetify-audio>
      </v-col>
      <v-col v-if="displayVideo">
        <vue-core-video-player
          v-for="(video, i) in story.videos"
          :src="video.downloadUrl"
          :key="i"
          downloadable
          color="black"
          >
         </vue-core-video-player>
      </v-col>
    </v-col>
  </v-col>
</template>
<script>

import { mapActions, mapGetters } from 'vuex'
import {
  MapBox
} from '@/components'
export default {
  name: 'newstorypick',
  components: {
    MapBox,
    VuetifyAudio: () => import('vuetify-audio')
  },
  computed: {
        ...mapGetters('story',
      [
        'getStories'
      ]),
      ...mapGetters('auth', [
        'getUser'
      ]),
      showOptions () {
        return this.story.authorID === this.getUser.uid
      },
      displayAudio () {
        return this.story.audios && this.story.audios.length > 0
      },
      displayVideo () {
        return this.story.videos && this.story.videos.length > 0
      },
      displayImages () {
        return this.story.images && this.story.images.length > 0
      }
  },
  mounted () {
    this.setCurrentStory(null)
    this.fetchStories().then(() => {
      this.story = this.getStories.find((story) => {
        return story.id === this.$route.params.id
      })
    })
    console.log(this.story)
  },
  data: () => ({
    story: null,
    file: 'http://www.hochmuth.com/mp3/Boccherini_Concerto_478-1.mp3'
  }),

  methods: {
    getStoryDate (story) {
      return this.$moment(story.date).format('DD-MM-YYYY')
    },
    ...mapActions('auth', [
      'login'
    ]),
    ...mapActions('story', [
      'fetchStories',
      'deleteStory'
    ]),
    ...mapActions('explore', [
      'setCurrentStory'
    ]),
    submit () {
    },
    reset () {
      this.$refs.form.reset()
    },
    resetValidation () {
      this.$refs.form.resetValidation()
    },
    onClickDelete () {
      this.deleteStory(this.story).then(() => {
        this.$router.push('/')
      })
    }
  }
}
</script>

<style lang='sass' scoped>
.map
  pointer-events: none !important
  margin-top: 5px
  width: 100vw !important
  height: 150px !important

.story
  height: 100%
  &_header
    position: fixed
    z-index: 1000
    &_menu
      &_item
        padding-left: 4px
        padding-right: 4px
        cursor: pointer
  &_body
    height: calc(100vh - 140px)
    margin-top: 50px
    overflow-x: hidden
    overflow-y: scroll
    &_description
      margin-top: 20px
      margin-bottom: 20px
      height: fit-content
      text-align: left
      color: black

.v-chip
  padding: 5px
  margin-left: 20px
</style>