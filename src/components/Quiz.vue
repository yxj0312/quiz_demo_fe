<template>
    <div class="container">
        <div class="correctAnswers" v-if="completed">
            You have
            <strong>{{ point }} Points !</strong>
        </div>
        <h1 v-if="currentQuestion" v-text="currentQuestion.question"></h1>

        <form v-if="currentQuestion">
            <button v-for="answer in currentQuestion.answers" :index="currentQuestion.key" :key="answer" v-text="answer"
                @click.prevent="handleClick">


            </button>

        </form>

        <button @click.prevent="next" v-text="completed ? 'Reset' : 'Next'"></button>
    </div>


</template>

<script>
export default {
    data() {
        return {
            questions: [],
            index: 0,
            point: 0
        }
    },

    computed: {
        currentQuestion() {
            if (this.questions !== []) {
                return this.questions[this.index];
            }
            return null;
        },
        completed() {
            return this.index === 4;
        }
        // score() {
        //     return {
        //         allQuestions: this.questions.length,
        //         answeredQuestions: this.questions.reduce((count, currentQuestion) => {
        //             if (currentQuestion.userAnswer) {
        //                 // userAnswer is set when user has answered a question, no matter if right or wrong
        //                 count++;
        //             }
        //             return count;
        //         }, 0),
        //         correctlyAnsweredQuestions: this.questions.reduce(
        //             (count, currentQuestion) => {
        //                 if (currentQuestion.rightAnswer) {
        //                     // rightAnswer is true, if user answered correctly
        //                     count++;
        //                 }
        //                 return count;
        //             },
        //             0
        //         ),

        //     }
        // }
    },

    watch: {

    },

    mounted() {
        this.fetchQuestions()
    },


    methods: {
        async fetchQuestions() {
            let response = await fetch(
                "http://127.0.0.1:8000/api/quizzes"
            );
            let data = await response.json();

            let index = 0;

            let questions = data.map((question) => {
                //add right answers and key
                question.rightAnswer = null;
                question.key = index;
                index++;
                return question;
            });
            this.questions = questions;
        },

        handleClick(e) {
            let index = e.target.getAttribute("index");
            let userAnswer = e.target.innerHTML;

            this.questions[index].userAnswer = userAnswer;
            e.target.classList.add("clicked");
            let allButtons = document.querySelectorAll(`[index="${index}"]`);


            for (let i = 0; i < allButtons.length; i++) {
                if (allButtons[i] === e.target) continue;
                // allButtons[i].setAttribute("disabled", "");
            }
            this.checkCorrectAnswer(e, index);
        },

        checkCorrectAnswer(e, index) {
            let question = this.questions[index];

            if (question.userAnswer) {
                if (question.correct_answers.includes(question.userAnswer)) {
                    e.target.classList.add("rightAnswer");
                    this.questions[index].rightAnswer = true;
                    this.point++;
                } else {
                    e.target.classList.add("wrongAnswer");
                    this.questions[index].rightAnswer = false;
                    /* Show right Answer */
                    let correctAnswers = this.questions[index].correct_answers;
                    let allButtons = document.querySelectorAll(`[index="${index}"]`);
                    correctAnswers.forEach(element => {
                        allButtons.forEach(function (button) {
                            if (button.innerHTML === element) {
                                button.classList.add("showRightAnswer");
                            } else {
                                button.classList.add("wrongAnswer");
                            }
                        });
                    });
                }
            }
        },

        next() {
            if (this.index <= 4) {
                this.index += 1;
            }
        }
    }
}
</script>

<style>
.container {
    margin: 1rem auto;
    padding: 1rem;
    max-width: 750px;
}

form {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
}

button {
    font-size: 1.1rem;
    box-sizing: border-box;
    padding: 1rem;
    margin: 0.5rem;
    width: 40%;
    background-color: rgba(100, 100, 100, 0.3);
    border: none;
    border-radius: 0.4rem;
    box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.2);
}

button:hover:enabled {
    transform: scale(1.02);
    box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.14), 0 1px 7px 0 rgba(0, 0, 0, 0.12),
        0 3px 1px -1px rgba(0, 0, 0, 0.2);
    cursor: pointer;
}

button:focus {
    outline: none;
}

button:active:enabled {
    transform: scale(1.05);
}

@keyframes flashButton {
    0% {
        opacity: 1;
        transform: scale(1.01);
    }

    50% {
        opacity: 0.7;
        transform: scale(1.02);
    }

    100% {
        opacity: 1;
        transform: scale(1);
    }
}

button.clicked {
    pointer-events: none;
}

button.rightAnswer {
    animation: flashButton;
    animation-duration: 700ms;
    animation-delay: 200ms;
    animation-iteration-count: 3;
    animation-timing-function: ease-in-out;
    color: black;
    background: linear-gradient(210deg,
            rgba(0, 178, 72, 0.25),
            rgba(0, 178, 72, 0.5));
}

button.wrongAnswer {
    color: black;
    background: linear-gradient(210deg,
            rgba(245, 0, 87, 0.25),
            rgba(245, 0, 87, 0.5));
}

button.showRightAnswer {
    animation: flashButton;
    animation-duration: 700ms;
    animation-delay: 200ms;
    animation-iteration-count: 2;
    animation-timing-function: ease-in-out;
    color: black;
    background: linear-gradient(210deg,
            rgba(0, 178, 72, 0.25),
            rgba(0, 178, 72, 0.5));
}
</style>