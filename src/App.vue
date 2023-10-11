<template>
	<div id="quiz">
		<div v-if="isQuizDataLoaded">
			<div v-if="quizData.allowLanguageSwitching" class="language-toggle">
				<button class="btn btn-secondary" @click="toggleLanguage">{{ buttonText }}</button>
			</div>
			<div v-if="quizData.quiz_title">
				<h1>{{ quizData.quiz_title }}</h1>
			</div>

			<div v-if="!quizFinished">
				<div class="page-info sr-only" aria-live="polite">
					{{ currentPageInfo }}
				</div>
				<Question v-if="!quizFinished" :data="quizData.questions[currentQuestionIndex]"
					:index="currentQuestionIndex" :lastIndex="quizData.questions.length - 1"
					:preventChangingAnswers="quizData.prevent_changing_answers"
					:savedAnswer="savedAnswer[currentQuestionIndex]" :displayIndividualOptionFeedback="quizData.display_individual_option_feedback
						" @next="nextQuestion" @previous="previousQuestion" @update-results="updateResults" @submit="submitQuiz"
					@save-answers="storeCurrentAnswer" />
				<div class="page-info">{{ currentPageInfo }}</div>
				<div v-if="quizData.end_quiz_button" class="quizComplete">
					<button class="btn btn-secondary" v-if="allQuestionsSubmitted" @click="
						quizData.calculate_quiz_score ? showQuizFeedback() : finishQuiz()
						">
						{{
							quizData.calculate_quiz_score
							? $t('button.getFeedback')
							: $t('button.completeQuiz')
						}}
					</button>
				</div>
			</div>
			<div v-if="quizData.calculate_quiz_score && quizFinished" class="quiz-score">
				<h2>{{ $t('quiz.results') }}</h2>
				<p>
					{{
						$t('quiz.score', [
							score,
							quizData.questions.length,
							percentage,
						])
					}}
				</p>
				<p v-if="passed">{{ $t('quiz.passed') }}</p>
				<p v-else>{{ $t('quiz.failed') }}</p>
			</div>
			<div v-if="quizFinished && !quizData.calculate_quiz_score" class="quiz-completed">
				{{ $t('quiz.completed') }}
			</div>
		</div>
		<div v-else>
			<p>{{ $t('quiz.loading') }}</p>
		</div>
	</div>
</template>

<script>
import Question from './components/Question.vue';

