<script lang="ts">
const squareWidth = 10;
const gameSpeed = 1;
const correspondenceTeamColor = {
  "Yellow" : "#ffd500",
  "Blue" : "#0077ff",
  "Red" : "#c30000",
  "Green" : "#1bab1b",
  "Neutral" : "#d5d5d5"
}

const statistics = {
  "Green" : {
    "Attack" : 50,
    "Defense" : 20,
    "Food" : 10,
    "Counter" : 90
  },
  "Red" : {
    "Attack" : 80,
    "Defense" : 50,
    "Food" : 20,
    "Counter" : 20
  },
  "Blue" : {
    "Attack" : 40,
    "Defense" : 80,
    "Food" : 20,
    "Counter" : 70
  },
  "Yellow" : {
    "Attack" : 60,
    "Defense" : 60,
    "Food" : 15,
    "Counter" : 40
  },
}


export default {
  name: 'Playground',
  data() {
    return {
      boxes: [],
      htmlPlayground: "",
    }
  },
  methods: {
    generateProps() {
      for (let i = 1; i < 11; i++) {
        for (let j = 1; j < 11; j++) {
          this.boxes.push({
            "row": i,
            "col": j,
            "team": "Neutral"
          })
        }
      }
    },
    generateGrid() {
      console.log(this.boxes[0].team)
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
    launchGame() {
      let boxUpLeft = this.findBoxObject(1, 1);
      let boxUpRight = this.findBoxObject(10, 1);
      let boxDownLeft = this.findBoxObject(1, 10);
      let boxDownRight = this.findBoxObject(10, 10);

      boxUpLeft.team = "Yellow";
      boxUpRight.team = "Green";
      boxDownLeft.team = "Red";
      boxDownRight.team = "Blue";

      this.generateGrid()
    },
    findBoxObject(col, row) {
      return this.boxes.find(el => el.col === col && el.row === row)
    },
  }, created() {
    this.generateProps()
    this.generateGrid();
  },

  computed: {
    playgroundStyle() {
      return {
        "grid-template-rows": "repeat(" + squareWidth + ", 1fr)",
        "grid-template-columns": "repeat(" + squareWidth + ", 1fr)"
      }
    }
  }
}
</script>

<template>
  <div id="container">
    <div :style="playgroundStyle" id="playground" v-html="htmlPlayground"></div>
    <div id="commands">
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
