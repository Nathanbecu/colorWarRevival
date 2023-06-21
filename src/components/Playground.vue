<script lang='ts'>

import $ from 'jquery'

const arrWidth = [3, 4, 5, 6, 7, 8]
const gameWidth = 6
const gameHeight = 6
const numberOfTeam = 4
const baseProbabilitieNeutral = 90
const baseProbabilitieTeam = 50
const gameSpeed = 1

const correspondenceTeamColor = {
  'Yellow': '#ffd500',
  'Blue': '#0077ff',
  'Red': '#c30000',
  'Green': '#1bab1b',
  'Neutral': '#d5d5d5'
}

const statistics = {
  'Neutral': {
    'Attack': 0,
    'Defense': 0,
    'Food': 0,
    'Counter': 0,
    'Speed': 0,
    'BP': baseProbabilitieNeutral
  },
  'PowerUp': {
    'Attack': 0,
    'Defense': 0,
    'Food': 0,
    'Counter': 0,
    'Speed': 0,
    'BP': baseProbabilitieNeutral
  },
  'Green': {
    'Attack': 50,
    'Defense': 20,
    'Food': 5,
    'Counter': 80,
    'Speed': 70,
    'BP': baseProbabilitieTeam
  },
  'Red': {
    'Attack': 80,
    'Defense': 50,
    'Food': 10,
    'Counter': 20,
    'Speed': 40,
    'BP': baseProbabilitieTeam
  },
  'Blue': {
    'Attack': 40,
    'Defense': 80,
    'Food': 10,
    'Counter': 70,
    'Speed': 40,
    'BP': baseProbabilitieTeam
  },
  'Yellow': {
    'Attack': 60,
    'Defense': 60,
    'Food': 8,
    'Counter': 40,
    'Speed': 50,
    'BP': baseProbabilitieTeam
  }
}

/* Idées pouvoir :

 - Rouge : Une bombe drop léatoirement sur le board qui prends 3/4 case en rond
 - Yellow : Une ligne traverse le plateau et une colonne et ça prends tout le monde
 - Bleu : la defense augmente à 100% pendant 3 tours
 - Vert : les verts jouent 2 tours sans que les ennemis ne joue et leur stat de contre attaque monte à 95%
*/

function mapIncrementOnObject(object) {
  Object.keys(object).forEach(function(key) {
    object[key] += statistics[key].Speed
  })
}

