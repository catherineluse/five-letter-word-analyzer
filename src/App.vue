<script>
import words from "./wordlist";
import { ref } from "vue";
import chroma from "chroma-js";

export default {
  name: "App",
  setup() {
    const positions = [0, 1, 2, 3, 4];
    const alphabet = "abcdefghijklmnopqrstuvwxyz";
    let originalWordlist = words.split("\n");
    let remainingWordlist = ref(originalWordlist);
    let guesses = ref(["", "", "", "", ""]);
    let exactPositionExclusions = ref(["", "", "", "", ""]);
    let excludeLetters = ref("");
    let includeLetters = ref("");

    let freq = ref({
      0: {},
      1: {},
      2: {},
      3: {},
      4: {},
    });

    const isValidWord = (word) => {
      for (let i = 0; i < word.length; i++) {
        // If it doesn't have the letters in the positions
        // that we guessed, the word is invalid.
        if (guesses.value[i] !== "" && guesses.value[i] !== word[i]) {
          return false;
        }

        // If the word has any letter in a position where we know
        // the letter cannot be used, the word is invalid.
        if (
          exactPositionExclusions.value[i] !== "" &&
          exactPositionExclusions.value[i] === word[i]
        ) {
          return false;
        }

        // If it has letters we know are not in the word
        // in any position, the word is invalid.
        for (let j = 0; j < excludeLetters.value.length; j++) {
          const excludeLetter = excludeLetters.value[j];

          if (word.includes(excludeLetter)) {
            return false;
          }
        }

        // If it doesn't have letters we know are somewhere in
        // the word, the word is invalid.
        for (let j = 0; j < includeLetters.value.length; j++) {
          const includeLetter = includeLetters.value[j];

          if (!word.includes(includeLetter)) {
            return false;
          }
        }
      }
      return true;
    };

    const updateRemainingWordlist = () => {
      remainingWordlist.value = originalWordlist.filter((word) => {
        return isValidWord(word);
      });
    };

    const updateFreq = () => {
      for (let i = 0; i < positions.length; i++) {
        for (let j = 0; j < alphabet.length; j++) {
          let letter = alphabet[j];
          freq.value[i][letter] = {
            count: 0,
            words: []
          }
        }
      }

      for (let i = 0; i < remainingWordlist.value.length; i++) {
        const word = remainingWordlist.value[i];
        if (!word) {
          continue;
        }

        for (let j = 0; j < word.length; j++) {
          const letter = word[j];
          freq.value[j][letter].words.push(word)
        }
      }

      for (let i = 0; i < positions.length; i++) {
        for (let j = 0; j < alphabet.length; j++) {
          let letter = alphabet[j];
          let words = freq.value[i][letter].words;
          let dedupedWords = [...new Set(words)]
          freq.value[i][letter].words = dedupedWords;
          freq.value[i][letter].count = dedupedWords.length;
        }
      }
      return freq.value;
    };
    updateFreq();
    const getColorGradient = chroma.scale(["white", "darkgray"]);
    return {
      guesses,
      exactPositionExclusions,
      originalWordlist,
      remainingWordlist,
      updateFreq,
      freq,
      alphabet,
      positions,
      isValidWord,
      updateRemainingWordlist,
      includeLetters,
      excludeLetters,
      getColorGradient,
    };
  },
  computed: {
    sortedRemainingWordlist() {
      let copy = [...this.remainingWordlist]
      return copy.sort((first, second) => {
        const commonality1 = this.getLetterCommonality(first);
        const commonality2 = this.getLetterCommonality(second)
        return commonality2 - commonality1;
      })
    },
  },
  methods: {
    updateGuesses(guess, position) {
      this.guesses[position] = guess;
      this.updateRemainingWordlist();
      this.updateFreq();
    },
    updateInclusions() {
      this.updateRemainingWordlist();
      this.updateFreq();
    },
    updateExactPositionExclusions(letter, position) {
      this.exactPositionExclusions[position] = letter;
      this.updateRemainingWordlist();
      this.updateFreq();
    },
    
    getLetterCommonality(word) {
      // count the number of other words
      // that have the same letter at the same position
      let commonality = 0;
      for (let i = 0; i < word.length; i++) {
        if (this.guesses[i] !== '') {
          continue;
        }
        const letter = word[i]
        let wordsWithSameLetterAtSamePosition = this.freq[i][letter].words;
        commonality += wordsWithSameLetterAtSamePosition.length;
      }
      return commonality;
    }
  },
};
</script>

