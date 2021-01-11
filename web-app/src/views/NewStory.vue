<template>
  <v-col class='newstory overflow-y-auto' align-content="center">
    <h2 class='newstory_title mb-8'>Report sighting</h2>
    <v-row class='newstory_body' align="center" justify="center" align-content="center">
      <v-form
        class='newstory_body_form mx-4'
        ref="form"
        v-model="valid"
        :lazy-validation="lazy"
      >
        <!--
        <v-text-field
          v-model="name"
          :counter="10"
          :rules="nameRules"
          label="Name"
          required
        ></v-text-field>
        -->

        <v-text-field
          v-model="title"
          :rules="titleRules"
          label="Title"
          required
        ></v-text-field>


        <v-textarea
          v-model="description"
          name="input-7-1"
          filled
          label="Description"
          auto-grow
          full-width
          value=""
          :rules="descriptionRules"
        ></v-textarea>

        <v-spacer></v-spacer>

        <v-dialog
          ref="dialogDate"
          v-model="dateModal"
          :return-value.sync="date"
          persistent
          width="290px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="date"
              label="Date"
              prepend-icon="mdi-calendar"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker
            v-model="date"
            scrollable
          >
            <v-spacer></v-spacer>
            <v-btn
              text
              color="primary"
              @click="dateModal = false"
            >
              Cancel
            </v-btn>
            <v-btn
              text
              color="primary"
              @click="$refs.dialogDate.save(date)"
            >
              OK
            </v-btn>
          </v-date-picker>
        </v-dialog>

        <v-spacer></v-spacer>

        <v-dialog
          ref="dialogTime"
          v-model="timeModal"
          :return-value.sync="time"
          persistent
          width="290px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="time"
              label="Time"
              prepend-icon="mdi-clock-time-four-outline"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-time-picker
            v-if="timeModal"
            v-model="time"
            full-width
            format="24hr"
          >
            <v-spacer></v-spacer>
            <v-btn
              text
              color="primary"
              @click="timeModal = false"
            >
              Cancel
            </v-btn>
            <v-btn
              text
              color="primary"
              @click="$refs.dialogTime.save(time)"
            >
              OK
            </v-btn>
          </v-time-picker>
        </v-dialog>

        <!--

        <v-text-field
          v-model="city"
          :rules="cityRules"
          label="City"
          required
        ></v-text-field>

        <v-text-field
          v-model="country"
          :rules="countryRules"
          label="Country"
          required
        ></v-text-field>
        <v-select
          class='mt-4'
          v-model="crimeType"
          :items="types"
          :rules="[v => !!v || 'Type is required']"
          label="Type"
          required
        ></v-select>

        <v-text-field
          v-if="crimeType === 'Other'"
          v-model="specificCrimeType"
          :rules="[v => !!v || 'Specify type is required']"
          label="Please specify"
          required
        ></v-text-field>
        -->
        <v-spacer></v-spacer>

        <v-btn
          :loading="loading"
          tile
          :disabled="!valid"
          color="white"
          class="mt-4"
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
  mounted () {
    if (this.getCreatingStory) {
      this.title = this.getCreatingStory.title
      this.description = this.getCreatingStory.description
      /*
      this.crimeType = this.getCreatingStory.type
      this.city = this.getCreatingStory.city
      this.country = this.getCreatingStory.country
      */
     this.date = this.getCreatingStory.date
     this.time = this.getCreatingStory.time
    }
  },
  computed: {
    ...mapGetters('auth', [
      'getUser'
    ]),
    ...mapGetters('story', [
      'getCreatingStory'
    ])
  },
  data: () => ({
    valid: true,
    title: '',
    titleRules: [
      v => !!v || 'Title is required'
    ],
    description: '',
    descriptionRules: [
      v => !!v || 'Description is required'
    ],
    city: '',
    cityRules: [
      v => !!v || 'City is required'
    ],
    country: '',
    countryRules: [
      v => !!v || 'Country is required'
    ],
    crimeType: '',
    types: [
      'Illegal logging',
      'Envinronmental Killings',
      'Poaching/Hunting',
      'Abandoned vehicle',
      'Bushmeat',
      'Wildlife smuggling',
      'Illegal construction',
      'Illegal dumping',
      'Illegal mining',
      'Illegal fishing',
      'Littering',
      'Dumping',
      'Irregular toxic waste disposal',
      'Irregular e-waste disposal',
      'Oil spill',
      'Water pollution'
    ],
    lazy: false,
    loading: false,
    error: '',
    specificCrimeType: '',
    date: new Date().toISOString().substr(0, 10),
    time: '11:15',
    timeModal: false,
    dateModal: false
  }),

  methods: {
    ...mapActions('auth', [
      'login'
    ]),
    ...mapActions('story', [
      'storeCreatingStory'
    ]),
    submit () {
      let entity = {
        title: this.title,
        description: this.description,
        date: this.date,
        time: this.time,
        // type: this.crimeType === 'Other' ? this.crimeType : this.specificCrimeType,
        thumbnail: '',
        zoom: 0,
        authorName: this.getUser.displayName,
        authorID: this.getUser.uid,
        // city: this.city,
        // country: this.country,
        geometry: {
          coordinates: []
        },
        images: [
        ],
        videos: [
        ],
        audios: [
        ]
      }

      this.storeCreatingStory(entity)
      this.$router.push('/create/2')
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

<style lang='sass' scoped>
.newstory
  max-height: 80vh

  &_title
    margin-top: 40px
  &_body
    &_form
      width: 100%
      max-width: 400px
</style>