<template>
  <div id="canvas">
    <div id="deep-space" />
    <div id="space-field">
      <div id="menu" v-show="openMenu">
        <button class="button" v-on:click="startGame">Start</button>
        <button class="button" v-on:click="leaderboard">Leaderboard</button>
        <button class="button" v-on:click="exitGame">Exit</button>
			</div>

      <div v-show="showLeaderboard">
        <div class="leaderboard">
          <h1>Leaderboard</h1>
          <table>
            <thead>
              <tr>
                <!-- <th>Rank</th> -->
                <th>Score</th>
                <th>Asteroids</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(entry, index) in leaderboardJson" :key="entry.player">
                <!-- <td>{{ index }}</td> -->
                <td>{{ entry.score }}</td>
                <td>{{ entry.destroyedAsteroids }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <button class="button" v-on:click="openMenu = true; showLeaderboard = false">Back</button>
      </div>

			<SpaceObject id="spaceship" class="spaceship" :data="spaceField.ship" resolution="2" v-if="gameStarted" />

      <SpaceObject class="asteroid" :data="asteroid" resolution="2"
                  :key="asteroid.center"
                  v-for="asteroid in spaceField.asteroids" v-if="gameStarted"/>

      <SpaceObject class="missile" :data="missile" resolution="2"
                  :key="missile.center"
                  v-for="missile in spaceField.missiles" v-if="gameStarted"/>

      <SpaceObject class="explosion" :data="explosion" resolution="2"
                  :key="explosion.center"
                  v-for="explosion in spaceField.explosions" v-if="gameStarted"/>
    </div>
  </div>
</template>

<script setup>
const {
  data: spaceField,
  refresh: updateSpaceField
} = await $get("/space-field");

onMounted(() => {
  window.addEventListener("keydown", async (event) => {
    const keyToCommand = {
      "ArrowUp": "MOVE_SHIP_UP",
      "ArrowDown": "MOVE_SHIP_DOWN",
      "ArrowRight": "MOVE_SHIP_RIGHT",
      "ArrowLeft": "MOVE_SHIP_LEFT",
      "Space": "LAUNCH_MISSILE",
      "Escape": "PAUSE_GAME",
    };

    const command = keyToCommand[event.code];

    // Ignore if invalid key was pressed
    if (command === undefined) return;

    console.log(`Triggering command: ${command}`);
    await $post("/ship/commands", { command })
  });

  window.setInterval(updateSpaceField, 16);
})
</script>

<script>
import leaderboardJson from "~/mocks/score/leaderboard.json";

export default {
  data() {
    return {
      openMenu: true,
      gameStarted: false,
      showLeaderboard: false,
    };
  },

  methods: {
    startGame() {
      this.openMenu = false;
      this.gameStarted = true;
    },

    leaderboard() {
      this.showLeaderboard = true;
      this.openMenu = false;
    },

    exitGame() {
      this.openMenu = false;
    },
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=VT323&display=swap');

#canvas {
  height: calc(100vh - 4rem);
  width: calc(100vw - 4rem);

  padding: 2rem;

  background-color: #36bbf5;
  overflow: hidden;

  position: relative;

  display: flex;
  justify-content: center;
  align-items: center;
}

@keyframes slide {
  0% {
    transform: translate(1px);
  }
  50% {
    transform: translate(-1px);
  }
  100% {
    transform: translate(1px);
  }
}

#deep-space {
  height: calc(100% - 4rem);
  width: calc(100% - 4rem);

  background-image: url("~/assets/space.png");
  background-origin: content-box;
  animation: slide 3s linear infinite;

  position: absolute;
  z-index: 0;
}

#space-field {
  height: calc(100% - 4rem);
  width: calc(100% - 4rem);

  position: relative;
}

.spaceship {
  background-image: url("~/assets/spaceship.png");
}

.asteroid {
  background-image: url("~/assets/asteroid.png");
}

.missile {
  background-image: url("~/assets/missile.png");
}

.explosion {
  background-image: url("~/assets/explosion.png");
}

#menu {
	display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  opacity: 78%;
}

.button {
  background-color: transparent;
  border: 5px #36bbf5 solid;
  cursor: pointer;
  margin: 2rem;
  padding: 0.5rem 2rem;
  font-family: "VT323";
  font-size: 6rem;
  font-weight: bold;
  color: #36bbf5;
  border-radius: 0.5rem;
  transition: transform 0.2s ease-in-out;
}

.button:hover {
	cursor: pointer;
	transform: scale(1.2);
}

.leaderboard {
  font-family: "VT323";
  text-align: center;
  align-items: center;
  background-color: #36bbf5;
  border: 5px #36bbf5 solid;
  border-radius: 0.5rem;
  padding: 2rem;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.leaderboard h1 {
  font-size: 10rem;
  font-weight: bold;
  color: #000;
}

.leaderboard table {
  border-collapse: collapse;
  width: 100%;
}

.leaderboard th {
  font-size: 4rem;
  font-weight: bold;
  color: #000;
}

.leaderboard td {
  font-size: 3rem;
  font-weight: bold;
  color: #00000091;
}

</style>