export default {
	name: 'App',
	components: {
		Question,
	},
	data() {
		return {
			quizData_en: {},
			quizData_fr: {},
			currentQuestionIndex: 0,
			quizFinished: false,
			correctAnswers: 0,
			userAnswers: [],
			isQuizDataLoaded: false,
			savedAnswer: {},
		};
	},
	computed: {
		quizData() {
			return this.$i18n.locale === 'en'
				? this.quizData_en
				: this.quizData_fr;
		},
		buttonText() {
			return this.$i18n.locale === 'en' ? 'FR' : 'EN';
		},
		currentPageInfo() {
			if (this.quizData.questions.length > 1) {
				return this.$t('pageInfo', [
					this.currentQuestionIndex + 1,
					this.quizData.questions.length,
				]);
			}
		},
		score() {
			return this.calculateScore();
		},
		percentage() {
			const result =
				(this.score / this.quizData.questions.length) * 100;
			return Number.isInteger(result) ? result : result.toFixed(2);
		},
		passed() {
			return this.percentage >= this.quizData.passing_grade;
		},
		allQuestionsSubmitted() {
			if (this.quizData?.questions) {
				return (
					this.userAnswers.length ===
					this.quizData.questions.length &&
					!this.userAnswers.some((answer) => answer === null)
				);
			}
			return [];
		},
	},
	methods: {
		toggleLanguage() {
			const newLocale = this.$i18n.locale === 'en' ? 'fr' : 'en';
			this.$i18n.locale = newLocale;
			document.documentElement.lang = newLocale;
		},

		async loadQuizData() {
			try {
				// const response_en = await fetch("/content/enforced/12481-SB-Noam_Stulberg/test/QuizData_en.txt");
				const response_en = await fetch('QuizData_en.txt');
				const rawData_en = await response_en.text();
				this.quizData_en = JSON.parse(JSON.parse(rawData_en));

				// const response_fr = await fetch("/content/enforced/12481-SB-Noam_Stulberg/test/QuizData_fr.txt");
				const response_fr = await fetch('QuizData_fr.txt');
				const rawData_fr = await response_fr.text();
				this.quizData_fr = JSON.parse(JSON.parse(rawData_fr));
				const RandomOrder = Math.random() - 0.5;
				// Randomize questions if needed
				if (this.quizData_en.randomize_questions) {
					this.quizData_en.questions.sort(() => RandomOrder);
				}
				if (this.quizData_fr.randomize_questions) {
					this.quizData_fr.questions.sort(() => RandomOrder);
				}

				this.userAnswers = new Array(
					this.quizData.questions.length,
				).fill(null);
				this.isQuizDataLoaded = true;
			} catch (error) {
				console.error('Error fetching quiz data:', error);
			}
		},

		storeCurrentAnswer(answer, index) {
			this.savedAnswer[index] = answer;
		},
		showQuizFeedback() {
			this.quizFinished = true;
		},

		nextQuestion(isCorrect) {
			if (isCorrect) this.correctAnswers++;
			this.currentQuestionIndex++;
		},
		previousQuestion() {
			this.currentQuestionIndex--;
		},
		submitQuiz(answer, questionIndex) {
			this.userAnswers[questionIndex] = answer;
		},
		calculateScore() {
			return this.userAnswers.reduce((score, answer) => {
				if (answer) {
					score++;
				}
				return score;
			}, 0);
		},
		resetQuiz() {
			this.quizFinished = false;
			this.currentQuestionIndex = 0;
			this.correctAnswers = 0;
		},
		shuffleArray(array) {
			for (let i = array.length - 1; i > 0; i--) {
				const j = Math.floor(Math.random() * (i + 1));
				[array[i], array[j]] = [array[j], array[i]];
			}
			return array;
		},
		updateResults({ correctAnswers, totalQuestions }) {
			this.correctAnswers = correctAnswers;
			this.totalQuestions = totalQuestions;
		},
		finishQuiz() {
			this.quizFinished = true;
			this.$emit('finish');
		},
	},
	created() {
		this.loadQuizData();
	},
};
</script>
<style>
/* General styles */
:root {
	--school-purple: #3f2a56;
	--school-purple-drk: #18032f;
	--slate-blue: #4e5b73;
	--true-blue: #1c578a;
	--true-light: #226aa8;
	--system-gray: #f1f5fb;
	--body-grey: #333;
	--white-heat: #ffffff;
	--horizontal-rule: #ddd;
	--off-white: #f6f7f8;
	--hyperlink-blue: #2b4380;
	--hyperlink-hover: #0535d2;
	--hyperlink-visited: #7834bc;
	--hyperlink-menu: #337ab7;
	--disabled: #6c6c6c;
	--input-gradient: linear-gradient(to bottom, #e6e5e5, #fff 50%);
	--input-box-shadow: 0px 0px 8px 4px #3b99fc, 2px 5px 16px 0px #0b325e,
		0px 0px 21px 5px rgba(125, 195, 255, 0.3);
	--border-feedback-colour: var(--white-heat);
	--border-feedback-bg-colour: #369;
	--feedback-bg-colour: var(--white-heat);
	--feedback-colour: #333;
}

#app {
	max-width: 800px;
	margin: 40px auto;
}

.quiz {
	background-color: var(--white-heat);
	border-radius: 20px;
	box-shadow: 2px 2px 25px #cecece, 0px 4px 0 #cecece;
	margin: 60px auto;
	position: relative;
	border: 1px solid var(--white-heat);
}

.navigation-control {
	display: flex;
	justify-content: space-between;
	padding: 2rem;
	padding-top: 0;
}

.question-text {
	position: relative;
	display: block;
	text-align: left;
	font-size: 1.3em;
	color: var(--body-grey);
	margin-bottom: 15px;
	padding: 1.4rem 0.7rem;
}

#app ol,
ul {
	padding-inline-start: 0;
}

#app .quiz li {
	padding-bottom: 1.25rem;
}

input[type='checkbox']+label::before {
	margin-right: 10px;
}

.page-info {
	display: flex;
	justify-content: space-around;
}

#app fieldset {
	border: none;
	padding-left: 0;
}

#app legend {
	text-align: left;
	font-size: 1.3em;
	font-weight: 500;
	padding: 1.5rem 0;
}

#app .header {
	padding: 1.5rem 1.4rem 1rem;
	position: relative;
	text-align: center;
	border-bottom: 1px solid var(--white-heat);
}

