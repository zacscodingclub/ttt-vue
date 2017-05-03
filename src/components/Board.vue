<template lang="html">
<div>
  <div class="gameStatus" :class="gameStatusColor">
    {{ gameStatusMessage }}
  </div>
  <table class="grid">
    <tr>
      <cell name="1"></cell>
      <cell name="2"></cell>
      <cell name="3"></cell>
    </tr>
    <tr>
      <cell name="4"></cell>
      <cell name="5"></cell>
      <cell name="6"></cell>
    </tr>
    <tr>
      <cell name="7"></cell>
      <cell name="8"></cell>
      <cell name="9"></cell>
    </tr>
  </table>
</div>
</template>

<script>
import Cell from './Cell.vue';

export default {
  components: { Cell },

  created () {
    Event.$on('strike', cellNumber => {
      this.cells[cellNumber] = this.activePlayer;
      this.moves++;

      this.gameStatus = this.changeGameStatus();

      this.gameStatusMessage = `${this.activePlayer}'s turn`;
    });

    Event.$on('gridReset', () => {
      debugger;
      Object.assign(this.$data, this.$options.data());
    })
  },

  data () {
    return {
      activePlayer: 'X',
      gameStatus: 'turn',
      gameStatusMessage: `X's turn`,
      gameStatusColor: 'statusTurn',
      moves: 0,
      cells: {
        1: '', 2: '', 3: '',
        4: '', 5: '', 6: '',
        7: '', 8: '', 9: '',
      },
      winCombinations: [
        [1,2,3], [4,5,6], [7,8,9],
        [1,4,7], [2,5,8], [3,6,9],
        [1,5,9], [3,5,7]
      ]
    }
  },

  methods: {
    checkForWin () {
      let cells = this.cells;
      for(let i = 0; i < this.winCombinations.length; i++) {
        let combo = this.winCombinations[i];
        let isWinner = cells[combo[0]] === cells[combo[1]] && cells[combo[1]] === cells[combo[2]] && cells[combo[0]] !== '';

        if (isWinner) {
          return true;
        }
      }
    },

    changeGameStatus () {
      if (this.checkForWin()) {
        return this.gameIsWon();
      }
      else if (this.moves === 9) {
        return 'draw';
      }

      this.changePlayer();
      return 'turn';
    },

    changePlayer () {
      this.activePlayer = this.nonActivePlayer();
    },

    gameIsWon() {
      let winner = this.activePlayer;
      Event.$emit('win', winner);
      this.gameStatusMessage = `${winner} Wins!`;
      Event.$emit('freeze');

      return 'win';
    },

    nonActivePlayer () {
      if (this.activePlayer === 'X') { return 'O' };
      return 'X';
    }
  },

  watch: {
    gameStatus () {
      if (this.gameIsWon() === 'win') {
        this.gameStatusColor = 'statusWin';
        return;
      }
      else if (this.gameStatus === 'draw') {
        this.gameStatusColor = 'statusDraw';
        this.gameStatusMessage = 'Draw!';

        return;
      }
    }
  }
}
</script>

<style lang="scss">
.grid {
  background-color: #34495e;
  color: #fff;
  width: 100%;
  border-collapse: collapse;
}

.gameStatus {
  margin: 0px;
  padding: 15px;
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
  background-color: #f1c40f;
  color: #fff;
  font-size: 1.4em;
  font-weight: bold;
}

.statusTurn {
    background-color: #f1c40f;
}

.statusWin {
    background-color: #2ecc71;
}

.statusDraw {
    background-color: #9b59b6;
}
</style>
