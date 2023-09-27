<template>
    <div class="personality-quiz">
      <div v-if="question?.personalityQuiz?.questions && currentPersonalityQuestionIndex < question.personalityQuiz.questions.length">
        <h3>{{ question.personalityQuiz.questions[currentPersonalityQuestionIndex].text }}</h3>
        <ul aria-live="polite">
          <li
            v-for="(option, index) in question.personalityQuiz.questions[currentPersonalityQuestionIndex].options"
            :key="index"
          >
            <button class="option" @click="submitPersonalityAnswer(question.personalityQuiz.categories[index].name)">
              {{ option.text }}
            </button>
          </li>
        </ul>
      </div>
      <div v-else>
        <h4>Your result</h4>
        <p>{{ computedPersonalityResultText }}</p>
        <button @click="resetPersonalityQuiz">Restart Quiz</button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      question: Object,
      index: Number,
    },
    data() {
      return {
        currentPersonalityQuestionIndex: 0,
        personalityCategories: {},
        personalityResultText: "",
      };
    },
    computed: {
      computedPersonalityResultText() {
        return this.personalityResultText;
      },
    },
    methods: {
      async submitPersonalityAnswer(category) {
        if (!this.personalityCategories[category]) {
          this.personalityCategories[category] = 0;
        }
        this.personalityCategories[category]++;
        this.currentPersonalityQuestionIndex++;
  
        if (
          this.currentPersonalityQuestionIndex >=
          this.question.personalityQuiz.questions.length
        ) {
          await this.$nextTick();
          this.calculatePersonalityResult();
        }
      },
      resetPersonalityQuiz() {
        this.currentPersonalityQuestionIndex = 0;
        this.personalityCategories = {};
      },
      calculatePersonalityResult() {
  const maxScore = Math.max(...Object.values(this.personalityCategories));

  const tiedCategories = this.question.personalityQuiz.categories.filter(
    (category) => this.personalityCategories[category.name] === maxScore
  );

  this.personalityResultText = tiedCategories
    .map((category) => category.feedback)
    .join(', ');
},
    },
  };
  </script>
  