#app .header::before {
	align-items: center;
	color: #000000;
	content: '?';
	display: flex;
	font-size: 2em;
	height: 60px;
	justify-content: center;
	left: 50%;
	line-height: 60px;
	position: absolute;
	top: -30px;
	width: 80px;
	-webkit-user-select: none;
	-moz-user-select: none;
	-ms-user-select: none;
	user-select: none;
	transform: translate(-50%);
	background-color: var(--white-heat);
	border-radius: 20px;
	border-top: 2px solid var(--off-white);
	border-bottom: 2px solid var(--white-heat);
}

.quiz-progressbar {
	position: absolute;
	left: 0;
	width: 100%;
	height: 4px;
	background-color: var(--slate-blue);
	box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
}

.quiz-feedback {
	padding-top: 20px;
	display: block;
	text-align: left;
}

.quiz-body {
	padding: 0 2.1rem;
}

.quiz-feedback {
	padding-top: 20px;
	display: block;
	text-align: left;
}

.quiz-body {
	padding: 0 2.1rem;
}

/*  Button styles  */
.button-control {
	display: flex;
	align-items: flex-end;
	justify-content: flex-end;
	padding-bottom: 2.1rem;
}

button.btn.btn-secondary:focus,
button.btn.btn-secondary:hover:not(:disabled) {
	border: 1px solid var(--white-heat);
	outline: 2px solid var(--true-blue);
	color: var(--white-heat);
	background-color: var(--slate-blue);
}

button.btn.btn-primary:focus,
button.btn.btn-primary:hover:not(:disabled) {
	outline: 2px solid var(--true-blue);
	border: 1px solid var(--white-heat);
	background-color: var(--school-purple);
}

/* Custom Radio inputs */
input[type='radio'] {
	position: absolute;
	opacity: 0;
	cursor: pointer;
	height: 0;
	width: 0;
}

input[type='radio']+label {
	display: block;
	position: relative;
	padding-left: 60px;
	margin-bottom: 30px;
	cursor: pointer;
	font-size: 22px;
	line-height: 36px;
	color: #000;
}

input[type='radio']+label::before {
	content: '';
	display: block;
	position: absolute;
	left: 0;
	top: 0;
	width: 36px;
	height: 36px;
	outline: 2px solid #000;
	border-radius: 50%;
	background-color: var(--white-heat);
}

input[type='radio']:checked+label::after {
	content: '';
	display: block;
	width: 26px;
	height: 26px;
	position: absolute;
	left: 5px;
	top: 5px;
	border-radius: 50%;
	background-color: #444;
}

input[type='radio']:focus+label::before {
	-webkit-box-shadow: var(--input-box-shadow);
	box-shadow: var(--input-box-shadow);
	outline: 3px solid #141414;
}

input[type='radio']:not(:disabled)+label:hover::before {
	border: 4px solid var(--white-heat);
	background-image: var(--input-gradient);
}

/* disabled input styles */
input[type='radio']:disabled,
input[type='radio']:disabled+label,
input[type='checkbox']:disabled,
input[type='checkbox']:disabled+label {
	cursor: default;
	color: var(--disabled);
}

input[type='radio']:disabled+label::after {
	background-color: var(--disabled);
	cursor: default !important;
}


/* Custom Checkbox */
input[type='checkbox'] {
	position: absolute;
	opacity: 0;
	cursor: pointer;
	height: 0;
	width: 0;
}

input[type='checkbox']:focus+label::before {
	-webkit-box-shadow: var(--input-box-shadow);
	box-shadow: var(--input-box-shadow);
	outline: 3px solid #141414;
}

input[type='checkbox']+label {
	display: block;
	position: relative;
	padding-left: 50px;
	padding-bottom: 20px;
	cursor: pointer;
	font-size: 22px;
	line-height: 24px;
	color: #000;
}

input[type='checkbox']+label::before {
	--_box-shadow: #000;
	content: '';
	display: inline-block;
	position: absolute;
	left: 0;
	top: 2px;
	width: 36px;
	height: 36px;
	border: 4px solid var(--white-heat);
	border-radius: 4px;
	background-color: var(--white-heat);
	box-shadow: 0 0 0 2px var(--_box-shadow);
}

input[type='checkbox']:checked+label::before {
	background-color: var(--white-heat);
}

input[type='checkbox']:not(:disabled)+label:hover::before {
	background-image: var(--input-gradient);
}

input[type='checkbox']:checked+label::after {
	content: '';
	position: absolute;
	left: 12px;
	top: 4px;
	width: 13px;
	height: 26px;
	transform: rotate(45deg);
	border-style: solid;
	border-width: 0 5px 5px 0;
}

/* Disabled checkbox */
input[type='checkbox']:disabled+label::before {
	--_box-shadow: var(--disabled);

}
</style>
