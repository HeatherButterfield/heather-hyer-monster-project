<template>
  <v-layout justify-center>
    <v-container id="app">
      <v-row>
          <div class="small-6 columns">
              <h1 class="text-center">YOU</h1>
              <div class="healthbar">
                  <div
                          class="healthbar text-center"
                          style="background-color: green; margin: 0; color: white;"
                          :style="{width: playerHealth + '%'}">
                      {{ playerHealth }}
                  </div>
              </div>
          </div>
          <div class="small-6 columns">
              <h1 class="text-center">MONSTER</h1>
              <div class="healthbar">
                  <div
                          class="healthbar text-center"
                          style="background-color: green; margin: 0; color: white;"
                          :style="{width: monsterHealth + '%'}">
                      {{ monsterHealth }}
                  </div>
              </div>
          </div>
      </v-row>
      <v-row class="controls" v-if="!gameIsRunning">
          <div class="small-12 columns">
              <v-btn id="start-game" @click="startGame">START NEW GAME</v-btn>
          </div>
      </v-row>
      <v-row class="controls" v-else>
          <div class="small-12 columns">
              <v-btn id="attack" @click="attack">ATTACK</v-btn>
              <v-btn id="special-attack" @click="specialAttack">SPECIAL ATTACK</v-btn>
              <v-btn id="heal" @click="heal">HEAL</v-btn>
              <v-btn id="give-up" @click="areYouSureDialog = true">GIVE UP</v-btn>
          </div>
      </v-row>
      <v-row class="row log" v-if="turns.length > 0">
          <div class="small-12 columns">
              <ul>
                  <v-chip v-for="turn in turns"
                      :class="{'player-turn': turn.isPlayer, 'monster-turn': !turn.isPlayer}" :key="turn">
                      {{ turn.text }}
                  </v-chip>
              </ul>
          </div>
      </v-row>
      <v-dialog
        v-model="winDialog"
        max-width="290"
        >
        <v-card>
          <v-card-title class="headline">You won! Play again?</v-card-title>

          <v-card-actions>
            <div class="flex-grow-1"></div>

            <v-btn
              color="green darken-1"
              text
              @click="dontStartNewGame"
            >
              Disagree
            </v-btn>

            <v-btn
              color="green darken-1"
              text
              @click="startNewGame"
            >
              Agree
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog
        v-model="loseDialog"
        max-width="290"
        >
        <v-card>
          <v-card-title class="headline">You lost. :( Play again?</v-card-title>

          <v-card-actions>
            <div class="flex-grow-1"></div>

            <v-btn
              color="green darken-1"
              text
              @click="dontStartNewGame"
            >
              Disagree
            </v-btn>

            <v-btn
              color="green darken-1"
              text
              @click="startNewGame"
            >
              Agree
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog
        v-model="areYouSureDialog"
        max-width="290"
        >
        <v-card>
          <v-card-title class="headline">Are you sure you want to quit?</v-card-title>

          <v-card-actions>
            <div class="flex-grow-1"></div>

            <v-btn
              color="green darken-1"
              text
              @click="areYouSureDialog = false"
            >
              Disagree
            </v-btn>

            <v-btn
              color="green darken-1"
              text
              @click="giveUp"
            >
              Agree
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </v-layout>
</template>

<script>
export default {
  name: 'app',
  data: function() {
      return {
        playerHealth: 100,
        monsterHealth: 100,
        gameIsRunning: false,
        turns: [],
        winDialog: false,
        loseDialog: false,
        areYouSureDialog: false,
        startNew: false
      }
    },
    methods: {
        startGame: function () {
            this.gameIsRunning = true;
            this.playerHealth = 100;
            this.monsterHealth = 100;
            this.turns = [];
        },
        attack: function () {
            var damage = this.calculateDamage(3, 10);
            this.monsterHealth -= damage;
            this.turns.unshift({
                isPlayer: true,
                text: 'Player hits Monster for ' + damage
            });
            if (this.checkWin()) {
                return;
            }

            this.monsterAttacks();
        },
        specialAttack: function () {
            var damage = this.calculateDamage(10, 20);
            this.monsterHealth -= damage;
            this.turns.unshift({
                isPlayer: true,
                text: 'Player hits Monster hard for ' + damage
            });
            if (this.checkWin()) {
                return;
            }
            this.monsterAttacks();
        },
        heal: function () {
            if (this.playerHealth <= 90) {
                this.playerHealth += 10;
            } else {
                this.playerHealth = 100;
            }
            this.turns.unshift({
                isPlayer: true,
                text: 'Player heals for 10'
            });
            this.monsterAttacks();
        },
        giveUp: function () {
            this.gameIsRunning = false;
            this.areYouSureDialog = false;
        },
        monsterAttacks: function() {
            var damage = this.calculateDamage(5, 12);
            this.playerHealth -= damage;
            this.checkWin();
            this.turns.unshift({
                isPlayer: false,
                text: 'Monster hits Player for ' + damage
            });
        },
        calculateDamage: function(min, max) {
            return Math.max(Math.floor(Math.random() * max) + 1, min);
        },
        checkWin: function() {
            if (this.monsterHealth <= 0) {
                this.winDialog = true;
                return true;
            } else if (this.playerHealth <= 0) {
                this.loseDialog = true;
                return true;
            }
            return false;
        },
        startNewGame: function() {
          this.winDialog = false;
          this.loseDialog = false;
          this.startGame();
        },
        dontStartNewGame: function() {
          this.winDialog = false;
          this.loseDialog = false;
          this.gameIsRunning = false;
        }
    }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
}
.text-center {
    text-align: center;
}

.healthbar {
    width: 80%;
    height: 40px;
    background-color: #eee;
    margin: auto;
    transition: width 500ms;
}

.controls, .log {
    margin-top: 30px;
    text-align: center;
    padding: 10px;
    border: 1px solid #ccc;
    box-shadow: 0px 3px 6px #ccc;
}

.turn {
    margin-top: 20px;
    margin-bottom: 20px;
    font-weight: bold;
    font-size: 22px;
}

.log ul {
    list-style: none;
    font-weight: bold;
    text-transform: uppercase;
}

.log ul li {
    margin: 5px;
}

.log ul .player-turn {
    color: #6d87cb;
}

.log ul .monster-turn {
    color: #95173c;
}

button {
    font-size: 20px;
    background-color: #eee;
    padding: 12px;
    box-shadow: 0 1px 1px black;
    margin: 10px;
}

#start-game {
    background-color: #aaffb0;
}

#start-game:hover {
    background-color: #76ff7e;
}

#attack {
    background-color: #ff7367;
}

#attack:hover {
    background-color: #ff3f43;
}

#special-attack {
    background-color: #ffaf4f;
}

#special-attack:hover {
    background-color: #ff9a2b;
}

#heal {
    background-color: #aaffb0;
}

#heal:hover {
    background-color: #76ff7e;
}

#give-up {
    background-color: #ffffff;
}

#give-up:hover {
    background-color: #c7c7c7;
}
</style>