export default {
  name: 'Playground',
  data() {
    return {
      boxes: [],
      htmlPlayground: '',
      teamVelocity: {},
      teamFood: {},
      pile: [],
      year: 0,
      turnCount: 0,
      waitingThread: [],
      teams: []
    }
  },
  methods: {
    generateProps() {
      let k = 1
      for (let i = 1; i < gameWidth + 1; i++) {
        for (let j = 1; j < gameHeight + 1; j++) {
          this.boxes.push({
            'row': i,
            'col': j,
            'team': 'Neutral',
            'id': k
          })
          k++
        }
      }
    },
    generateGrid() {
      return new Promise(resolve => {
        let html = ''
        this.boxes.forEach(function(box) {
          let rowStart = box.col
          let columnStart = box.row
          let rowEnd = parseInt(box.col) + 1
          let columnEnd = parseInt(box.row) + 1

          html += '<div id=\'box' + box.id + '\' class=\'box ' + box.team + '\' style=\'grid-area: ' + columnStart + ' / ' + rowStart + ' / ' + columnEnd + ' / ' + rowEnd + '\'></div>'
        })

        this.htmlPlayground = html
        resolve()
      })

    },
    replaceElementInTab(boxCircle: (any)[]) {
      boxCircle.forEach((item) => {
        if (item !== undefined) {
          const box = item
          const index = this.boxes.indexOf(this.boxes.find(function(el) {
            return el.col === box.col && el.row === box.row
          }))
          this.boxes[index] = item
        }
      })
    },
    async launchGame() {
      this.teams = ['Yellow', 'Green', 'Red', 'Blue']
      this.teams.sort(() => 0.5 - Math.random())

      let boxUpLeft = this.findBoxObject(1, 1)
      let boxUpRight = this.findBoxObject(gameWidth, 1)
      let boxDownLeft = this.findBoxObject(1, gameHeight)
      let boxDownRight = this.findBoxObject(gameWidth, gameHeight)

      boxUpLeft.team = this.teams[0]
      boxUpRight.team = this.teams[1]
      boxDownLeft.team = this.teams[2]
      boxDownRight.team = this.teams[3]

      let powerUp = this.getRandomItem(this.boxes.filter(el => el.col !== 1 && el.row !== 4 && el.col !== gameWidth && el.row !== gameHeight))
      powerUp.team = 'PowerUp'
      this.replaceElementInTab([powerUp])

      await this.generateGrid()
      this.initializeVelocity(this.teams)
      this.initializeFood(this.teams)

      await this.play()

    },
    specialPower(team) {
      return new Promise(async resolve => {
        if (team === 'Yellow') {
          const randomLine = this.getRandomItem(arrWidth)
          const randomColumn = this.getRandomItem(arrWidth)

          if (Math.floor(Math.random() * 2) === 1) {
            this.boxes.filter((el) => {
              return el.col === randomColumn
            }).forEach((box) => {
              box.team = 'Yellow'
            })
          } else {
            this.boxes.filter((el) => {
              return el.row === randomLine
            }).forEach((box) => {
              box.team = 'Yellow'
            })
          }

        } else if (team === 'Red') {
          const randomBox = this.getRandomItem(this.boxes.filter((el) => {
            return el.team !== team
          }))
          let boxesAround = this.boxesAround(randomBox.col, randomBox.row, 2)
          const boxCircle = [
            randomBox,
            ...boxesAround
          ]

          this.replaceElementInTab(boxCircle)

          boxCircle.forEach((box) => {
            if (box !== undefined) box.team = 'Red'
          })

        } else if (team === 'Green') {
          console.log(JSON.stringify(this.pile))
          this.pile.push({ 'team': 'Green' }, { 'team': 'Green' })
          console.log(JSON.stringify(this.pile))
          let baseAttack = statistics['Green'].Attack
          statistics['Green'].Attack = 90
          await this.playThePile()
          statistics['Green'].Attack = baseAttack
        }

        resolve()
      })

    },
    getRandomItem(arr) {
      const randomIndex = Math.floor(Math.random() * arr.length)
      return arr[randomIndex]
    },
    /** Find the correct box with coordinates, returns an object */
    findBoxObject(col, row): object {
      return this.boxes.find(el => el.col === col && el.row === row)
    },
    /** Find the correct boxes with the color, returns an array of objects */
    findBoxesColor(team): object[] {
      return this.boxes.filter(el => el.team === team)
    },
    initializeVelocity(teams: string[]) {
      teams.forEach((team) => this.teamVelocity[team] = 0)
    },
    initializeFood(teams: string[]) {
      teams.forEach((team) => this.teamFood[team] = 100)
    },
    boxesAround(col, row, radius): (object | undefined)[] {
      let boxesAround = []

      if (radius === 1) {
        boxesAround = [
          this.boxes.find((el) => {
            return el.col === col + 1 && el.row === row
          }),
          this.boxes.find((el) => {
            return el.col === col && el.row === row + 1
          }),
          this.boxes.find((el) => {
            return el.col === col - 1 && el.row === row
          }),
          this.boxes.find((el) => {
            return el.col === col && el.row === row - 1
          })
        ]
      } else if (radius === 2) {
        boxesAround = [
          this.boxes.find((el) => {
            return el.col === col + 2 && el.row === row
          }),
          this.boxes.find((el) => {
            return el.col === col + 1 && el.row === row
          }),
          this.boxes.find((el) => {
            return el.col === col + 1 && el.row === row + 1
          }),
          this.boxes.find((el) => {
            return el.col === col + 1 && el.row === row - 1
          }),
          this.boxes.find((el) => {
            return el.col === col && el.row === row + 2
          }),
          this.boxes.find((el) => {
            return el.col === col && el.row === row + 1
          }),
          this.boxes.find((el) => {
            return el.col === col && el.row === row - 1
          }),
          this.boxes.find((el) => {
            return el.col === col && el.row === row - 2
          }),
          this.boxes.find((el) => {
            return el.col === col - 1 && el.row === row + 1
          }),
          this.boxes.find((el) => {
            return el.col === col - 1 && el.row === row
          }),
          this.boxes.find((el) => {
            return el.col === col - 1 && el.row === row - 1
          }),
          this.boxes.find((el) => {
            return el.col === col - 2 && el.row === row
          })
        ]
      }

      return boxesAround
    },
    boxesAroundWithoutThisColor(col, row, team): object[] {
      return this.boxesAround(col, row, 1).filter(el => el !== undefined && el.team !== team)
    },
    attack(attacker: object, defender: object) {
      return new Promise(resolve => {
        if (defender.team === 'PowerUp') {
          this.pile.push({
            'team': 'PowerUp',
            'caster': attacker.team
          })
        }

        let success = this.boolOnProbabilitie(statistics[defender.team].BP, statistics[attacker.team], statistics[defender.team])
        let foodAttack = statistics[attacker.team].Food
        let htmlDefender = $('#box' + defender.id)
        htmlDefender.css('animation', 'pulsate-' + attacker.team + ' 1.5s infinite alternate')
        htmlDefender.css('z-index', '9999')

        if (success) {
          // ATTACK SUCCESSFUL
          this.teamFood[attacker.team] -= foodAttack
          setTimeout(() => {
            htmlDefender.css('animation', 'none')
            htmlDefender.css('z-index', 'initial')
            htmlDefender.css('background-color', correspondenceTeamColor[attacker.team])
            this.findBoxObject(defender.col, defender.row).team = attacker.team

            resolve()
          }, 1000)
        } else {
          // ATTACK FAILED
          setTimeout(() => {
            this.teamFood[attacker.team] -= Math.floor(foodAttack / 2)
            htmlDefender.css('animation', 'none')
            htmlDefender.css('z-index', 'initial')
            if (this.teamFood[defender.team] < statistics[defender.team].Food) {
              let htmlAttacker = $('#box' + attacker.id)
              htmlAttacker.css('animation', 'pulsate-' + defender.team + ' 1.5s infinite alternate')
              htmlAttacker.css('z-index', '9999')

              if ((statistics[defender.team].Counter / 100) - Math.random() > 0) {
                // COUNTER SUCCESSFUL
                this.teamFood[defender.team] -= Math.floor(statistics[defender.team].Food / 2)
                htmlAttacker.css('background-color', correspondenceTeamColor[defender.team])
                this.findBoxObject(attacker.col, attacker.row).team = defender.team
              } else {
                // COUNTER FAILED
                htmlAttacker.css('animation', 'none')
                htmlAttacker.css('z-index', 'initial')
              }
            }

            resolve()
          }, 1000)
        }


      })
    },
    async playThePile() {
      return new Promise(async resolve => {
        for (const teamObject of this.pile) {
          if (teamObject.team !== 'PowerUp') {
            const boxes = this.findBoxesColor(teamObject.team)

            for (const box of boxes) {
              if (this.teamFood[box.team] >= statistics[box.team].Food) {
                const boxesAround = this.boxesAroundWithoutThisColor(box.col, box.row, teamObject.team)

                if (boxesAround.length > 0) {
                  let randomBox = this.getRandomItem(boxesAround)
                  await this.attack(box, randomBox)
                }
              }
            }
          } else {
            await this.specialPower(teamObject.caster)
          }
        }

        this.pile = []
        resolve()
      })

    },
    async play() {
      //
      // KNOW WHICH TEAM WILL PLAY IN WHICH ORDER
      //

      // We create a temporary array to make sure all team plays during  a turn
      let uniqueTeam = []
      while (uniqueTeam.length !== numberOfTeam) {
        // Function to add speed to velocity meter
        mapIncrementOnObject(this.teamVelocity)

        // Now we check if some team have 100 velocity or more, and we had a "turn" on the pile
        const filteredVelocity = Object.keys(this.teamVelocity).filter((key) => {
          return this.teamVelocity[key] >= 100
        })

        if (filteredVelocity.length > 0) {
          filteredVelocity.sort((a, b) => {
            // If teams have the same velocity, the order is random
            if (this.teamVelocity[b] === this.teamVelocity[a]) {
              return Math.random() - 0.5
            }
            return this.teamVelocity[b] - this.teamVelocity[a]
          }).forEach(team => {
            this.pile.push({ 'team': team });
            (uniqueTeam.indexOf(team) === -1) ? uniqueTeam.push(team) : false
            this.teamVelocity[team] -= 100
          })
        }

        this.year += 1
      }
      //
      // DO ACTIONS TEAMS NEED TO DO
      //

      await this.playThePile()

      await this.generateGrid()
      await this.harvest()
      this.turnCount += 1

      setTimeout(async () => {
        if (this.turnCount < 10) {
          await this.play()
        }
      }, 1000)
    },
    boolOnProbabilitie(baseProbabilitie: number, attacker: object, defender: object) {
      let probabilitie = (baseProbabilitie * (1 + (attacker.Attack / 100))) * (1 - (defender.Defense / 100))
      return (probabilitie / 100) - Math.random() >= 0
    },
    harvest() {
      return new Promise(resolve => {
        this.teams.forEach((team) => {
          this.teamFood[team] += this.findBoxesColor(team).length
        })

        resolve()
      })
    }
  },
  created() {
    this.generateProps()
    this.generateGrid()
  },
  computed: {
    playgroundStyle() {
      return {
        'grid-template-rows': 'repeat(' + gameHeight + ', 1fr)',
        'grid-template-columns': 'repeat(' + gameWidth + ', 1fr)'
      }
    }
  }
}
</script>

