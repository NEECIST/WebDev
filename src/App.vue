<template>
	<div id="app">
		<div class="top">
			<div class="container-center">
				<h1>Speed Quiz</h1>
			</div>
		</div>
		<Quiz />
		<footer>
			<p id="createdBy">
				created by <a href="https://github.com/NEECIST">NEECIST</a>
			</p>
		</footer>
	</div>
</template>

<script>
import Quiz from "@/components/Quiz.vue";

export default {
	name: "App",
	components: {
		Quiz,
	},
};
</script>

<style>
/* Aplica-se a tudo */
* {
	box-sizing: border-box;
	margin: 0;
}

/* Importa font do public */
@font-face {
	font-family: Arcade;
	src: url(../public/PressStart2P-Regular.ttf);
}

/* Por class */
.top {
	background-color: #009de0;
	font-family: Arcade;
	font-size: 14px;
	display: flex;
}

.container-center {
	display: flex;
	height: auto;
	width: auto;
	margin: auto;
}

.top h1 {
	margin: 5px;
	color: white;
	font-size: 2.5em;
}

/* Por id */
#app {
	font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
		Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji",
		"Segoe UI Symbol";
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

#createdBy {
	float: right;
	color: white;
	font-weight: bold;
}

#createdBy a {
	color: white;
}

/* Por tag */
footer {
	position: fixed;
	bottom: 0;
	padding: 0.5rem 1rem;
	width: 100%;
	font-size: 0.7rem;
	background-color: #009de0;
}
</style>