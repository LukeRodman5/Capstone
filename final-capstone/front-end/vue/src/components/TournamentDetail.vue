<template>
  <div>
    <div class="loading" v-if="isLoading">
      <img id="loadingImg" src="../assets/ping_pong_loader.gif" />
    </div>
    <div v-else id="tourney-desc">
      <h1>{{ currentTournament.tourneyName }}</h1>
      <p>{{ currentTournament.tourneyDesc }}</p>
      <p>Max Number of Participants: {{ currentTournament.maxNumOfParticipants }}</p>
      <p> Start Date: {{currentTournament.startDate}} | End Date: {{currentTournament.endDate}}</p>
      <button class="btnJoinTourney" v-on:click="joinTourney(currentTournament.tourneyId, $store.state.user.username)">Join Tournament</button>
      <generate-tournament
        v-bind:currentTourney=currentTournament
        v-bind:usersInTourney=usersInTourney
        v-bind:matchesInTourney=matchesInTourney />
    </div>
  </div>
</template>

<script>
import applicationServices from "../services/ApplicationServices";
import GenerateTournament from './GenerateTournament.vue';

export default {
  name: "tournament-detail",
  components: {
    GenerateTournament
  },
  data() {
    return {
      isLoading: true,
      errorMsg: "",
      currentTournament: {},
      usersInTourney: [],
      matchesInTourney: []
    };
  },
  methods: {
    deleteTournament() {
      if (confirm("Are you sure you want to delete this tournament? This action cannot be undone.")) {
        applicationServices.deleteTournament(this.currentTournament.tourneyID).then(response => {
          if (response.status === 200) {
            alert("Tournament successfully deleted");
            this.$router.push(`/`);
          }
        }).catch(error => {
          if (error.response) {
            this.errorMsg =
              "Error deleting tournament. Response received was '" +
              error.response.statusText +
              "'.";
          } else if (error.request) {
            this.errorMsg =
              "Error deleting tournament. Server could not be reached.";
          } else {
            this.errorMsg =
              "Error deleting tournament. Request could not be created.";
          }
        })
      }
    },//end of delete card
    joinTourney(tourneyId, username){
      applicationServices.joinTourney(tourneyId, username, 'Approved')
      .then(response=>{
        if(response.status===200 || response.status===201){
          alert("You have successfully joined this tournament")
        }
        else{
          alert("Attempt to join this tournament was unsuccessful.")
        }
      this.$router.push("/")
      }) //end of then
    
    },
    retrieveTournament() {
      applicationServices.getTournament(this.$route.params.tourneyID).then(response => {
        this.currentTournament = response.data
        this.getMatchesInTourney(this.$route.params.tourneyID)
        this.isLoading = false
      })
    },
    getMatchesInTourney(tourneyId) {
      applicationServices.getMatchesInTourney(tourneyId).then((response) => {
        if (response.status === 200 || response.status === 201) {
          this.matchesInTourney = response.data
          if (this.matchesInTourney.length === 0) {
            this.SetUpMatches()
          }
        }
      })
    },
    getUsersInTourney() {
      applicationServices.getUsersInTourney(this.$route.params.tourneyID).then((response) => {
        if (response.status === 200 || response.status === 201) {
          this.usersInTourney = response.data
        } else {
          console.log("Error")
        }
      })
    },
    SetUpMatches() {
      let roundCount = 0
      let match = {}
      let matchesCount = this.currentTournament.maxNumOfParticipants / 2
      
      for (let i = 0; i < matchesCount; i++) {
        match = {startDate: null, startTime: null, roundLevel: roundCount}
        applicationServices.createMatch(match, this.$route.params.tourneyID).then((response) => {
          if (response.status === 200 || response.status === 201) {
            this.matchesInTourney.push(response.data)
          } else {
            console.log("Create match error")
          }
        })
        if (i === matchesCount - 1 && matchesCount > 1) {
          i = -1
          matchesCount = matchesCount / 2
          roundCount++
        }
      }
    }
  },//end of methods
  created() {
    this.retrieveTournament(),
    this.getUsersInTourney()
  }
};
</script>

<style scoped>
#bracket {
  display: flex;
  justify-content: center;
}
#tourney-desc, #tourney-desc>h2 {
  text-align: center;
}
#tourney-desc {
  margin-bottom: 30px;
}
.btn.editTournament {
  color: #fff;
  background-color: #508ca8;
  border-color: #508ca8;
  margin-bottom: 10px;
}
#loadingImg {
  height: 200px;
  width:  200px;
}
.btn.deleteTournament {
  color: #fff;
  background-color: #ef031a;
  border-color: #ef031a;
  margin-bottom: 10px;
}
</style>