<template>
  <div id='container'>
    <div :style='playgroundStyle' id='playground' v-html='htmlPlayground'></div>
    <div id='commands'>
      <div>Turn : {{ turnCount }}</div>
      <div>Year : {{ year }}</div>
      <div @click='launchGame'>declare a war</div>
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

@keyframes pulsate-Yellow {
  100% {
    /* Larger blur radius */
    box-shadow: 0 0 4px #fff,
    0 0 11px #fff,
    0 0 19px #fff,
    0 0 40px #ffd500,
    0 0 80px #ffd500,
    0 0 90px #ffd500,
    0 0 100px #ffd500,
    0 0 150px #ffd500;
  }
  0% {
    /* Smaller blur radius */
    box-shadow: 0 0 2px #fff,
    0 0 4px #fff,
    0 0 6px #fff,
    0 0 10px #ffd500,
    0 0 45px #ffd500,
    0 0 55px #ffd500,
    0 0 70px #ffd500,
    0 0 80px #ffd500;
  }
}

@keyframes pulsate-Red {
  100% {
    /* Larger blur radius */
    box-shadow: 0 0 4px #fff,
    0 0 11px #fff,
    0 0 19px #fff,
    0 0 40px #c30000,
    0 0 80px #c30000,
    0 0 90px #c30000,
    0 0 100px #c30000,
    0 0 150px #c30000;
  }
  0% {
    /* Smaller blur radius */
    box-shadow: 0 0 2px #fff,
    0 0 4px #fff,
    0 0 6px #fff,
    0 0 10px #c30000,
    0 0 45px #c30000,
    0 0 55px #c30000,
    0 0 70px #c30000,
    0 0 80px #c30000;
  }
}

