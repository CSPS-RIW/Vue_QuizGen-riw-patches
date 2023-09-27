# QuizGen

QuizGen is a Vue.js quiz generator component that provides a variety of question types, including single-select, multiple-select, true/false, fill-in-the-blanks, and drag-and-drop. It allows you to easily create and customize quizzes for your applications.

## Features

- Supports multiple question types:
  - Single-select
  - Multiple-select
  - True/false
  - Fill-in-the-blanks
  - Drag-and-drop
- Randomize answer options
- Prevent users from changing answers
- Display individual option feedback
- Save and restore answer states

## Props

| Prop                        | Type    | Default | Description                                                                                     |
|-----------------------------|---------|---------|-------------------------------------------------------------------------------------------------|
| data                        | Object  | -       | The quiz data object                                                                            |
| index                       | Number  | -       | The current question index                                                                      |
| lastIndex                   | Number  | -       | The index of the last question                                                                  |
| preventChangingAnswers      | Boolean | false   | Prevent users from changing their answers after submission                                      |
| displayIndividualOptionFeedback | Boolean | false   | Display feedback for each individual answer option                                              |
| savedAnswer                 | Object  | -       | The saved answer object containing user answers and submission states                           |

## Events

| Event          | Payload           | Description                                          |
|----------------|-------------------|------------------------------------------------------|
| next           | -                 | Go to the next question                              |
| previous       | -                 | Go to the previous question                          |
| save-answers   | answerStates, index | Save user answers and submission states for a question |
| submit         | isCorrect, index  | Submit the user's answers for a question            |


## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
