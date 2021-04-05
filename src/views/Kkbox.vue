<template>
  <div class="kkbox">
    <h1>Guess the Song!</h1>
    <h3>Score: {{ score }}</h3>
    <h3>{{ gameStatus }}</h3>
    <a
      v-if="!userToken"
      href="https://account.kkbox.com/oauth2/authorize?redirect_uri=https://kkbox-oauth-helper.web.app/1091f0/getToken&client_id=85e66ad411cf7cee23e2873738475e53&response_type=code&state=test"
      >Login with KKBOX</a
    >
    <button v-if="userToken" @click="newSongQuiz">Guess it!</button>
    <answer-list
      v-if="randomTrack.length"
      :tracks="randomTrack"
      @handleUserAnswer="handleUserAnswer"
    ></answer-list>
    <track-player v-if="ansTrack.id" :trackID="ansTrack.id"></track-player>
  </div>
</template>

<script>
import axios from "axios";
import AnswerList from "@/components/AnswerList";
import TrackPlayer from "@/components/TrackPlayer.vue";

const TERRITORY = "TW";

function getRandomTrack(items, n = 1) {
  let res = [];
  let randomIndex = null;
  if (n <= items.length) {
    for (let i = 0; i < n; i++) {
      randomIndex = Math.floor(Math.random() * items.length);
      res = [...res, items[randomIndex]];
      items.splice(randomIndex, 1);
    }
  }
  return res;
}

export default {
  components: {
    AnswerList,
    TrackPlayer,
  },
  name: "Kkbox",
  data() {
    return {
      //   datas: [],
      randomTrack: [],
      ansTrack: [],
      seedTrackID: "8oRsSBpAlULSC9H9V8",
      gameStatus: "",
      score: 0,
      userToken: null,
      authStr: null,
    };
  },
  mounted() {
    let selfUri = new URL(window.location.href);
    let retObj = JSON.parse(selfUri.searchParams.get("ret"));
    // let state = selfUri.searchParams.get('state');

    if (retObj !== null) {
      if (typeof Storage == "undefined") {
        console.log("not support WebStorage");
      } else {
        localStorage.setItem("kkboxOAuth", JSON.stringify(retObj));
      }
    } else {
      console.log("getToken fail");
    }
    // window.location.replace('./kkbox?state=' + state);

    if (typeof Storage == "undefined") {
      console.log("not support WebStorage");
    } else {
      let kkboxOAuth = JSON.parse(localStorage.getItem("kkboxOAuth"));

      if (kkboxOAuth !== null && kkboxOAuth.access_token !== undefined) {
        this.userToken = kkboxOAuth.access_token;
        this.authStr = "Bearer " + this.userToken;
      }
    }
  },
  methods: {
    newSongQuiz() {
      this.gameStatus = "Loading...";
      let url = encodeURI(
        `https://api.kkbox.com/v1.1/me/recommended-seed-tracks/${this.seedTrackID}?q=territory=${TERRITORY}` //&limit=10
      );
      axios
        .get(url, { headers: { Authorization: this.authStr } })
        .then((res) => {
          const arr = res.data.tracks.data;

          this.randomTrack = getRandomTrack(arr, 4);

          const questionList = [...this.randomTrack];

          this.ansTrack = getRandomTrack(questionList)[0];

          this.gameStatus = "Guess it!";
        })
        .catch((err) => console.log(err));
    },
    handleUserAnswer(userAnswerID) {
      if (userAnswerID === this.ansTrack.id) {
        this.gameStatus = "Correct!";
        this.score += 10;
      } else this.gameStatus = "False!";

      console.log("Your score is: " + this.score);
    },
  },
};
</script>