@keyframes pulsate-Blue {
  100% {
    /* Larger blur radius */
    box-shadow: 0 0 4px #fff,
    0 0 11px #fff,
    0 0 19px #fff,
    0 0 40px #0077ff,
    0 0 80px #0077ff,
    0 0 90px #0077ff,
    0 0 100px #0077ff,
    0 0 150px #0077ff;
  }
  0% {
    /* Smaller blur radius */
    box-shadow: 0 0 2px #fff,
    0 0 4px #fff,
    0 0 6px #fff,
    0 0 10px #0077ff,
    0 0 45px #0077ff,
    0 0 55px #0077ff,
    0 0 70px #0077ff,
    0 0 80px #0077ff;
  }
}

@keyframes pulsate-Green {
  100% {
    /* Larger blur radius */
    box-shadow: 0 0 4px #fff,
    0 0 11px #fff,
    0 0 19px #fff,
    0 0 40px #1bab1b,
    0 0 80px #1bab1b,
    0 0 90px #1bab1b,
    0 0 100px #1bab1b,
    0 0 150px #1bab1b;
  }
  0% {
    /* Smaller blur radius */
    box-shadow: 0 0 2px #fff,
    0 0 4px #fff,
    0 0 6px #fff,
    0 0 10px #1bab1b,
    0 0 45px #1bab1b,
    0 0 55px #1bab1b,
    0 0 70px #1bab1b,
    0 0 80px #1bab1b;
  }
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

.PowerUp {
  border: 5px solid transparent;
  border-image: linear-gradient(to bottom right, #b827fc 0%, #2c90fc 25%, #b8fd33 50%, #fec837 75%, #fd1892 100%);
  border-image-slice: 1;
  background-color: #ededed;
}
</style>
