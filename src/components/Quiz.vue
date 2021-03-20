<template>
	<div id="quiz-container">
		<img id="neec-logo" src="../../public/Logo_NEEC_Blue.png" />
		<h1 id="logo-headline">Web-Dev Quiz</h1>
		<h1 id="question-number"></h1>
		<h1 id="count-down-timer"></h1>
		<hr class="divider" />
		<div class="buttons">
			<button id="start-quiz" @click.prevent="startQuiz(false)">Start Quiz</button>
			<button id="custom-quiz" @click.prevent="customQuizLayout">
				Custom Quiz
			</button>
			<input id="file-input" type="file" accept=".txt" hidden="true" />
			<label id="file-label" for="file-input" hidden="true">
				Upload Custom Quiz
			</label>
			<button id="start-custom-quiz" @click="createCustomQuiz" hidden="true">
				Start Custom Quiz
			</button>
		</div>

		<div id="quiz" hidden="true">
			<h1 v-html="loading ? 'Loading...' : currentQuestion.question"></h1>
			<!-- Aqui usamos o operador ternário para verificar se o quiz já está carregado -->
			<form v-if="currentQuestion">
				<button
					v-for="answer in currentQuestion.answers"
					:qgroup="currentQuestion.key"
					:key="answer"
					v-html="answer"
					@click.prevent="handleButtonClick"
				></button>
			</form>
		</div>
		<h1 id="score" hidden="true"></h1>
		<hr class="divider" />
	</div>
</template>


