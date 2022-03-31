<template>
  <div class="wrapper">
    <div class="left">
      <div class="up">
        <div
          class="green"
          :class="{ greenOpacity: opacityValue.greenOpacity }"
          @click="opacityOnClick('greenOpacity')"
        ></div>
        <div
          class="red"
          :class="{ redOpacity: opacityValue.redOpacity }"
          @click="opacityOnClick('redOpacity')"
        ></div>
      </div>
      <div class="down">
        <div
          class="yellow"
          :class="{ yellowOpacity: opacityValue.yellowOpacity }"
          @click="opacityOnClick('yellowOpacity')"
        ></div>
        <div
          class="blue"
          :class="{ blueOpacity: opacityValue.blueOpacity }"
          @click="opacityOnClick('blueOpacity')"
        ></div>
      </div>
    </div>
    <div class="right">
      <h2 class="lose" v-if="gameOver">Вы проиграли!</h2>
      <h1>Раунд: {{ roundNumber }}</h1>
      <h1>Score: {{ score }}</h1>
      <button v-if="!start" @click="startGame">Start</button>
      <hr />
      <h2>Выберите сложность</h2>
      <div class="selectLevel">
        <div>
          <input
            type="radio"
            id="easy"
            name="level"
            value="easy"
            v-model="currentLevel"
          />
          <label for="easy">Легкий</label>
        </div>
        <div>
          <input
            type="radio"
            id="medium"
            name="level"
            value="medium"
            v-model="currentLevel"
          />
          <label for="medium">Средний</label>
        </div>
        <div>
          <input
            type="radio"
            id="hard"
            name="level"
            value="hard"
            v-model="currentLevel"
          />
          <label for="hard">Сложный</label>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      start: false,
      gameOver: false,
      gameSequence: [],
      playerSequence: [],
      score: 0,
      baseColor: {
        1: "greenOpacity",
        2: "redOpacity",
        3: "yellowOpacity",
        4: "blueOpacity",
      },
      currentColor: "",
      opacityValue: {
        greenOpacity: false,
        redOpacity: false,
        yellowOpacity: false,
        blueOpacity: false,
      },
      levels: {
        easy: 1500, //1.5
        medium: 1000, //1.0
        hard: 400, //0.4
      },
      currentLevel: "easy",
      roundNumber: 1,
      isClickable: false,
    };
  },
  methods: {
    startGame() {
      this.start = true;
      this.gameOver = false;
      this.roundNumber = 1;
      this.startGameBot(this.getLevel);
    },
    startNextRound() {
      this.clearPlayerSequence();
      this.roundNumber++;
      this.startGameBot(this.getLevel);
    },
    clearPlayerSequence() {
      this.playerSequence.length = 0;
    },
    //Выбираем случайный сектор
    getRandomNumber(first, last) {
      const rand = Math.random() * (last - first + 1) + first;
      return Math.floor(rand);
    },
    startGameBot(level) {
      this.gameSequence.push(this.getRandomNumber(1, 4));
      this.gameSequence.forEach((el, index) => {
        setTimeout(() => {
          let color = this.baseColor[el];
          this.opacityValue[color] = true;
          setTimeout(() => {
            this.opacityValue[color] = false;
            this.audioOnClick(color);
          }, level * (index + 1) - 200);
        }, level * (index + 1));
      });
    },
    // Заставляем сектора загораться при нажатии
    opacityOnClick(color) {
      this.opacityValue[color] = true;
      this.currentColor = color;
      this.isClickable = true;
      setTimeout(() => {
        this.opacityValue[color] = false;
        this.audioOnClick(color);
        this.isClickable = false;
      }, 200);
    },
    // Получаем ключ.цвет из объекта
    getKeyByValue(object, value) {
      return Object.keys(object).find((key) => object[key] === value);
    },
    // Проверка последовательности
    checkUserSequence() {
      this.playerSequence.forEach((el, index) => {
        if (el !== this.gameSequence[index]) {
          this.gameOver = true;
          return;
        }
      });
      if (
        this.playerSequence.length === this.gameSequence.length &&
        !this.gameOver
      ) {
        this.score++;
        this.startNextRound();
      }
    },
    // Звуки при нажатии на один из секторов (возможно сделать прелоад)
    audioOnClick(color) {
      let audio;
      switch (color) {
        case "blueOpacity":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"
          );
          audio.play();
          break;
        case "redOpacity":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"
          );
          audio.play();
          break;
        case "yellowOpacity":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"
          );
          audio.play();
          break;
        case "greenOpacity":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"
          );
          audio.play();
          break;
      }
    },
  },
  watch: {
    // Обнуление при переключении сложности
    currentLevel() {
      this.start = false;
      this.score = 0;
      this.gameSequence.length = 0;
      this.playerSequence.length = 0;
    },
    // Следим за кликами пользователя
    isClickable() {
      if (this.isClickable) {
        let value = this.getKeyByValue(this.baseColor, this.currentColor);
        this.playerSequence.push(Number(value));
        this.checkUserSequence();
      }
    },
    // Обнуление при проигрыше
    gameOver() {
      if (this.gameOver) {
        this.start = false;
        this.gameSequence.length = 0;
        this.clearPlayerSequence();
        this.score = 0;
      }
    },
  },
  computed: {
    // Получение уровня сложности
    getLevel() {
      return this.levels[this.currentLevel];
    },
  },
};
</script>

<style scoped>
.wrapper {
  width: 1000px;
  margin: auto;

  display: flex;
  flex-direction: row;
  justify-content: space-around;
}

.up,
.down {
  margin-top: 10px;

  display: flex;
  flex-direction: row;
}

.green,
.red,
.yellow,
.blue {
  width: 200px;
  height: 200px;
  opacity: 0.1;
  margin: 10px;
  cursor: pointer;
  border: 0 solid;
  border-radius: 5px;
}

.green {
  background-color: green;
}
.greenOpacity {
  opacity: 1;
}

.red {
  background-color: red;
}
.redOpacity {
  opacity: 1;
}

.yellow {
  background-color: yellow;
}
.yellowOpacity {
  opacity: 1;
}

.blue {
  background-color: blue;
}
.blueOpacity {
  opacity: 1;
}

.rigth {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.lose {
  color: red;
  font-size: 25pt;
}
.levelSelect {
  display: flex;
}
h2 {
  font-size: 15pt;
}
h1 {
  font-size: 25pt;
}
button {
  font-size: 15pt;
  width: 150px;
  height: 50px;
  cursor: pointer;
}
</style>
