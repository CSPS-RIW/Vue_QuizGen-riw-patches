<template>
  <div class="highlight-correct-sentence">
    <p>
      <span v-for="(sentence, index) in sentences" :key="index" style="display: inline">
        <span tabindex="0" role="button" :aria-label="getAriaLabel(index)" @click="selectSentence(index)"
          @keydown.enter.space="selectSentence(index)" :class="{ highlighted: isHighlighted(index) }">{{ sentence.text
          }}</span>
        <span>&nbsp; </span>
      </span>
    </p>
    <button class="verify-button" @click="verifySelection">
      {{ $t("verifyButton") }}
    </button>
    <div v-if="feedback" aria-live="polite">
      <p class="feedback">{{ feedback }}</p>
      <p v-html="question.generic_feedback"></p>
    </div>
  </div>
</template>

<script>
export default {
  name: "HighlightCorrectSentence",
  emits: ["userAnswered"],
  props: ["question", "previousAnswer", "preventChange"],
  data() {
    return {
      selectedIndices: [],
      feedback: null,
      isCorrect: false,
    };
  },

  computed: {
    correctFeedback() {
      return this.question.correct_feedback;
    },
    incorrectFeedback() {
      return this.question.incorrect_feedback;
    },
    correctIndices() {
      return this.question.correct_answer
    },
    text() {
      return this.question.question_text;
    },
    sentences() {
      return this.text
        .split(/(?<=[.!?])\s+/) // Split the text using a regex that matches spaces following a period, exclamation point, or question mark
        .filter((sentence) => sentence.trim())
        .map((sentence, index, array) => ({
          text: sentence.trim() + (index < array.length - 1 ? "" : ""),
          index,
        }));
    },
  },

  methods: {
    getAriaLabel(index) {
      const baseLabel = this.$t("sentence", {
        index: index + 1,
        text: this.sentences[index].text,
      });
      const isSelected = this.isHighlighted(index);
      const selectionStatus = isSelected ? this.$t("selected") : "";
      return `${baseLabel} ${selectionStatus}`;
    },
    selectSentence(index) {
      this.feedback = null;
      if (this.selectedIndices.includes(index)) {
        this.selectedIndices = this.selectedIndices.filter(
          (selectedIndex) => selectedIndex !== index
        );
      } else {
        this.selectedIndices.push(index);
      }
    },
    isHighlighted(index) {
      return this.selectedIndices.includes(index);
    },
    verifySelection() {
      if (this.selectedIndices.length === 0) {
        alert(this.$t("noSelectionFeedback"));
        return;
      }
      if (
        this.selectedIndices.length === this.correctIndices.length &&
        this.selectedIndices.every((index) =>
          this.correctIndices.includes(index)
        )
      ) {
        this.feedback = this.correctFeedback;
        this.isCorrect = true;
      } else {
        this.feedback = this.incorrectFeedback;
        this.isCorrect = false;
      }
      this.$emit("userAnswered", {
        correct: this.isCorrect,
        answer: this.selectedIndices,
      });
    },
  },
};
</script>

<style scoped>
.highlight-correct-sentence span {
  cursor: pointer;
}

.highlighted {
  background-color: rgb(255, 255, 185);
  outline: 1px dashed black;
  outline-offset: 0.15rem;
}

.highlighted:focus {
  outline: 3px dashed black;
}

.feedback {
  font-weight: bold;
}

.verify-button {
  background-color: #559;
  border: 1px solid black;
  color: white;
  padding: 12px 24px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
  transition-duration: 0.4s;
}

.verify-button:hover {
  background-color: #33b;
  box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
</style>