<script>
export default {
	name: "Quiz",
	data() {
		// data() function stores state variables
		return {
			customQuiz: [],
			questions: [],
			loading: true,
			playing: true,
			index: 0, // initialize index at 0
			score: 0,
			timer: 10,
		};
	},
	computed: {
		// this computed property returns the current question at the current index
		currentQuestion() {
			if (this.questions !== []) {
				// "this" usually refers to the Vue component instance properties, e.g. "this.questions" points to the
				return this.questions[this.index]; // questions array in our data() function
			}
			return null;
		},
	},
	// All methods of a Vue component are written on the methods property of the compenent instance
	// Custom methods of the Vue component
	methods: {
		//this method is used to fetch the questions, manipulate them, and store them in the questions array
		fetchQuestions(custom) {
			this.loading = true;

			var jsonResponse;

			if (!custom) {
				jsonResponse = require("../../public/Quiz.json");
			}
			else {
				jsonResponse = this.customQuiz;
			}
			let index = 0; // index is used to identify single answer
			// manipulate questions
			let data = (custom ? jsonResponse : jsonResponse.results).map((question) => {
				// put answers on question into single array
				question.answers = [
					question.correct_answer,
					...question.incorrect_answers,
				];
				// shuffle question.answers array
				for (let i = question.answers.length - 1; i > 0; i--) {
					const j = Math.floor(Math.random() * (i + 1));
					[question.answers[i], question.answers[j]] = [
						question.answers[j],
						question.answers[i],
					];
				}
				// add rightAnswer and key property to each question
				question.rightAnswer = null;
				question.key = index;
				index++;
				return question;
			});

			let arr = [];
			for (let i = 0; i < data.length; i++) {
				arr.push(i);
			}
			var currentIndex = arr.length,
				temporaryValue,
				randomIndex;
			// While there remain elements to shuffle...
			while (0 !== currentIndex) {
				// Pick a remaining element...
				randomIndex = Math.floor(Math.random() * currentIndex);
				currentIndex -= 1;
				// And swap it with the current element.
				temporaryValue = arr[currentIndex];
				arr[currentIndex] = arr[randomIndex];
				arr[randomIndex] = temporaryValue;
			}

			var newdata = [];
			for (let i = 0; i < arr.length; i++) {
				newdata.push(data[arr[i]]);
			}

			// put data on questions property
			this.questions = newdata;

			this.loading = false;
			document.getElementById("question-number").innerHTML =
				"Question number " + (this.index + 1) + "/" + this.questions.length;
		},
		handleButtonClick: function (event) {
			// find index to identify question object in data
			let index = this.index; //event.target.getAttribute("index");

			// innerHTML is polluted with decoded HTML entities e.g ' from &#039;
			let userAnswer = event.target.innerHTML;

			// clear from pollution with '
			//let userAnswer = pollutedUserAnswer.replace(/'/, "&#039;");

			// set userAnswer on question object in data
			this.questions[index].userAnswer = userAnswer;

			// set class "clicked" on button with userAnswer -> for CSS Styles
			// disable other sibling buttons
			event.target.classList.add("clicked");
			let allButtons = document.querySelectorAll(`[qgroup="${index}"]`);
			for (let i = 0; i < allButtons.length; i++) {
				if (allButtons[i] === event.target) continue;
				allButtons[i].setAttribute("disabled", "disabled");
			}
			// Invoke checkAnswer to check Answer
			this.checkAnswer(event, index);
			this.questions[index].userAnswer = null;
		},
		checkAnswer: function (event, index) {
			let correct;
			console.log(correct);
			let question = this.questions[index];
			if (question.userAnswer) {
				correct = (question.userAnswer === question.correct_answer);
				if (this.index < this.questions.length - 1) {
					setTimeout(
						function () {
							this.index += 1;
							document.getElementById("question-number").innerHTML =
								"Question number " +
								(this.index + 1) +
								"/" +
								this.questions.length;
								event.target.classList.remove("clicked");
								event.target.classList.remove(correct ? "rightAnswer" : "wrongAnswer");
								if (correct) {
									let allButtons = document.querySelectorAll(`[qgroup="${index}"]`);
									let correctAnswer = this.questions[index].correct_answer;
									allButtons.forEach(function (button) {
										if (button.innerHTML === correctAnswer) {
											button.classList.remove("showRightAnswer");
										}
									});
								}
						}.bind(this),
						1000
					);
				}
			}

			if (correct) {
				// Set class on Button if user answered right, to celebrate right answer with animation joyfulButton
				event.target.classList.add("rightAnswer");

				// Set rightAnswer on question to true, computed property can track a streak out of 10 questions
				this.questions[index].rightAnswer = true;

				if (this.timer + 5 < 30) {
					this.timer += 5;
				}
				this.score++;
			} else {
				// Mark users answer as wrong answer
				event.target.classList.add("wrongAnswer");
				this.questions[index].rightAnswer = false;
				this.timer -= 3;

				// Show right Answer
				let correctAnswer = this.questions[index].correct_answer;
				let allButtons = document.querySelectorAll(`[qgroup="${index}"]`);
				allButtons.forEach(function (button) {
					if (button.innerHTML === correctAnswer) {
						button.classList.add("showRightAnswer");
					}
				});
			}
			if (this.questions.length - 1 === this.index) {
				setTimeout(() => {
					this.playing = false;
					document.getElementById("quiz").hidden = true;
					document.getElementById("score").hidden = false;
					document.getElementById("score").innerHTML =
						"You got " +
						this.score +
						" out of " +
						this.questions.length +
						" questions right!";
				}, 1000);
			}
		},
		startQuiz(custom) {
			document.getElementById(custom ? "start-custom-quiz" : "start-quiz").hidden = true;
			document.getElementById(custom ? "file-label" : "custom-quiz").hidden = true;
			document.getElementById("quiz").hidden = false;
			this.fetchQuestions(custom);
			this.countDownTimer();
		},
		countDownTimer() {
			document.getElementById("count-down-timer").innerHTML =
				"Time remaining: " + this.timer;
			let interval = setInterval(() => {
				this.timer--;
				if (this.timer <= 0 || !this.playing) {
					this.timer = 0;
					document.getElementById("quiz").hidden = true;
					document.getElementById("score").hidden = false;
					document.getElementById("score").innerHTML =
						"You got " +
						this.score +
						" out of " +
						this.questions.length +
						" questions right!";
					clearInterval(interval);
				}
				document.getElementById("count-down-timer").innerHTML =
					"Time remaining: " + this.timer;
			}, 1000);
		},
		customQuizLayout() {
			document.getElementById("file-label").hidden = false;
			document.getElementById("start-custom-quiz").hidden = false;
			document.getElementById("start-quiz").hidden = true;
			document.getElementById("custom-quiz").hidden = true;
		},
		createCustomQuiz() {
			const fileInput = document.getElementById('file-input').files[0];
			var file = new FileReader();
			file.onload = () => {
				var lines = file.result.split(/\r\n|\n\r|\n/).filter(function(value) {
					if (value !== '') return true;
				});
				var questions = [];
				for (const i in lines) {
					if (i%2 === 0) {
						var pergunta = lines[i];
                    }
                    else {
						var respostas = lines[i].split(';');
                        questions.push({
							question: pergunta,
                            correct_answer: respostas[0],
                            incorrect_answers: respostas.slice(1,4)
                        })
                    }
                }
				this.customQuiz = questions;
				this.fetchQuestions(true);
				this.startQuiz(true);
			}
			file.readAsText(fileInput);
		},
	},
	// We want the Component to fetch and store the data, when the Component mounts
};
</script>

