<template>
  <div class="main">
    <div class="explanation">Fetch last 15 matches of any Dota 2 player</div>
    <button @click="fetchData(), fetchPlayer()" class="fetch-button">
      Fetch data
    </button>
    <div class="playerbox">
      <div id="player-name" class="hide"></div>
      <img id="player-pfp" src="" alt="" class="hide" />
    </div>
    <button @click="filterLiked" class="fetch-button filter-button hide">
      Liked filter
    </button>
    <div class="matches">
      <div v-for="(match, k) in filteredList" :key="k" class="match">
        <button class="close-btn" @click="removeSelf(k)">
          <svg viewBox="0 0 31 31" xmlns="http://www.w3.org/2000/svg">
            <line
              x1="3.53553"
              y1="4.46447"
              x2="26.163"
              y2="27.0919"
              stroke="white"
              stroke-width="10"
            />
            <line
              x1="26.5355"
              y1="4.53553"
              x2="3.90812"
              y2="27.163"
              stroke="white"
              stroke-width="10"
            />
          </svg>
        </button>

        <div
          v-if="
            (match.radiant_win == true &&
              isRadiant(match.player_slot) == true) ||
            (match.radiant_win == false &&
              isRadiant(match.player_slot) == false)
          "
          class="match-won"
        >
          WIN
        </div>
        <div v-else class="match-lost">LOSE</div>
        <div>
          {{ convertStartTime(match.start_time) }}
        </div>
        <div class="duration">
          Match duration: {{ convertTime(match.duration) }}
        </div>
        <div>KDA: {{ match.kills }}/{{ match.deaths }}/{{ match.assists }}</div>
        <button
          class="like-btn"
          @click="switchLike(k)"
          v-if="match.liked == false"
        >
          <svg
            width="396"
            height="396"
            viewBox="0 0 396 396"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <circle
              cx="198"
              cy="198"
              r="190.5"
              stroke="#20C93C"
              stroke-width="15"
            />
            <path
              fill-rule="evenodd"
              clip-rule="evenodd"
              d="M112.645 137.645C132.171 118.118 163.829 118.118 183.355 137.645L198 152.289L212.645 137.645C232.171 118.118 263.829 118.118 283.355 137.645C302.882 157.171 302.882 188.829 283.355 208.355L198 293.711L112.645 208.355C93.1184 188.829 93.1184 157.171 112.645 137.645Z"
              stroke="#20C93C"
              stroke-width="15"
              stroke-linejoin="round"
            />
          </svg>
        </button>
        <button @click="switchLike(k)" class="like-btn" v-else>
          <svg
            width="396"
            height="396"
            viewBox="0 0 396 396"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              fill-rule="evenodd"
              clip-rule="evenodd"
              d="M198 396C307.352 396 396 307.352 396 198C396 88.6476 307.352 0 198 0C88.6476 0 0 88.6476 0 198C0 307.352 88.6476 396 198 396ZM183.355 137.645C163.829 118.118 132.171 118.118 112.645 137.645C93.1184 157.171 93.1184 188.829 112.645 208.355L162.645 258.355L198 293.711L233.355 258.355L283.355 208.355C302.882 188.829 302.882 157.171 283.355 137.645C263.829 118.118 232.171 118.118 212.645 137.645L198 152.289L183.355 137.645Z"
              fill="#20C93C"
            />
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "PlayerMatches",
  data() {
    return {
      matches: [],
      player: [],
      playerID: "",
      likeFilter: false,
    };
  },
  methods: {
    fetchData() {
      this.playerID = prompt(
        "Enter ID of the player you want information of. Example: https://www.opendota.com/players/321580662"
      );
      if (this.playerID == null || this.playerID.length == 0) {
        this.playerID = "321580662";
      }
      fetch(
        `https://api.opendota.com/api/players/${this.playerID}/matches?limit=15`,
        {
          method: "GET",
        }
      )
        .then((response) => {
          if (response.ok) {
            response.json().then((data) => {
              this.matches = data;
              this.matches.forEach((match) => {
                match.liked = false;
              });
              console.log(this.matches);
            });
          }
        })
        .catch((err) => {
          console.error(err);
        });
    },
    fetchPlayer() {
      fetch(`https://api.opendota.com/api/players/${this.playerID}`, {
        method: "GET",
      })
        .then((response) => {
          if (response.ok) {
            response.json().then((data) => {
              this.player = data;
              this.changeInfo(this.player);
              console.log(this.player);
            });
          } else {
            alert("Invalid player ID");
          }
        })
        .catch((err) => {
          console.error(err);
        });
    },
    changeInfo(player) {
      const playerName = document.querySelector("#player-name");
      const playerPfp = document.querySelector("#player-pfp");
      if (player.profile.name != null) {
        playerName.textContent = `Last 15 matches of ${player.profile.name}`;
      } else {
        playerName.textContent = `Last 15 matches of ${player.profile.personaname}`;
      }
      playerPfp.src = player.profile.avatarfull;
      playerName.classList.remove("hide");
      playerPfp.classList.remove("hide");
      document.querySelector(".filter-button").classList.remove("hide");
    },
    isRadiant(slot) {
      return 0 <= slot && slot <= 127;
    },
    convertTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = seconds % 60;
      if (secs < 10) {
        return `${mins}:0${secs}`;
      }
      return `${mins}:${secs}`;
    },
    convertStartTime(unixcode) {
      const time = new Date(unixcode * 1000);
      return time.toLocaleDateString();
    },
    removeSelf(index) {
      this.matches.splice(index, 1);
    },
    switchLike(index) {
      this.matches[index].liked = !this.matches[index].liked;
    },
    filterLiked() {
      this.likeFilter = !this.likeFilter;
    },
  },
  computed: {
    filteredList() {
      if (this.likeFilter == true) {
        return this.matches.filter((item) => item.liked == true);
      } else {
        return this.matches;
      }
    },
  },
};
</script>
<style>
@import "../assets/style.css";
</style>
