<template>
  <div>
    <!-- usersID : {{ usersID }}
    selectedUserID : {{ selectedUserID }}
    selectedAnswerDocument : {{ selectedAnswerDocument }}
    selectedAnswerSentiment : {{ selectedAnswerSentiment }}  -->
    <div v-if="usersID">
      <v-card
        flat
        class="task-container"
      >
        <v-row class="ma-0 pa-0">
          <!------------------------------------------------------------------------------------------------------------------------->
          <!--------------------------------------------- Answers List [Left] ------------------------------------------------------->
          <!------------------------------------------------------------------------------------------------------------------------->
          <v-col
            class="ma-0 pa-0 task-list"
            cols="3"
          >
            <v-list
              density="compact"
              class="list-scroll"
            >
              Ascendantly">
              <v-list-subheader>Evaluators</v-list-subheader>
              <v-divider />

              <v-list
                v-model:selected="selectedUserID"
                selection-mode="single"
              >
                <v-list-item
                  v-for="(item, i) in usersID"
                  :key="i"
                  :value="item"
                  :active="selectedUserID === item"
                  @click="selectedUserID = item"
                >
                  <v-list-item-title>
                    {{ getCooperatorEmail(item) }}
                    <!-- {{ item }} -->
                  </v-list-item-title>
                </v-list-item>
              </v-list>
            </v-list>
          </v-col>

          <!------------------------------------------------------------------------------------------------------------------------->
          <!--------------------------------------------- Vertical Line [Split] ----------------------------------------------------->
          <!------------------------------------------------------------------------------------------------------------------------->

          <v-divider
            vertical
            inset
          />

          <!------------------------------------------------------------------------------------------------------------------------->
          <!---------------------------------------------------------- Body [Right] ------------------------------------------------->
          <!------------------------------------------------------------------------------------------------------------------------->
          <v-col
            v-if="selectedAnswerDocument"
            class="ma-0 pa-1 answer-list"
            cols="9"
          >
            <!-- Co-operators -->
            <ModeratedTestCard
              :moderator="{ name: testDocument ? testDocument.testAdmin.email : '<Error>' }"
              :evaluator="{ name: selectedAnswerDocument ? getCooperatorEmail(selectedAnswerDocument.userDocId) : '<Error>' }"
            />

            <!-- Audio Wave -->
            <AudioWave
              ref="audioWave"
              v-model:active-region="activeRegion"
              :file="selectedAnswerDocument.cameraUrlEvaluator"
              :regions="selectedAnswerSentiment ? selectedAnswerSentiment.regions || [] : []"
            />

            <!-- Audio Wave End Banner-->
            <v-row class="align-center justify-space-between pa-3">
              <!-- Left Text -->
              <v-col
                cols="12"
                md="8"
              >
                <span class="text--secondary text-caption">
                  Drag the sliders to adjust the start and end points or enter the exact times in the input fields.
                </span>
              </v-col>

              <!-- Right Controls -->
              <v-col
                cols="12"
                md="4"
                class="text-right"
              >
                <v-btn
                  color="orange"
                  class="text-white"
                  @click="analyzeTimeStamp()"
                >
                  + Analyze
                </v-btn>
              </v-col>
            </v-row>

            <!-- Segments Transcripts Sentiment -->
            <SentimentTranscriptsList
              :play-segment="playSegmentInAudioWave"
              :regions="selectedAnswerSentiment ? selectedAnswerSentiment.regions || [] : []"
              :delete-region="deleteRegion"
            />
          </v-col>
        </v-row>
      </v-card>
    </div>

    <v-overlay
      :model-value="overlay.visible"
      class="text-center"
    >
      <v-progress-circular
        indeterminate
        color="#fca326"
        size="50"
      />
      <div class="white-text mt-3">
        {{ overlay.text }}
      </div>
    </v-overlay>

    <v-snackbar
      v-model="snackbar.visible"
      :color="snackbar.color"
      :timeout="4000"
    >
      {{ snackbar.text }}
      <template #actions>
        <v-btn
          color="white"
          variant="text"
          @click="snackbar.visible = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>
// External Libraries
import axios from 'axios'

// Components
import ModeratedTestCard from '@/components/molecules/ModeratedTestCard.vue';
import AudioWave from '@/components/molecules/AudioWave.vue';
import SentimentTranscriptsList from './SentimentTranscriptsList.vue';

// Controllers
import AudioSentimentController from '@/controllers/AudioSentimentController';

// Init audioSentimentController
const audioSentimentController = new AudioSentimentController();

