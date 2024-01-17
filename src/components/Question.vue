<template>
	<div class="quiz" v-if="data" tabindex="-1">
		<div v-if="data.question_type === 'single-select'">
			<div id="Header" class="header">
				<div>
					<p class="question-text" :class="data.question_text.split(' ').length < 8 && 'center-text'">{{
						data.question_text }}</p>
				</div>
				<div class="quiz-progressbar"></div>
			</div>
			<div class="quiz-body">
				<fieldset>
					<legend>{{ data.instructions }}</legend>
					<div v-for="(option, qindex) in shuffledAnswerOptions" :key="qindex">
						{{ option.correct_answer }}
						<input :key="qindex" type="radio" :id="`option-${qindex}`" :name="`question-${index}`"
							:value="qindex" :checked="userAnswers === qindex" :disabled="preventNextChange"
							@change="handleAnswerChange(qindex)" />
						<label :for="`option-${qindex}`" :class="{
							'correct-answer':
								displayIndividualOptionFeedback &&
								isAnswerCorrect(qindex) === true &&
								userAnswers === qindex,
							'incorrect-answer':
								displayIndividualOptionFeedback &&
								isAnswerCorrect(qindex) === false &&
								userAnswers === qindex,
						}">
							{{ option.text }}
							<span v-if="displayIndividualOptionFeedback &&
								isAnswerCorrect(qindex) === true &&
								userAnswers === qindex" class="checkmark">&#10003;</span>
							<span v-else-if="displayIndividualOptionFeedback &&
								isAnswerCorrect(qindex) === false &&
								userAnswers === qindex" class="xmark">&#10007;</span>
							<div v-if="displayIndividualOptionFeedback && option.feedback" class="option-feedback">
								{{ option.feedback }}
							</div>
						</label>
					</div>
				</fieldset>
			</div>
		</div>
		<div v-if="data.question_type === 'multiple-select' && userAnswers?.length
			">
			<div id="Header" class="header">
				<div>
					<p class="question-text" :class="data.question_text.split(' ').length < 8 && 'center-text'">{{
						data.question_text }}</p>
				</div>
				<div class="quiz-progressbar"></div>
			</div>

			<div class="quiz-body">
				<fieldset>
					<legend>{{ data.instructions }}</legend>
					<div v-for="(option, qindex) in data.answer_options" :key="qindex" class="input-wrapper">
						<input type="checkbox" v-model="userAnswers[qindex]" :value="qindex" :id="`option-${qindex}`"
							:disabled="preventNextChange" @change="updateMultipleSelectAnswer($event, index, qindex)"
							@keyup.enter="checkWithEnter(qindex)" />
						<label :for="`option-${qindex}`" :class="{
							'correct-answer':
								displayIndividualOptionFeedback &&
								isAnswerCorrect(qindex) === true,
							'incorrect-answer':
								displayIndividualOptionFeedback &&
								isAnswerCorrect(qindex) === false,
						}">
							{{ option.text }}
							<!-- <span v-if="displayIndividualOptionFeedback && isAnswerCorrect(qindex) === true"
								class="checkmark">&#10003;</span>
							<span v-else-if="displayIndividualOptionFeedback && isAnswerCorrect(qindex) === false"
								class="xmark">&#10007;</span> -->
							<!-- Display feedback -->
							<div v-if="displayIndividualOptionFeedback && this.submitted[index]"
								:class="[isAnswerCorrect(qindex) ? 'CorrectFeedback' : 'IncorrectFeedback']">
								<span class="feedback-icon" aria-hidden="true"></span>
								{{ option.feedback }}
							</div>
							<span v-else-if="displayIndividualOptionFeedback &&
								isAnswerCorrect(qindex) === false" class="xmark">

							</span>

						</label>
					</div>
				</fieldset>
			</div>
		</div>

		<div v-if="data.question_type === 'true-false'">
			<div id="Header" class="header">
				<p class="question-text" :class="data.question_text.split(' ').length < 8 && 'center-text'">{{
					data.question_text }}</p>
				<div class="quiz-progressbar"></div>
			</div>
			<div class="quiz-body">
				<fieldset>
					<legend>{{ data.instructions }}</legend>
					<div v-for="(option, qindex) in data.answer_options" :key="qindex">
						<input type="radio" :value="qindex" :id="`option-${qindex}`" :name="`question-${index}`"
							:checked="userAnswers === qindex" :disabled="preventNextChange"
							@change="handleAnswerChange(qindex)" />
						<label :for="`option-${qindex}`" :class="displayIndividualOptionFeedback
							? answerClasses(qindex)
							: ''
							">
							{{ option.text }}
							<span v-if="displayIndividualOptionFeedback &&
								userAnswers !== undefined &&
								isAnswerCorrect(qindex) === true &&
								userAnswers === qindex
								" class="checkmark">&#10003;</span>
							<span v-else-if="displayIndividualOptionFeedback &&
								userAnswers !== undefined &&
								isAnswerCorrect(qindex) === false &&
								userAnswers === qindex
								" class="xmark">&#10007;</span>
						</label>
					</div>
				</fieldset>
			</div>
		</div>
		<div v-if="data.question_type === 'fill-in-the-blanks'">
			<div id="Header" class="header">
				<div class="quiz-progressbar"></div>
			</div>
			<div class="quiz-body">
				<fieldset>
					<legend>{{ data.instructions }}</legend>
					<FillInTheBlanks :question="data" :previousAnswer="userAnswers" :preventChange="preventNextChange"
						@userAnswered="handleFillInTheBlanks"></FillInTheBlanks>
				</fieldset>
			</div>
		</div>
		<div v-if="data.question_type === 'highlight-correct-sentences'">
			<div id="Header" class="header">
				<div class="quiz-progressbar"></div>
			</div>
			<div class="quiz-body">
				<fieldset>
					<legend>{{ data.instructions }}</legend>
					<HighlightCorrectSentence :question="data" :previousAnswer="userAnswers"
						:preventChange="preventNextChange" @userAnswered="handleHighlightCorrectSentence">
					</HighlightCorrectSentence>
				</fieldset>
			</div>
		</div>
		<div v-if="data.question_type === 'drag-and-drop'">
			<drag-drop-activity :data_uncategorized-notes="mappedData.uncategorizedNotes"
				:data_categories="mappedData.categories" @submit="handleDragDropSubmit"></drag-drop-activity>
		</div>

		<div class="quiz-body quiz-feedback-body" aria-live="polite" tabindex="-1">
			<div class="quiz-feedback" v-if="isSubmitted && feedbackRecap">
				<div :class="isCorrect ? 'CorrectFeedback' : 'IncorrectFeedback'
					">
					<span class="feedback-icon" aria-hidden="true"></span>
					<div v-html="isCorrect
						? data.correct_feedback
						: data.incorrect_feedback
						"></div>
					<div v-html="data.generic_feedback"></div>
				</div>
			</div>
		</div>

		<div class="quiz-body button-control">
			<button class="btn btn-secondary btn-retry" v-if="!preventChangingAnswers && isSubmitted && !isCorrect"
				@click="reset">
				{{ $t("question.retry") }}
			</button>
			<button class="btn btn-primary " v-else :disabled="preventNextChange" @click="submit">
				{{ $t("question.submit") }}
			</button>
		</div>

		<div v-if="!(index === 0 && index === lastIndex)" class="navigation-control">
			<button class="btn btn-secondary" @click="previous" :disabled="index === 0">
				{{ $t("question.previous") }}
			</button>
			<button class="btn btn-secondary" :disabled="index === lastIndex" @click="next">
				{{ $t("question.next") }}
			</button>
		</div>
	</div>
