<template>
  <div class="pageContent">
    <Loading v-if="loading" />
    <MessageError v-if="error" message="No Training Entries Found" />
    <div v-if="!loading && !error">
      <HeadingPage name="Training Log" />
      <SpacerSmall />
      <ViewSession
        v-for="s in trainingLog.sessions"
        :key="s.id"
        :role="user.role"
        :session="s" />
    </div>
    <!-- <div v-if="!loading && !error">
      <button v-if="this.pageIndex > 1" class="actionBtn" @click="previousPage">
        Previous
      </button>
      <button class="actionBtn" @click="nextPage">
        Next
      </button>
    </div> -->
  </div>
</template>

<script>
import axios from 'axios'
axios.defaults.withCredentials = true;
const url = 'https://coach-easy-deploy.herokuapp.com';

import Loading from '~/components/Loading'
import HeadingPage from '~/components/HeadingPage'
import SpacerSmall from '~/components/SpacerSmall'
import ViewSession from '~/components/ViewSession'
import MessageError from '~/components/MessageError'

export default {
  components: {
    Loading,
    HeadingPage,
    SpacerSmall,
    ViewSession,
    MessageError
  },
  data() {
    return {
      loading: true,
      trainingLog: undefined,
      user: undefined,
      pageIndex: undefined,
      error: false,
    }
  },
  methods: {
    getUserData: function(){
      let self = this;
      Promise.all([ this.$store.state.userData ]).then( () => {
        self.user = this.$store.state.userData;
        self.getTrainingLog();
      },() => {
        self.loadingFailed = true
      })
    },
    setLoading: function(){
      this.loading=false;
    },
    getTrainingLog: function(){
      let self = this;
      let reqID = -1;
      const queryString = window.location.search;
      const urlParams = new URLSearchParams(queryString);
      const client_id = urlParams.get('client_id');
      if (!this.pageIndex) {
        this.pageIndex = urlParams.get('index');
      }
      if(self.user.role === 'CLIENT'){
        reqID = self.user.id
      } else {
        reqID = client_id
      }
      //&page=${this.pageIndex}&page_size=1
      axios.get(`${url}/client/trainingLog?client_id=${reqID}`)
      .then(result => {
        if (!result.data.sessions.length > 0) {
          self.setLoading();
          self.error = true;
          // this.previousPage();
        } else {
          self.trainingLog = result.data
          self.setLoading();
          self.error = false;
        }
      }).catch(error => {
        self.loading = false;
        self.error = true;
        console.log(error)
      });
    },
    previousPage: function() {
      if (this.pageIndex > 1) {
        this.pageIndex--;
        this.getTrainingLog();
      }
   
    },
    nextPage: function() {
      this.pageIndex++;
      this.getTrainingLog();
    }
  },
  mounted() {
    this.getUserData();
  },
}
</script>

<style>

</style>