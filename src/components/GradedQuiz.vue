<template>
	<div class="graded-quiz">
		<div
			v-if="
				question?.gradedQuiz?.questions &&
				currentQuestionIndex < question.gradedQuiz.questions.length
			"
		>
			<h3>
				{{ currentQuestionIndex + 1 }}.
				{{ question.gradedQuiz.questions[currentQuestionIndex].question }}
			</h3>
			<ul aria-live="polite">
				<li
					v-for="(option, index) in question.gradedQuiz.questions[
						currentQuestionIndex
					].options"
					:key="index"
				>
					<button class="option" @click="submitAnswer(option.score)">
						{{ option.text }}
					</button>
				</li>
			</ul>
		</div>
		<div v-else>
			<!-- <h4>Your score: {{ score }}</h4> -->
			<h4>Feedback</h4>
			<p>{{ feedbackText }}</p>
			<button @click="resetQuiz">Restart Quiz</button>
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
			currentQuestionIndex: 0,
			score: 0,
			feedbackText: "",
		};
	},
	methods: {
		submitAnswer(optionScore) {
			this.score += optionScore;
			this.currentQuestionIndex++;
			if (
				this.currentQuestionIndex >= this.question.gradedQuiz.questions.length
			) {
				this.calculateFeedback();
			}
		},
		calculateFeedback() {
			for (const feedback of this.question.gradedQuiz.feedbacks) {
				if (
					this.score >= feedback.minScore &&
					this.score <= feedback.maxScore
				) {
					this.feedbackText = feedback.text;
					break;
				}
			}
		},

		resetQuiz() {
			this.currentQuestionIndex = 0;
			this.score = 0;
			this.feedbackText = "";
		},
	},
};
</script>
