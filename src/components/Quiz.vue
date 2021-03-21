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
			<form v-if="currentQuestion" class="buttons">
				<button
					v-for="answer in currentQuestion.answers"
					:qgroup="currentQuestion.key"
					:key="answer"
					:index = "index"
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
	// "data()" guarda variáveis de estado
	data() {
		return {
			customQuiz: [],
			questions: [],
			loading: true,
			playing: true,
			index: 0,
			score: 0,
			timer: 10,
		};
	},
	// as propriedades "computed" designam funções simples a utilizar pelo template
	computed: {
		// esta função retorna a pergunta apontada pelo index atual
		currentQuestion() {
			if (this.questions !== []) {
				return this.questions[this.index]; // normalmente no Vue a keyword "this" remete para as variáveis em "data()"
			}
			return null;
		},
	},
	// Os métodos de um componente Vue são guardados na propriedade "methods"
	methods: {
		/** 
		 * Este método é usado para retirar, manipular, e guardar as perguntas do questionário
		 * 
		 * O parâmetro "custom" indica se o qustionário a ser usado é o questionário "base" ou 
		 * o questionário indicado pelo utilizador
		 */
		fetchQuestions(custom) {
			this.loading = true;

			var jsonResponse;

			if (!custom) {
				jsonResponse = require("../../public/Quiz.json");
			}
			else {
				jsonResponse = this.customQuiz;
			}
			let index = 0;

			// esta função manipula as perguntas do quiz e guarda-as num novo array "data"
			
			let data = jsonResponse.results.map((question) => {
				// aqui coloca-se todas as respostas num único array
				question.answers = [
					question.correct_answer,
					...question.incorrect_answers,
				];
				// as respostas são baralhadas
				for (let i = question.answers.length - 1; i > 0; i--) {
					const j = Math.floor(Math.random() * (i + 1));
					[question.answers[i], question.answers[j]] = [
						question.answers[j],
						question.answers[i],
					];
				}
				// é adicionada uma propriedade "key" para distinguir as perguntas
				question.key = index;
				index++;
				return question;
			});

			/* ========= Função de randomização ========= */

			let arr = [];
			for (let i = 0; i < data.length; i++) {
				arr.push(i);
			}
			var currentIndex = arr.length;
			var	temporaryValue;
			var	randomIndex;

			// Enquanto existirem elementos não baralhados...
			while (currentIndex !== 0) {
				// Escolher um elemento qualquer
				randomIndex = Math.floor(Math.random() * currentIndex);
				currentIndex--;
				// e trocar com o elemento atual
				temporaryValue = arr[currentIndex];
				arr[currentIndex] = arr[randomIndex];
				arr[randomIndex] = temporaryValue;
			}

			// substituir os valores de data pelos valores randomizados
			var newdata = [];
			for (let i = 0; i < arr.length; i++) {
				newdata.push(data[arr[i]]);
			}

			/* ========= Função de randomização ========= */

			// coloca-se os dados na variável "questions"
			this.questions = newdata;

			this.loading = false;

			// apresenta o contador de perguntas
			document.getElementById("question-number").innerHTML =
				"Question number 1/" + this.questions.length;
		},
		/**
		 * Este método ocorre quando o utilizador escolhe uma das opções
		 * 
		 * O parâmetro "event" remete para a ação de clicar no botão
		 */
		handleButtonClick: function (event) {
			// encontra o index que descreve a pergunta
			let index = event.target.getAttribute("index");
			console.log(index);

			// encontra a resposta que o utilziador escolheu
			let userAnswer = event.target.innerHTML;

			// coloca a propriedade da pergunta "userAnswer" com a resposta do utilizador
			this.questions[index].userAnswer = userAnswer;

			// informa que o butão foi carregado -> para o CSS
			event.target.classList.add("clicked");

			// desliga os outros botões
			let allButtons = document.querySelectorAll(`[index="${index}"]`);
			for (let i = 0; i < allButtons.length; i++) {
				if (allButtons[i] === event.target) continue;
				allButtons[i].setAttribute("disabled", "disabled");
			}

			// invoca a função checkAnswer para verificar a resposta
			this.checkAnswer(event, index);
		},
		/**
		 * Este método é invocado após a manipulação incial dos botões e verifica se a resposta do utilizador 
		 * está correta
		 * 
		 * Os parâmetros "index" e "event" asseguram a continuidade do método anterior
		 */
		checkAnswer: function (event, index) {
			let question = this.questions[index];
			if (question.userAnswer) {
				if (this.index < this.questions.length - 1) {
					setTimeout(
						function () {
							// atualiza o index atual
							this.index += 1;

							// atualiza o número da questão atual
							document.getElementById("question-number").innerHTML = 
								"Question number " + (this.index + 1) + "/" + this.questions.length;

							// ao passar à próxima pergunta, reinicializa os atributos dos botões
							// isto impede conflitos quando várias perguntas têm respostas iguais a perguntas anteriores
							let allButtons = document.querySelectorAll(`[index="${index}"]`);
							for (let i = 0; i < allButtons.length; i++) {
								allButtons[i].removeAttribute("disabled");
								allButtons[i].removeAttribute("class");
							}

						}.bind(this),
						1000
					);
				}
			}

			if (question.userAnswer === question.correct_answer) {
				// Se a resposta estiver correta, a classe "rightAnswer" é adicionada -> para CSS
				event.target.classList.add("rightAnswer");

				// quando o utilizador acerta uma pergunta, adiciona 5 segundos ao temporizador
				// e a sua pontuação aumenta
				if (this.timer + 5 < 30) {
					this.timer += 5;
				}
				this.score++;
			} else {
				// Se a resposta estiver errada, a classe "wrongAnswer" é adicionada -> para CSS
				event.target.classList.add("wrongAnswer");

				// quando o utilizador erra uma pergunta, o temporizador é diminuido 3 segundos
				this.timer -= 3;

				// é também adicionada uma classe "showRightAnswer" para mostrar ao utilzador 
				// qual a reposta certa da pergunta -> para CSS
				let correctAnswer = this.questions[index].correct_answer;
				let allButtons = document.querySelectorAll(`[index="${index}"]`);
				allButtons.forEach(function (button) {
					if (button.innerHTML === correctAnswer) {
						button.classList.add("showRightAnswer");
					}
				});
			}
			// no fim da última pergunta o quiz é removido e mostra-se a pontuação final do utilizador
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
		/**
		 * Este método dá ínicio ao quiz quando o utilizador carrega no botão "start-quiz" ou "start-custom-quiz"
		 * 
		 * O parâmetro "custom" permite saber se é o quiz base ou um quiz submetido
		 */
		startQuiz(custom) {
			// esconde os elementos que já não são necessários, mostra o quiz
			// e chama os métodos "fetchQuestions()" e "counDownTimer()"
			document.getElementById(custom ? "start-custom-quiz" : "start-quiz").hidden = true;
			document.getElementById(custom ? "file-label" : "custom-quiz").hidden = true;
			document.getElementById("quiz").hidden = false;
			this.fetchQuestions(custom);
			this.countDownTimer();
		},
		/**
		 * Este método dá início ao temporizador do quiz
		 */
		countDownTimer() {
			document.getElementById("count-down-timer").innerHTML =
				"Time remaining: " + this.timer;

			// A cada intervalo de 1000ms, o contador decresce
			// se o contador chegar ao fim, acaba o quiz e mostra a pontuação
			// do jogador
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
		/**
		 * Este método apenas serve para esconder certos elementos e fazer aparecer outros
		 * quando o utilizador quiser submeter o seu próprio quiz
		 */
		customQuizLayout() {
			document.getElementById("file-label").hidden = false;
			document.getElementById("start-custom-quiz").hidden = false;
			document.getElementById("start-quiz").hidden = true;
			document.getElementById("custom-quiz").hidden = true;
		},
		/**
		 * Este método converte o ficheiro ".txt" submetido pelo utilizador
		 * para estar de acordo com o formato de quiz lido pelo método
		 * "fetchQuestions"
		 */
		createCustomQuiz() {
			const fileInput = document.getElementById('file-input').files[0];
			// é criada um objeto FileReader que irá ler o ficheiro submetido
			var file = new FileReader();
			file.onload = () => {
				var questions = {
					results: []
				};

				// aqui procura-se por expressões comuns de fim de linha para poder
				// separar o conteúdo dos ficheiros por linhas e guardar num array
				var lines = file.result.split(/\r\n|\n\r|\n/).filter(function(value) {
					if (value !== '') return true;
				});

				// por sua vez esse array é de novo separado em perguntas e repostas
				// e é guardado na variável "questions"
				for (const i in lines) {
					if (i%2 === 0) {
						var pergunta = lines[i];
                    }
                    else {
						var respostas = lines[i].split(';');
                        questions.results.push({
							question: pergunta,
                            correct_answer: respostas[0],
                            incorrect_answers: respostas.slice(1,4)
                        })
                    }
                }
				
				// após isto são chamados os métodos "fetchQuestions" e "startQuiz"
				// com o parâmetro "custom" verdadeiro
				this.customQuiz = questions;
				this.fetchQuestions(true);
				this.startQuiz(true);
			}
			file.readAsText(fileInput);
		},
	}
};
</script>

<style scoped>
/* Display do conjunto de botões */
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
/* Estilos do Quiz.vue para UX na resposta no utilizador */
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
