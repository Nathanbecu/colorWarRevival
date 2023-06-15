<script lang="ts">

const arrWidth = [3, 4, 5, 6, 7, 8]
const gameWidth = 12;
const gameHeight = 12;
const numberOfTeam = 4;
const gameSpeed = 1;

const correspondenceTeamColor = {
  "Yellow": "#ffd500",
  "Blue": "#0077ff",
  "Red": "#c30000",
  "Green": "#1bab1b",
  "Neutral": "#d5d5d5"
}

const statistics = {
  "Green": {
    "Attack": 50,
    "Defense": 20,
    "Food": 10,
    "Counter": 80,
    "Speed": 70,
  },
  "Red": {
    "Attack": 80,
    "Defense": 50,
    "Food": 20,
    "Counter": 20,
    "Speed": 40,
  },
  "Blue": {
    "Attack": 40,
    "Defense": 80,
    "Food": 20,
    "Counter": 70,
    "Speed": 40,
  },
  "Yellow": {
    "Attack": 60,
    "Defense": 60,
    "Food": 15,
    "Counter": 40,
    "Speed": 50,
  },
}

/* Idées pouvoir :

 - Rouge : Une bombe drop léatoirement sur le board qui prends 3/4 case en rond
 - Yellow : Une ligne traverse le plateau et une colonne et ça prends tout le monde
 - Bleu : la defense augmente à 100% pendant 3 tours
 - Vert : les verts jouent 2 tours sans que les ennemis ne joue et leur stat de contre attaque monte à 95%
*/

function mapIncrementOnObject(object) {
  Object.keys(object).forEach(function (key, index) {
    object[key] += statistics[key].Speed;
  });
}

