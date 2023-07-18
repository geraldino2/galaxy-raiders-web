<template>
  <div id="canvas">
    <div id="deep-space" />
    <div id="menu">
      <button class="menu-button" @click='setCookie("gameStarted", "true");showSpaceField=true;'>Start</button>
      <button class="menu-button" @click='setCookie("gameStarted", "exit");exit();'>Exit</button>
      <button class="menu-button" @click='alert("todo");'>Update leaderboard</button>
    </div>
    <div id="leaderboard">
      <!--await $get("/leaderboard");-->
    </div>
    <div id="space-field" v-show="showSpaceField" >
      <SpaceObject id="spaceship" class="spaceship" :data="spaceField.ship" resolution="2" />

      <SpaceObject class="asteroid" :data="asteroid" resolution="2"
                  :key="asteroid.center"
                  v-for="asteroid in spaceField.asteroids" />

      <SpaceObject class="missile" :data="missile" resolution="2"
                  :key="missile.center"
                  v-for="missile in spaceField.missiles" />

      <SpaceObject class="explosion" :data="explosion" resolution="2"
                  :key="explosion.center"
                  v-for="explosion in spaceField.explosions" />
    </div>
  </div>
</template>

<script setup>
function getCookie(name) {
  var cookie = document.cookie;
  var prefix = name + "=";
  var begin = cookie.indexOf("; " + prefix);
  if (begin == -1) {
    begin = cookie.indexOf(prefix);
    if (begin != 0) return null;
  } else {
    begin += 2;
    var end = document.cookie.indexOf(";", begin);
    if (end == -1) end = cookie.length;
  }
  return unescape(cookie.substring(begin + prefix.length, end));
}

function setCookie(name, value) {
  let date = new Date();
  date.setTime(date.getTime() + 2592000);
  let expires = "; expires=" + date.toUTCString();
  document.cookie = name + "=" + (value || "")  + expires + "; path=/";
}

function exit() {
  $get("../exit");
}

const {
  data: spaceField,
  refresh: updateSpaceField
} = await $get("/space-field");

const showSpaceField = ref(false)

let gameStarted = false
let pauseOnStart = false

onMounted(() => {
  gameStarted = getCookie("gameStarted")
  if (gameStarted == undefined || gameStarted == "exit") {
    setCookie("gameStarted", "false", "2592000")
    pauseOnStart = true
    gameStarted = false
  } else if (gameStarted == "true") gameStarted = true; else gameStarted = false

  if (gameStarted == false && pauseOnStart) {
    $post("/ship/commands", '{"command":"PAUSE_GAME"}');
  }

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

    // Ignore if the game hasn't started yet
    if (getCookie("gameStarted") != "true") return;

    console.log(`Triggering command: ${command}`);
    await $post("/ship/commands", { command })
  });

  window.setInterval(updateSpaceField, 1000);
})
</script>

<style>
#menu {
  position: relative;
}

.menu-button {
  cursor: pointer;
  border-radius: 100rem;
  font-size: 1rem;
  padding: .5rem 3rem;
  color: $color-black;
  border: solid 3px transparent;
  background-image: linear-gradient(rgba(255, 255, 255, 0), rgba(255, 255, 255, 0)), linear-gradient(101deg, #075569, #0f323b);
  background-origin: border-box;
  background-clip: content-box, border-box;
  box-shadow: 2px 1000px 1px #000 inset;
  color: white;
}

.menu-button:hover {
  box-shadow: none;
  color: black;
}

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
</style>