</template>
<script>
import DragDropActivity from '@/components/DragDropActivity.vue';
import FillInTheBlanks from '@/components/FillInTheBlanks.vue';
import HighlightCorrectSentence from '@/components/HighlightCorrectSentence.vue';

export default {
	emits: ['next', 'previous', 'save-answers', 'submit'],
	components: {
		DragDropActivity,
		FillInTheBlanks,
		HighlightCorrectSentence,
	},
	props: {
		data: Object,
		index: Number,
		lastIndex: Number,
		preventChangingAnswers: Boolean,
		displayIndividualOptionFeedback: { type: Boolean, default: false },
		savedAnswer: Object,
	},
	data() {
		return {
			listType: '-',
			userAnswers: null,
			submitted: Array(this.lastIndex + 1).fill(null),
			isCorrect: false,
			answerStates: {
				submitted: Array(this.lastIndex + 1).fill(false),
				userAnswers: [],
			},
		};
	},

	computed: {
		preventNextChange() {
			return (
				(this.preventChangingAnswers && this.isSubmitted) ||
				this.isSubmitted
			);
		},
		isSubmitted() {
			return this.submitted[this.index] === true;
		},

		shuffledAnswerOptions() {
			if (this.data.randomize_answers) {
				return this.shuffleArray(this.data.answer_options);
			} else {
				return this.data.answer_options;
			}
		},
		preventChange() {
			return this.$parent.quizData.prevent_changing_answers;
		},
		feedbackRecap() {
			return this.$parent.quizData.feedback_recap;
		},
		mappedData() {
			const uncategorizedNotes = this.data.answer_options.map(
				(option, index) => ({
					id: index + 1,
					text: option.draggable,
					selected: false,
				}),
			);

			const categories = this.data.answer_options.map(
				(option, index) => ({
					name: option.dropzone,
					notes: [],
				}),
			);

			return {
				uncategorizedNotes,
				categories,
			};
		},
	},
	methods: {
		handleAnswerChange(optionIndex) {
			if (this.data.question_type === 'multiple-select') {
				this.userAnswers[optionIndex] = !this.userAnswers[optionIndex];
			} else {
				this.userAnswers = optionIndex;
			}
		},
		handleFillInTheBlanks(answers) {
			this.userAnswers = answers;
		},
		handleHighlightCorrectSentence(answers) {
			this.userAnswers = answers;
		},
		initializeUserAnswers() {
			if (
				this.data.question_type === 'single-select' ||
				this.data.question_type === 'true-false'
			) {
				this.userAnswers = this.savedAnswer
					? this.savedAnswer.userAnswers
					: null;
			} else if (this.data.question_type === 'multiple-select') {
				this.userAnswers = this.savedAnswer
					? this.savedAnswer.userAnswers
					: new Array(this.data.answer_options.length).fill(
						false,
					);
			} else if (this.data.question_type === 'fill-in-the-blanks') {
				this.userAnswers = this.savedAnswer
					? this.savedAnswer.userAnswers
					: new Array(this.data.answer_options.length).fill(0);
			}
		},

		shuffleArray(array) {
			let currentIndex = array.length,
				temporaryValue,
				randomIndex;

			// While there remain elements to shuffle...
			while (0 !== currentIndex) {
				// Pick a remaining element...
				randomIndex = Math.floor(Math.random() * currentIndex);
				currentIndex -= 1;

				// And swap it with the current element.
				temporaryValue = array[currentIndex];
				array[currentIndex] = array[randomIndex];
				array[randomIndex] = temporaryValue;
			}

			return array;
		},
		next() {
			// Store the state of userAnswers and submitted arrays
			this.storeQuestionState(this.index);
			this.isCorrect = false;
			this.$emit("next");
			this.setFocus()
		},
		previous() {
			// Store the state of userAnswers and submitted arrays
			this.storeQuestionState(this.index);
			this.$emit("previous");
			this.setFocus()
		},
		setFocus() {
			let quizBody = document.querySelector('.quiz');
			quizBody.focus();
		},
		storeQuestionState() {
			this.answerStates = {
				userAnswers: this.userAnswers,
				submitted: this.submitted[this.index],
			};
			this.$emit("save-answers", this.answerStates, this.index);
		},
		restoreQuestionState() {
			if (this.savedAnswer) {
				this.userAnswers = this.savedAnswer.userAnswers;
				this.submitted[this.index] = this.savedAnswer.submitted;
			} else {
				this.initializeUserAnswers();
			}
		},
		feedbackDivFocus() {
			// Find the next available button to focus on it
			let feedbackDiv = document.querySelector('.quiz-feedback-body');

			feedbackDiv.focus();
		},

		submit() {
			// Check if the user hasn't selected an answer
			if (
				(this.data.question_type === 'single-select' ||
					this.data.question_type === 'true-false') &&
				(this.userAnswers === null ||
					this.userAnswers === undefined)
			) {
				alert(this.$t('question.pleaseSelectAnswer'));
				return;
			} else if (
				this.data.question_type === 'multiple-select' &&
				!this.userAnswers.some((selected) => selected)
			) {
				alert(this.$t('question.pleaseSelectAtLeastOneAnswer'));
				return;
			} else if (
				this.data.question_type === 'fill-in-the-blanks' &&
				!this.userAnswers.every((answer) => answer !== 0)
			) {
				alert(this.$t('question.pleaseSelectEveryTerm'));
				return;
			}

			this.submitted[this.index] = true; // Set submitted for the current page to true
			const isCorrect = this.checkIfCorrect(
				this.data,
				this.userAnswers,
			);
			this.isCorrect = isCorrect;
			this.$emit('submit', isCorrect, this.index);
			this.feedbackDivFocus();
		},

		reset() {
			this.submitted[this.index] = false;
			if (this.data.question_type === 'multiple-select') {
				this.userAnswers = this.userAnswers.map(() => false);
			} else {
				this.userAnswers = null;
			}
			this.setFocus()
		},

		checkIfCorrect(question, userAnswer) {
			let isCorrect = false;

			switch (question.question_type) {
				case 'single-select':
					if (userAnswer !== null && userAnswer !== undefined) {
						isCorrect =
							question.answer_options[userAnswer].isCorrect;
					}
					break;
				case 'multiple-select':
					if (userAnswer) {
						const totalCorrectOptions =
							question.answer_options.filter(
								(option) => option.isCorrect,
							).length;

						const selectedOptionsCount = userAnswer.reduce(
							(count, selected) => count + (selected ? 1 : 0),
							0,
						);

						const selectedCorrectOptions = userAnswer.reduce(
							(count, selected, index) => {
								return (
									count +
									(selected &&
										question.answer_options[index].isCorrect
										? 1
										: 0)
								);
							},
							0,
						);

						isCorrect =
							totalCorrectOptions ===
							selectedCorrectOptions &&
							selectedOptionsCount === selectedCorrectOptions;
					} else {
						isCorrect = false;
					}
					break;
				case 'true-false':
					isCorrect = userAnswer === question.correct_answer;
					break;
				case 'fill-in-the-blanks':
					isCorrect = question.answer_options.every(
						(options, index) =>
							options.correctIndex === userAnswer[index],
					);

					break;
				case 'drag-and-drop':
					// Assuming the correct order is represented by the correct_answer array
					isCorrect = true;

					break;
				default:
					break;
			}

			return isCorrect;
		},

		isAnswerCorrect(qindex) {
			if (!this.isSubmitted || this.userAnswers === undefined) {
				return null;
			}

			const userAnswer = this.userAnswers;

			if (this.data.question_type === 'multiple-select') {
				return (
					userAnswer[qindex] ===
					this.data.answer_options[qindex].isCorrect
				);
			} else {
				return (
					userAnswer === qindex &&
					this.data.answer_options[qindex].isCorrect
				);
			}
		},

		answerClasses(qindex) {
			if (!this.isSubmitted || this.userAnswers === undefined) {
				return {};
			}

			const isCorrect = this.isCorrect;

			if (
				this.data.question_type === 'single-select' ||
				this.data.question_type === 'true-false'
			) {
				if (isCorrect && this.userAnswers === qindex) {
					return { 'correct-answer': true };
				} else if (this.userAnswers === qindex) {
					return { 'incorrect-answer': true };
				}
			} else if (this.data.question_type === 'multiple-select') {
				if (isCorrect === true) {
					return { 'correct-answer': true };
				} else if (isCorrect === false) {
					return { 'incorrect-answer': true };
				}
			}
			return {};
		},

		handleDragDropSubmit(categoriesWithNotes) {
			// Logic to handle the submitted data
			// categoriesWithNotes is an array of categories containing an object "notes" which is an array
		},
		updateMultipleSelectAnswer(event, index, qindex) {
			if (!this.userAnswers) {
				this.userAnswers = {};
			}
			this.userAnswers[qindex] = event.target.checked;
		},
		checkWithEnter(qindex) {
			// Handle checking the checkbox on Enter key press
			if (this.data.question_type === 'single-select') {
				this.handleAnswerChange(qindex);
			} else if (this.data.question_type === 'multiple-select') {
				this.userAnswers[qindex] = !this.userAnswers[qindex];
			}
		},
	},
	mounted() {
		if (this.data) {
			// this.answerStates.submitted = Array(this.lastIndex + 1).fill(false);
			// this.answerStates.userAnswers = Array(this.lastIndex + 1).fill(null);
			this.initializeUserAnswers();
		}
	},

	watch: {
		savedAnswer: {
			handler(newSavedAnswer) {
				if (newSavedAnswer) {
					this.userAnswers = newSavedAnswer.userAnswers;
				} else {
					this.initializeUserAnswers();
				}
			},
			immediate: true,
		},
		index: {
			handler() {
				// Restore the state of userAnswers and submitted arrays
				this.restoreQuestionState(this.index);
				// if (this.userAnswers === undefined) {
				// 	this.initializeUserAnswers();
				// }
			},
			immediate: true,
		},
	},
};
</script>

