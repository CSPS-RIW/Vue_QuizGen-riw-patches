<template>
	<div>
		<span v-for="(part, index) in generateDisplayedFillInBlanks" :key="index">
			<select :disabled="preventChange" v-if="part.isSelect" @change="onSelect($event, part.index)">
				<option
					v-for="(option, optionIndex) in part.options"
					:key="optionIndex"
					:value="optionIndex"
					:disabled="optionIndex == 0"
					:selected="optionIndex == userAnswers[part.index]"
				>
					{{ option }}
				</option>
			</select>
			<template v-else>
				{{ part.content }}
			</template>
		</span>
	</div>
</template>

<script>
export default {
	emits: ["userAnswered"],
	props: ["question", "previousAnswer","preventChange"],
	data() {
		return {
			userAnswers: this.previousAnswer
				? [...this.previousAnswer]
				: Array(this.question.answer_options.length).fill(0),
			submitted: false,
			correct: false,
		};
	},

	computed: {
		generateDisplayedFillInBlanks() {
			if (!this.question.question_text || !this.question.answer_options) {
				return [];
			}

			const parts = [];
			const splits = this.question.question_text.split("___");
			const numOptions = this.question.answer_options.length;

			for (let i = 0; i < splits.length; i++) {
				parts.push({ isSelect: false, content: splits[i] });

				if (i < numOptions) {
					const options = this.question.answer_options[i];
					parts.push({
						isSelect: true,
						index: i,
						options: options.options,
					});
				}
			}

			return parts;
		},
	},
	methods: {
		onSelect(event, index) {
			this.userAnswers[index] = parseInt(event.target.value);
			this.$emit("userAnswered", this.userAnswers);
		},
	},
};
</script>
