<template>
  <div class="table">

    <div class="row row--player">
      <p class="cell cell--label">{{ $t('playerTitle') }}</p>
      <div v-for="player in playerCount" :key="player" class="cell cell--label cell--player-num">{{player}}</div>
    </div>

    <div class="row" v-for="(scoreType, i) in scoreTypes" :key="i">
      <p class="cell cell--label">{{localizedScoreTypes[i]}}</p>
      <div
        class="cell cell--score"
        v-for="(player, playerNum) in activePlayers"
        :key="playerNum"
      >
        <input
          class="score-input"
          type="number"
          min="0"
          :id="scoreId(scoreType, playerNum)"
          :title="label(localizedScoreTypes[i], playerNum)"
          :aria-label="label(localizedScoreTypes[i], playerNum)"
          :value="score(scoreType, playerNum) == -1 ? null : score(scoreType, playerNum)"
          @input="updateScore($event, playerNum, scoreType)"
        />
      </div>
    </div>

    <div class="row row--total">
      <div class="cell cell--label">
        <ToggleButton :name="totalText" v-model="showResults"/>
      </div>
      <div
        v-for="(player, playerNum) in activePlayers"
        :key="playerNum"
        class="cell cell--total"
        >
          <span class="result" v-show="showResults" :title="totalLabel(playerNum)">
            {{player.total}}
          </span>
        </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import ToggleButton from './ToggleButton'

export default {
  name: 'ScoringTable',
  components: {
    ToggleButton
  },
  data () {
    return {
      showResults: false
    }
  },
  computed: {
    ...mapGetters(['activePlayers', 'winner']),
    ...mapState(['playerCount', 'scoreTypes', 'localizedScoreTypes']),
    totalText () {
      return this.$t('total')
    }
  },
  methods: {
    score (scoreType, playerNum) {
      return this.activePlayers[playerNum].scores[scoreType]
    },

    scoreId (scoreType, playerNum) {
      return `player-${playerNum}-score-${scoreType
        .toLowerCase()
        .replace(/\s/, '-')}`
    },

    label (scoreType, playerNum) {
      return this.$t('pointsInputTitle', { playerNum, scoreType })
    },

    totalLabel (playerNum) {
      return this.$t('playerTotalTitle', { playerNum })
    },

    updateScore (event, playerNum, scoreType) {
      const value = event.target.value
      let score = Number.parseInt(value)
      if (value.length === 0) {
        score = 0
      } else if (isNaN(score)) {
        event.target.setCustomValidity('Input must be a number')
        score = 0
      }
      this.$store.commit('setPlayerScore', {
        playerNum,
        scoreType,
        score
      })
    }
  }
}
</script>

<style scoped lang="scss">

$color--border: rgba(0, 0, 0, 0.2);

.table {
  display: grid;
}

.row {
  display: grid;
  grid-auto-flow: column;
  grid-template-columns: minmax(8.5rem, 1fr) 1fr 1fr 1fr 1fr 1fr;
  text-align: center;

  @include break-phone {
    grid-template-columns: minmax(11rem, 1fr) 1fr 1fr 1fr 1fr 1fr;
  }

  + .row {
    border-top: 1px solid $color--border;
  }

  + .row--total {
    border-top-width: 2px;
  }
}

.row--player {
  font-weight: bold;

  + .row {
    border-top-width: 2px;
  }
}

.cell {
  margin: 0;
  padding: 0.25rem;

  @include break-phone {
    padding: 0.5rem
  }

  + .cell {
    border-left: 1px solid $color--border;
  }
}

.cell--label {
  font-family: $font-primary;
  letter-spacing: 0.02rem;
  font-size: 1.2rem;
  text-align: left;

  @include break-phone {
    font-size: 1.5rem;
  }
}

.cell--player-num {
  text-align: center;
}

.cell--total {
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cell--score {
  padding: 0;
}

.score-input {
  width: 100%;
  height:  100%;
  border: 0;
  text-align: center;
  background-color: $color-fg--light;

  &:focus {
  outline: 0;
    box-shadow: inset 0 0 5px 1px rgba(0,0,0,0.1);
  }

  &:invalid {
    box-shadow: inset 0 0 5px 1px rgba(255,0,0,0.6);
    outline: 0;
  }
}

/deep/ .toggle-button {
  font-weight: bold;
}

</style>