<style scoped>
/* Style question header*/

.header {
	padding: 1.5rem 1.4rem 1rem;
	position: relative;
	border-bottom: 1px solid var(--white-heat);
}

.header::before {
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

.header .question-text {
	position: relative;
	display: block;
	text-align: left;
	font-size: 1.3em;
	color: var(--body-grey);
	margin-bottom: 15px;
	padding: 1.4rem 0.7rem;
}

.header .question-text.center-text {
	text-align: center;

}

/* Style quiz when focused */
.quiz[tabindex='-1']:focus-visible {
	outline: 2px solid var(--body-grey) !important;
}

/* style feedback body when focused */
.quiz-feedback-body[tabindex='-1']:focus-visible {
	outline: 2px solid var(--body-grey) !important;
}

.correct-answer {
	color: green;
}

.incorrect-answer {
	color: red;
}

.checkmark {
	margin-left: 5px;
}

.xmark {
	margin-left: 5px;
}

label {
	user-select: none;
}

/* Feedback styles */
.CorrectFeedback,
.IncorrectFeedback{
	position: relative;
	outline-style: solid;
	outline-width: 2px;
	outline-color: var(--border-feedback-bg-colour);
	background-color: var(--feedback-bg-colour);
	color: var(--feedback-colour);
	border-radius: 12px;
	padding: 1rem 1.5rem;
	margin-bottom: 15px;
}

.CorrectFeedback .feedback-icon::after,
.IncorrectFeedback .feedback-icon::after {
	content: '';
	color: var(--border-feedback-colour);
	background-color: var(--border-feedback-bg-colour);
	width: 30px;
	height: 30px;
	position: absolute;
	top: -10px;
	right: -11px;
	background-origin: padding-box;
	padding: 0px 0px 0px 9px;
	border-radius: 50%;
}

.IncorrectFeedback {
	--border-feedback-bg-colour: #9e0404;
	--feedback-bg-colour: #f3e2e2;
	--feedback-colour: #2b0000;
}

.IncorrectFeedback .feedback-icon::after {
	content: '\2716';
	padding: 0px 0px 0px 7px;
	outline: 2px solid #ffffff00;
}

.CorrectFeedback {
	--border-feedback-bg-colour: #18703a;
	--feedback-bg-colour: #e2f3e8;
	--feedback-colour: #072b00;
}

.CorrectFeedback .feedback-icon::after {
	content: '\2714';
	padding: 0px 0px 0px 7px;
	outline: 2px solid #ffffff00;
}
</style>