export default {
  components: {
    ModeratedTestCard,
    AudioWave,
    SentimentTranscriptsList,
  },
  data() {
    return {
      selectedUserID: null, // Will store the selected user ID
      selectedAnswerSentiment: null, // The sentiment state for the selectedUserID
      // Active Region Data
      activeRegion: {
        start: 0,
        end: 5,
      },
      // State Management
      overlay: {
        visible: false,
        text: '',
      },
      snackbar: {
        visible: false,
        text: '',
        color: '', // Use a valid color name or hex code
      },
    };
  },
  computed: {
    testDocument() {
      return this.$store.getters.test; // Derived state
    },
    testAnswerDocument() {
      return this.$store.getters.testAnswerDocument.taskAnswers; // Access the store getter
    },
    usersID() {
      return Object.values(this.testAnswerDocument).map(answer => answer.userDocId);
    },
    // Compute selectedAnswerDocument based on selectedUserID
    selectedAnswerDocument() {
      return this.testAnswerDocument[this.selectedUserID] || null;
    },
  },
  watch: {
    selectedUserID: {
      immediate: true, // Runs when the component is mounted
      async handler(newUserId) {
        if (!newUserId) return;
        this.fetchSelectedAnswerSentiment();
      },
    },
  },
  mounted() {
    console.log('Component is mounted!');
  },
  methods: {
    getCooperatorEmail(userDocId) {
      let cooperatorEmail = null;
      if (this.testDocument.cooperators && Array.isArray(this.testDocument.cooperators)) {
        for (const element of this.testDocument.cooperators) {
          if (element && element.email && element.userDocId === userDocId) {
            cooperatorEmail = element.email;
          }
        }
      }
      return cooperatorEmail;
    },
    async fetchSelectedAnswerSentiment() {
      console.log('Fetching Sentiment Document..............................');
      const answerDocId = this.testDocument.answersDocId;
      const userDocId = this.selectedUserID;

      try {
        let result = await audioSentimentController.getByAnswerDocIdandUserDocId(answerDocId, userDocId);
        if (!result) {
          console.warn(`Sentiment document for answerDocId ${answerDocId} and userDocId ${userDocId} does not exist. Creating new document.`);
          const payload = {
            answerDocId: answerDocId,
            userDocId: userDocId,
          };
          result = await audioSentimentController.create(payload);
          console.warn(`Created new sentiment document with ID ${result.id}.`);
        }
        this.selectedAnswerSentiment = result;
      } catch (error) {
        console.error('Error fetching sentiment document:', error);
        this.selectedAnswerSentiment = null;
      }
    },
    playSegmentInAudioWave(start, end) {
      this.$refs.audioWave.playSegment(start, end);
    },
    async deleteRegion(region) {
      if (!confirm('Are you sure you want to delete this region?')) return;

      this.overlay = { visible: true, text: 'Deleting...' };

      try {
        await audioSentimentController.deleteSentimentRegion(this.selectedAnswerSentiment.id, region.idx);
        await this.fetchSelectedAnswerSentiment();
        this.overlay.visible = false;
        this.snackbar = {
          visible: true,
          color: 'success',
          text: 'Region Deleted Successfully',
        };
      } catch (error) {
        console.error('Error deleting region:', error);
        this.overlay.visible = false;
        this.snackbar = {
          visible: true,
          color: 'error',
          text: 'Error Deleting Region',
        };
      }
    },
    async analyzeTimeStamp() {
      console.log('Analyzing Timestamp..............................', this.activeRegion.start, this.activeRegion.end);
      this.overlay = { visible: true, text: 'Analyzing...' };

      try {
        const response = await axios.post('http://localhost:8001/audio-transcript-sentiment/process', {
          url: this.selectedAnswerDocument.cameraUrlEvaluator,
          start_time_ms: this.activeRegion.start * 1000.0, // Convert to milliseconds
          end_time_ms: this.activeRegion.end * 1000.0, // Convert to milliseconds
        });

        if (!response.data || response.data.status !== 'success') {
          throw new Error(`API Error: ${response.data?.error || 'Unknown error'}`);
        }

        const data = response.data.data;
        console.log('Analysis Completed', data);

        const utterances_sentiment = data.utterances_sentiment;
        const answerSentimentDocId = this.selectedAnswerSentiment.id;

        for (const utterance of utterances_sentiment) {
          await audioSentimentController.addSentimentRegion(answerSentimentDocId, {
            start: utterance.timestamp[0] + this.activeRegion.start,
            end: utterance.timestamp[1] + this.activeRegion.start,
            transcript: utterance.text,
            sentiment: utterance.label,
            confidence: utterance.confidence,
          });
        }

        await this.fetchSelectedAnswerSentiment();
        this.overlay.visible = false;
        this.snackbar = {
          visible: true,
          color: 'success',
          text: 'Analysis Completed',
        };
      } catch (error) {
        this.overlay.visible = false;
        this.snackbar = {
          visible: true,
          color: 'error',
          text: 'Analysis Failed',
        };
        console.error('Error:', error);
      }
    },
  },
};
</script>

<style>
.cardsTitle {
  color: #455a64;
  font-size: 18px;
  font-weight: 600;
}
.list-scroll {
  height: 508px;
  overflow: auto;
}
.list-scroll::-webkit-scrollbar {
  width: 7px;
}
.list-scroll::-webkit-scrollbar-track {
  background: none;
}
.list-scroll::-webkit-scrollbar-thumb {
  background: #ffcd86;
  border-radius: 4px;
}
.list-scroll::-webkit-scrollbar-thumb:hover {
  background: #fca326;
}
</style>