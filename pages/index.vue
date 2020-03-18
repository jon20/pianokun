<template>
  <div>
    <b-field>
      <b-input
        placeholder="再生したいyoutubeのurlを貼り付けてください"
        v-model="search"
        icon="magnify"
      >
      </b-input>
      <p class="control">
        <button
          @click="searchVideo"
          class="button is-primary"
        >Search</button>
      </p>
    </b-field>
    <div class="container is-fullhd is-three-quarters">
      <youtube
        :video-id="videoId"
        :player-vars="playerVars"
        ref="youtube"
        @playing="playing"
        @onStateChange="onStateChange"
      ></youtube>
    </div>
    <div class="items columns">

      <b-button
        type="is-success"
        class="is-offset-2"
        @click="rewindVideo"
      >
        <b-icon icon="chevron-left"></b-icon>
      </b-button>
      <b-button
        type="is-success"
        @click="fastVideo"
      >
        <b-icon icon="chevron-right"></b-icon>
      </b-button>

      再生速度
      <b-select
        placeholder="1"
        v-model="playbackControll"
        @input="setRate"
      >
        <option
          v-for="option in playbackControlls"
          :value="option"
          :key="option.id"
        >
          {{ option }}
        </option>
      </b-select>
      <b-field label="リピート">
        <b-timepicker
          v-model="repeatStart"
          inline
          class="is-1"
        ></b-timepicker>
        〜
        <b-timepicker
          v-model="repeatEnd"
          inline
        ></b-timepicker>
        <b-button
          type="is-success"
          @click="repeatVideo"
        >
          Start
        </b-button>
        <b-button
          type="is-danger"
          @click="stopRepeatVideo"
        >
          Stop
        </b-button>
      </b-field>
    </div>
    <b-collapse
      class="card"
      aria-id="contentIdForA11y3"
    >
      <div
        slot="trigger"
        slot-scope="props"
        class="card-header"
        role="button"
        aria-controls="contentIdForA11y3"
      >
        <p class="card-header-title">
          カスタム
        </p>
        <a class="card-header-icon">
          <b-icon :icon="props.open ? 'menu-down' : 'menu-up'">
          </b-icon>
        </a>
      </div>
      <div class="card-content">
        <b-field label="スキップ間隔">
          <b-input
            type="float"
            style="width:70px"
            v-model="movementInterval"
          ></b-input>
        </b-field>
        <b-field label="再生速度">
          <b-input
            type="float"
            style="width:70px"
            v-model="playbackControll"
            @input="setRate"
          ></b-input>
        </b-field>
        <b-field label="リピート後の待ち時間">
          <b-input
            type="float"
            style="width:70px"
            v-model="repeatIntervalTime"
            @input="setRate"
          ></b-input>
        </b-field>
      </div>

    </b-collapse>
  </div>
</template>

<script>
import Logo from "~/components/Logo.vue";
import { async } from "q";
const wait = sec => {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, sec * 1000);
    //setTimeout(() => {reject(new Error("エラー！"))}, sec*1000);
  });
};
export default {
  components: {
    Logo
  },
  data() {
    return {
      videoId: "DN_e_5ckvZk",
      playerVars: {
        autoplay: 1,
        //controls: 0,
        iv_load_policy: 3,
        modestbranding: 1,
        rel: 0
      },
      movementInterval: 5,
      playbackControlls: [0.25, 0.5, 0.75, 1, 2, 3, 4, 5, 6],
      playbackControll: 1,
      repeatStart: new Date("1995-12-17T00:00:00"),
      repeatEnd: new Date("1995-12-17T00:00:00"),
      repeatInterval: null,
      repeatIntervalTime: 0,
      isRepeat: false,
      undo: null,
      redo: null,
      url: "",
      search: ""
    };
  },
  methods: {
    urlToVideoID() {
      let queryString = this.url;
      console.log(
        [...new URLSearchParams(queryString).entries()].reduce(
          (obj, e) => ({ ...obj, [e[0]]: e[1] }),
          {}
        )
      );
    },
    playVideo() {
      console.log(this.player.getPlaybackRate());
      this.player.playVideo();
    },
    playing() {
      console.log("o/ we are watching!!!");
    },
    setRate() {
      this.player.setPlaybackRate(this.playbackControll);
      console.log(this.playbackControll);
      console.log(this.player.getAvailablePlaybackRates());
    },
    rewindVideo() {
      this.player.getCurrentTime().then(time => {
        this.player.seekTo(time - parseInt(this.movementInterval, 10));
      });
    },
    fastVideo() {
      this.player.getCurrentTime().then(time => {
        this.player.seekTo(time + parseInt(this.movementInterval, 10));
      });
    },
    onStateChange() {
      //setTimeout(this.stopVideo, 2000);
    },
    stopVideo() {
      this.player.pauseVideo();
    },
    stopRepeatVideo() {
      clearInterval(this.repeatInterval);
    },
    repeatVideo() {
      if (this.repeatInterval !== null) {
        clearInterval(this.repeatInterval);
      }
      let st = this.repeatStart.getHours() * 60 + this.repeatStart.getMinutes();
      let ed = this.repeatEnd.getHours() * 60 + this.repeatEnd.getMinutes();
      this.repeatInterval = setInterval(() => {
        this.player.getCurrentTime().then(async time => {
          if (time > ed) {
            if (this.repeatIntervalTime > 0 && !this.isRepeat) {
              this.player.pauseVideo();
              this.isRepeat = true;
              console.log(this.repeatIntervalTime);
              await wait(this.repeatIntervalTime);
              this.isRepeat = false;
              this.player.playVideo();
            }
            this.player.seekTo(st);
          }
        });
      }, 1000);
      this.player.seekTo(st).then(() => {
        //this.player.playVideo();
      });

      //setTimeout(this.stopVideo, (ed - st) * 1000 + 1000);
      //this.player.addEventListener("onStateChange", this.repv);
    },
    wait(sec) {
      return new Promise((resolve, reject) => {
        setTimeout(resolve, sec * 1000);
        //setTimeout(() => {reject(new Error("エラー！"))}, sec*1000);
      });
    },
    searchVideo() {
      console.log("hoe");
      console.log(this.search);
      if (!this.search.includes("https")) {
        this.search = "htps://" + this.search;
      }
      const ur = new URL(this.search);
      const pm = new URLSearchParams(ur.search);
      this.videoId = pm.get("v");
    }
  },
  computed: {
    player() {
      return this.$refs.youtube.player;
    }
  },
  mounted() {
    this.player.addEventListener("onStateChange", this.onStateChange);
  }
};
</script>

<style>
iframe {
  border: none !important;
  position: absolute !important;
  width: 100vw;
  height: 80vh;
}
.items {
  padding-top: 85vh;
  padding-left: 10vw;
}
</style>