<template>
  <h1>Include Letters</h1>

  <h3>Include in Exact Position</h3>
  <div class="guesses">
    <input v-model="guesses[0]" @change="updateGuesses(guesses[0], 0)" />
    <input v-model="guesses[1]" @change="updateGuesses(guesses[1], 1)" />
    <input v-model="guesses[2]" @change="updateGuesses(guesses[2], 2)" />
    <input v-model="guesses[3]" @change="updateGuesses(guesses[3], 3)" />
    <input v-model="guesses[4]" @change="updateGuesses(guesses[4], 4)" />
  </div>

  <h3>Include in Any Position</h3>
  <div class="guesses">
    <input v-model="includeLetters" @change="updateInclusions" />
  </div>

  <h1>Exclude Letters</h1>
  <h3>Exclude in Exact Positions</h3>
  <div class="guesses">
    <input
      v-model="exactPositionExclusions[0]"
      @change="updateExactPositionExclusions(exactPositionExclusions[0], 0)"
    />
    <input
      v-model="exactPositionExclusions[1]"
      @change="updateExactPositionExclusions(exactPositionExclusions[1], 1)"
    />
    <input
      v-model="exactPositionExclusions[2]"
      @change="updateExactPositionExclusions(exactPositionExclusions[2], 2)"
    />
    <input
      v-model="exactPositionExclusions[3]"
      @change="updateExactPositionExclusions(exactPositionExclusions[3], 3)"
    />
    <input
      v-model="exactPositionExclusions[4]"
      @change="updateExactPositionExclusions(exactPositionExclusions[4], 4)"
    />
  </div>
  <h3>Exclude In Any Position</h3>
  <div class="guesses">
    <input v-model="excludeLetters" @change="updateInclusions" />
  </div>
  <h1>Frequencies of Letter by Position in Remaining Answers</h1>
  
  <table>
    <thead>
      <tr>
        <th>Letter</th>
        <th :key="position" v-for="position in positions">
          {{ position + 1 }}
        </th>
      </tr>
    </thead>
    <tr :key="letter" v-for="letter in alphabet">
      <td>{{ letter }}</td>
      <td
        :key="position"
        v-for="position in positions"
        :style="{
          'background-color': getColorGradient(
            (freq[position][letter].count / remainingWordlist.length).toFixed(2)
          ).toString(),
        }"
      >
        {{
          `${(
            (freq[position][letter].count / remainingWordlist.length) *
            100
          ).toFixed(2)}% (${freq[position][letter].count})`
        }}
      </td>
    </tr>
  </table>
  <h1>Remaining Possible Answers ({{ remainingWordlist.length }})</h1>
  <p>Remaining answers are sorted by a commonality score. To get the commonality score, we start with 0, then we look at each letter in the word. For each letter we increase the count by the number of other possible answers that have the same letter in the same position.</p>
  <ol>
    <li :key="i" v-for="(word, i) in sortedRemainingWordlist">{{ word }} ({{ getLetterCommonality(word) }})</li>
  </ol>
</template>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
body {
 margin: 10%;
}
h1, h3,
.guesses {
  text-align: center;
}

table {
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}
td,
th {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #04aa6d;
  color: white;
}

input {
  padding: 12px 20px;
  margin: 8px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 16px;
  width: 150px;
}
</style>