export default {
  name: 'Playground',
  data() {
    return {
      boxes: [],
      htmlPlayground: "",
      teamVelocity: {},
      teamFood: {},
      pile: [],
      year: 0,
      turnCount: 0,
      waitingThread: [],
    }
  },
  methods: {
    generateProps() {
      for (let i = 1; i < gameWidth + 1; i++) {
        for (let j = 1; j < gameHeight + 1; j++) {
          this.boxes.push({
            "row": i,
            "col": j,
            "team": "Neutral"
          })
        }
      }
    },
    generateGrid() {
      let html = "";
      this.boxes.forEach(function (box) {
        let rowStart = box.col;
        let columnStart = box.row;
        let rowEnd = parseInt(box.col) + 1;
        let columnEnd = parseInt(box.row) + 1;

        html += "<div class='box " + box.team + "' style='grid-area: " + columnStart + " / " + rowStart + " / " + columnEnd + " / " + rowEnd + "'></div>"
      })

      this.htmlPlayground = html;
    },
    replaceElementInTab(boxCircle: (any)[]) {
      boxCircle.forEach((item) => {
        if (item !== undefined) {
          const box = item
          const index = this.boxes.indexOf(this.boxes.find(function (el) {
            return el.col === box.col && el.row === box.row
          }))
          this.boxes[index] = item;
        }
      })
    },
    launchGame() {
      const teams = ["Yellow", "Green", "Red", "Blue"]
      teams.sort(() => 0.5 - Math.random());

      let boxUpLeft = this.findBoxObject(1, 1);
      let boxUpRight = this.findBoxObject(gameWidth, 1);
      let boxDownLeft = this.findBoxObject(1, gameHeight);
      let boxDownRight = this.findBoxObject(gameWidth, gameHeight);

      boxUpLeft.team = teams[0];
      boxUpRight.team = teams[1];
      boxDownLeft.team = teams[2];
      boxDownRight.team = teams[3];

      this.generateGrid()
      this.initializeVelocity(teams);
      this.initializeFood(teams);

      this.play();


      this.specialPower("Yellow")
      setTimeout(() => {
        this.generateGrid()
      }, 1000)
      this.specialPower("Red")
      setTimeout(() => {
        this.generateGrid()
      }, 1000)

    },
    specialPower(team) {
      if (team === "Yellow") {
        const randomLine = this.getRandomItem(arrWidth);
        const randomColumn = this.getRandomItem(arrWidth);

        if (Math.floor(Math.random() * 2) === 1) {
          this.boxes.filter((el) => {
            return el.col === randomColumn
          }).forEach((box) => {
            box.team = "Yellow"
          })
        } else {
          this.boxes.filter((el) => {
            return el.row === randomLine
          }).forEach((box) => {
            box.team = "Yellow"
          })
        }

      } else if (team === "Red") {
        const randomBox = this.getRandomItem(this.boxes.filter((el) => {
          return el.team !== team
        }))

        const boxCircle = [
          randomBox,
          this.boxes.find((el) => {
            return el.col === randomBox.col + 2 && el.row === randomBox.row
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col + 1 && el.row === randomBox.row
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col + 1 && el.row === randomBox.row + 1
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col + 1 && el.row === randomBox.row - 1
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col && el.row === randomBox.row + 2
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col && el.row === randomBox.row + 1
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col && el.row === randomBox.row - 1
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col && el.row === randomBox.row - 2
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col - 1 && el.row === randomBox.row + 1
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col - 1 && el.row === randomBox.row
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col - 1 && el.row === randomBox.row - 1
          }),
          this.boxes.find((el) => {
            return el.col === randomBox.col - 2 && el.row === randomBox.row
          }),
        ]

        this.replaceElementInTab(boxCircle);

        boxCircle.forEach((box) => {
          if (box !== undefined) box.team = "Red"
        })

      }
    },
    getRandomItem(arr) {
      const randomIndex = Math.floor(Math.random() * arr.length);
      return arr[randomIndex];
    },
    findBoxObject(col, row) {
      return this.boxes.find(el => el.col === col && el.row === row)
    },
    initializeVelocity(teams: string[]) {
      teams.forEach((team) => this.teamVelocity[team] = 0)
    },
    initializeFood(teams: string[]) {
      teams.forEach((team) => this.teamVelocity[team] = 100)
    },
    play() {
      //
      // KNOW WHICH TEAM WILL PLAY IN WHICH ORDER
      //

      // We create a temporary array to make sure all team plays during  a turn
      let uniqueTeam = [];
      while (uniqueTeam.length !== numberOfTeam) {
        // Function to add speed to velocity meter
        mapIncrementOnObject(this.teamVelocity)

        // Now we check if some team have 100 velocity or more, and we had a "turn" on the pile
        const filteredVelocity = Object.keys(this.teamVelocity).filter((key, index) => {
          return this.teamVelocity[key] >= 100
        });

        if (filteredVelocity.length > 0) {
          filteredVelocity.sort((a, b) => {
            // If teams have the same velocity, the order is random
            if (this.teamVelocity[b] === this.teamVelocity[a]) {
              return Math.random() - 0.5
            }
            return this.teamVelocity[b] - this.teamVelocity[a]
          }).forEach(team => {
            this.pile.push({"team": team});
            (uniqueTeam.indexOf(team) === -1) ? uniqueTeam.push(team) : false;
            this.teamVelocity[team] -= 100;
          })
        }

        //
        // DO ACTIONS TEAMS NEED TO DO
        //



        this.year += 1;
      }

    }
  }, created() {
    this.generateProps()
    this.generateGrid();
  },

  computed: {
    playgroundStyle() {
      return {
        "grid-template-rows": "repeat(" + gameHeight + ", 1fr)",
        "grid-template-columns": "repeat(" + gameWidth + ", 1fr)"
      }
    }
  }
}
</script>

<template>
  <div id="container">
    <div :style="playgroundStyle" id="playground" v-html="htmlPlayground"></div>
    <div id="commands">
      <div>Turn : {{ turnCount }}</div>
      <div>Year : {{ year }}</div>
      <div @click="launchGame">declare a war</div>
    </div>
  </div>
</template>

<style>
#playground {
  width: 500px;
  height: 500px;
  display: grid;
}

#container {
  display: flex;
  justify-content: space-around;
  align-items: center;
  width: 100%;
}

.box {
  border: black 1px solid;
}

.Neutral {
  background-color: #ededed;
}

.Yellow {
  background-color: #ffd500;
}

.Blue {
  background-color: #0077ff;
}

.Red {
  background-color: #c30000;
}

.Green {
  background-color: #1bab1b;
}
</style>
