<template>
  <div class="kkbox">
    <div class="text-effect">
      <h1>Guess the Song!</h1>
    </div>
    <categories-selector
      :categories="categories"
      @handleSelectCategory="handleSelectCategory"
    ></categories-selector>
    <h3>
      Score: {{ score }} / {{ totalSocre }} | Quiz left: {{ totalQuizCount }}
    </h3>
    <h3>Category: {{ currentCategory }}</h3>
    <h3>{{ gameStatus }}</h3>
    <div class="game-button-container">
      <a
        class="white"
        v-if="!userToken"
        href="https://account.kkbox.com/oauth2/authorize?redirect_uri=https://kkbox-oauth-helper.web.app/1091f0/getToken&client_id=85e66ad411cf7cee23e2873738475e53&response_type=code&state=test"
      >
        <p>
          <span class="bg"></span>
          <span class="base"></span>
          <span class="text">Login With KKBOX</span>
        </p>
      </a>
    </div>

    <div class="game-button-container">
      <a href="#" class="white" v-if="userToken" @click="newSongQuiz">
        <p>
          <span class="bg"></span>
          <span class="base"></span>
          <span class="text">{{ controlButtonText }}</span>
        </p>
      </a>
    </div>
    <answer-list
      v-if="randomTrack.length"
      :tracks="randomTrack"
      :isDisable="acceptAnswerInput"
      @handleUserAnswer="handleUserAnswer"
    ></answer-list>
    <track-player v-if="ansTrack.id" :trackID="ansTrack.id"></track-player>
  </div>
</template>

<script>
import axios from "axios";
import AnswerList from "@/components/AnswerList";
import TrackPlayer from "@/components/TrackPlayer.vue";
import CategoriesSelector from "@/components/CategoriesSelector";

const TERRITORY = "TW";
const QUIZ_COUNT = 10;
const SCORE_STEP = 10;

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
    CategoriesSelector,
  },
  name: "Kkbox",
  data() {
    return {
      //   datas: [],
      categories: [],
      randomTrack: [],
      ansTrack: [],
      currentCategory: "",
      seedTrackID: "",
      gameStatus: "",
      controlButtonText: "Initizaling...",
      totalQuizCount: QUIZ_COUNT,
      score: 0,
      acceptAnswerInput: true,
      userToken: null,
      authStr: null,
    };
  },
  mounted() {
    let selfUri = new URL(window.location.href);
    let retObj = JSON.parse(selfUri.searchParams.get("ret"));
    // let state = selfUri.searchParams.get("state");

    if (retObj !== null) {
      if (typeof Storage == "undefined") {
        console.log("not support WebStorage");
      } else {
        localStorage.setItem("kkboxOAuth", JSON.stringify(retObj));
        window.location.replace(selfUri.origin + "/guess-the-song");
      }
    } else {
      console.log("getToken fail");
    }

    if (typeof Storage == "undefined") {
      console.log("not support WebStorage");
    } else {
      let kkboxOAuth = JSON.parse(localStorage.getItem("kkboxOAuth"));

      if (kkboxOAuth !== null && kkboxOAuth.access_token !== undefined) {
        this.userToken = kkboxOAuth.access_token;
        this.authStr = "Bearer " + this.userToken;
        this.controlButtonText = "Start";

        let url = encodeURI(
          `https://api.kkbox.com/v1.1/new-release-categories?territory=${TERRITORY}&offset=1`
        );
        axios
          .get(url, { headers: { Authorization: this.authStr } })
          .then((res) => {
            this.categories = [...res.data.data];
          })
          .catch((err) => console.log(err));
      }
    }
  },
  methods: {
    newSongQuiz() {
      if (this.seedTrackID === "") {
        this.gameStatus = "Please select a category first";
        return null;
      }
      if (this.totalQuizCount > 0) {
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
            this.acceptAnswerInput = true;
            this.totalQuizCount -= 1;
            this.gameStatus = "Guess it!";
            this.controlButtonText = "Next quiz";
          })
          .catch((err) => console.log(err));
      } else {
        this.controlButtonText = "Play again!";
        this.gameStatus = "Game ended! Your total score is: " + this.score;
        this.score = 0;
        this.totalQuizCount = QUIZ_COUNT;
      }
    },
    handleSelectCategory(categoryID, categoryTitle) {
      this.score = 0;
      this.totalQuizCount = QUIZ_COUNT;
      this.acceptAnswerInput = false;
      let url = encodeURI(
        `https://api.kkbox.com/v1.1/radio-categories/${categoryID}/tracks?q=territory=${TERRITORY}` //&limit=10
      );
      axios
        .get(url, { headers: { Authorization: this.authStr } })
        .then((res) => {
          console.log(categoryTitle);
          this.currentCategory = categoryTitle;
          let albumID = res.data.data[0].album.id;
          url = encodeURI(
            `https://api.kkbox.com/v1.1/albums/${albumID}/tracks?q=territory=${TERRITORY}&limit=1`
          );
          axios
            .get(url, { headers: { Authorization: this.authStr } })
            .then((res) => {
              this.seedTrackID = res.data.data[0].id;
            })
            .catch((err) => console.log(err));
        })
        .catch((err) => console.log(err));
    },
    handleUserAnswer(userAnswerID) {
      if (!this.acceptAnswerInput) return null;
      if (userAnswerID === this.ansTrack.id) {
        this.gameStatus =
          "Correct! The answer is: " +
          this.ansTrack.name +
          " by " +
          this.ansTrack.album.artist.name;
        this.score += SCORE_STEP;
      } else
        this.gameStatus =
          "False! The answer is: " +
          this.ansTrack.name +
          " by " +
          this.ansTrack.album.artist.name;
      this.acceptAnswerInput = false;
      console.log("Your score is: " + this.score);
    },
  },
  computed: {
    totalSocre() {
      return QUIZ_COUNT * SCORE_STEP;
    },
  },
};
</script>

<style lang="scss">
@import "@/assets/master.scss";
</style>