<style scoped>
form {
	display: flex;
	flex-direction: row;
	flex-wrap: wrap;
	justify-content: center;
}
.buttons {
	display: flex;
	flex-direction: row;
	flex-wrap: wrap;
	justify-content: center;
}
label {
	font: 400 13.3333px Arial;
	font-size: 1.1rem;
	font-weight: 700;
	box-sizing: border-box;
	padding: 1rem;
	margin: 0.3rem;
	width: 47%;
	background-color: rgba(100, 100, 100, 0.3);
	border: none;
	border-radius: 0.4rem;
	box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.2);
	cursor: pointer;
}
label:hover {
	transform: scale(1.02);
	box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.14), 0 1px 7px 0 rgba(0, 0, 0, 0.12),
		0 3px 1px -1px rgba(0, 0, 0, 0.2);
}
button {
	font-size: 1.1rem;
	font-weight: 700;
	box-sizing: border-box;
	padding: 1rem;
	margin: 0.3rem;
	width: 47%;
	background-color: rgba(100, 100, 100, 0.3);
	border: none;
	border-radius: 0.4rem;
	box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.2);
	cursor: pointer;
}
button:hover {
	transform: scale(1.02);
	box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.14), 0 1px 7px 0 rgba(0, 0, 0, 0.12),
		0 3px 1px -1px rgba(0, 0, 0, 0.2);
}
button:focus {
	outline: none;
}
button:active:enabled {
	transform: scale(1.05);
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
	background: linear-gradient(
		210deg,
		rgba(0, 178, 72, 0.25),
		rgba(0, 178, 72, 0.5)
	);
}
button.wrongAnswer {
	color: black;
	background: linear-gradient(
		210deg,
		rgba(245, 0, 87, 0.25),
		rgba(245, 0, 87, 0.5)
	);
}
button.showRightAnswer {
	animation: flashButton;
	animation-duration: 700ms;
	animation-delay: 200ms;
	animation-iteration-count: 2;
	animation-timing-function: ease-in-out;
	color: black;
	background: linear-gradient(
		210deg,
		rgba(0, 178, 72, 0.25),
		rgba(0, 178, 72, 0.5)
	);
}
#quiz-container {
	text-align: center;
	margin: 1rem auto;
	padding: 1rem;
	max-width: 750px;
}
#logo-headline {
	font-size: 3rem;
	padding: 0.5rem;
	color: #000000;
	text-align: center;
}
#neec-logo {
	display: block;
	width: 40%;
	margin: 0 auto;
}
@media only screen and (max-width: 500px) {
	#neec-logo {
		width: 30%;
	}
	#logo-headline {
		font-size: 1.8rem;
	}
}
/* Styles in Quiz.vue for UX on user answer */
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
h1 {
	font-size: 1.3rem;
	padding: 0.7rem;
	text-shadow: 1px 3px 3px rgba(0, 0, 0, 0.3);
}
.divider {
	margin: 0.5rem 0;
	border: 3.7px solid rgba(0, 0, 0, 0.7);
	border-radius: 3px;
	box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.3);
}
</style>
