<template>
	<div id="app" class="container">
		<div style="flex: 1;">
			<div class="input__div">
				<div class="input__wrapper">
					<!-- add new task to the List. Works with enter or the button below -->
					<input type="text" placeholder="Write your task here" v-model="newTaskText" @keyup.enter="addNewTodo" />
				</div>
				<div class="border"/>
			</div>
			<button v-on:click="addNewTodo">Add task</button>
			<!-- Loop over all existing states for dynamically create lists. If a new state is added a new Headline or edit the conditions is recommended -->
			<div class="todo-list" v-for="state of taskState">
				<!-- Show headline for each state -->
				<h2 v-if="state === taskState.OPEN && remaining > 0">Not finished tasks</h2>
				<h2 v-else-if="state === taskState.COMPLETE && remaining < taskList.length">Finished tasks</h2>
				<!-- Show only the tasks belong to current state -->
				<div v-for="(todo, index) in taskList" v-if="todo.done === state" class="list">
					<label class="material-checkbox">
						<!--
						Currently only two states (finished and open) are available. If more States are added its recommended to change the checkbox
						for something like a dropdown to distinguish the states
						-->
						<input type="checkbox" v-bind:checked="todo.done === taskState.COMPLETE" v-on:change="onChangeState(todo, $event.target.checked)">
						<span></span>
					</label>
					<!-- add class "completed" only for finished tasks -->
					<div class="text" :class="{completed: todo.done === taskState.COMPLETE}">{{ todo.title }}</div>
					<button v-on:click="removeTodo(index)">Remove</Button>
				</div>
			</div>
		</div>
		<!-- if tasks are unfinished show the amount -->
		<footer v-if="remaining > 0">
			<span>{{ remaining }} tasks left.</span>
		</footer>
	</div>
</template>

<script>
import axios from 'axios'
import config from './config.js'

export default {
	name: 'app',

	data () {
		return {
			// load list of possible states from config
			taskState: config.taskState,
			newTaskText: '',
			// fill task list with test data. 
			taskList: [
				{ title: 'Call Adam', done: config.taskState.OPEN, },
				{ title: 'Buy concert tickets', done: config.taskState.COMPLETE, },
				{ title: 'Call dad', done: config.taskState.COMPLETE, },
				{ title: 'Buy flowers', done: config.taskState.OPEN, },
				{ title: 'Send email to John', done: config.taskState.OPEN, },
			],
		}
	},

	computed: {
		// calculates how many tasks aren't finished and return the numeric value
		remaining() {
			return this.taskList.filter(todo => todo.done != config.taskState.COMPLETE).length;
		}
	},

	methods: {
		// API CALLS
		// currently these are dummy calls, therefore no error handling is required
		addNewTodo: function () {
			axios
				.get(config.apiUrl)
				.then( response => {
					if(response.status === 200 && response.data.editSuccess){
						this.taskList.push({
							title: this.newTaskText,
							done: config.taskState.OPEN
						});
						this.newTaskText = '';
					}
				});
		},
		removeTodo: function (index) {
			axios
				.get(config.apiUrl)
				.then( response => {
					this.taskList.splice(index, 1);
				});
		},
		onChangeState: function(todo, newVal) {
			axios
				.get(config.apiUrl)
				.then( response => {
					if(newVal){
						todo.done = config.taskState.COMPLETE;
					} else {
						todo.done = config.taskState.OPEN;
					}
				});
		},
	}
}
</script>

<style>
* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}
body {
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  color: #2c3e50;
  font-family: Karla, sans-serif;
  font-size: 16px;
  margin: 0;
}

::-webkit-scrollbar {
  height: 10px;
  width: 10px;
}

::-webkit-scrollbar-thumb {
  background-clip: padding-box;
  background-color: #b4c7d0;
  border: 3px solid transparent;
  border-radius: 5px;
}

::-webkit-scrollbar-track-piece {
  background: 0 0;
}

#app {
  max-width: 510px;
  margin: 50px auto;
  padding: 0 10px;
}

img {
  display: block;
  width: 80px;
  height: 80px;
  margin: 0 auto 15px auto;
}

p.title {
  font-size: 20px;
  font-weight: bold;
  text-align: center;
}
.container {
  min-height: 300px;
  display: flex;
  flex-direction: column;
}

footer {
  padding: 8px 15px;
  background: #76dbae;
  border-radius: 3px;
}

.todo-list {
  padding: 8px 0;
}

.todo-list .list:hover {
  background: #f7fcfa;
}

.todo-list .list {
  display: flex;
  align-items: center;
  padding: 0 15px;
}

.todo-list .list .text {
  padding: 0 8px;
  height: 35px;
  line-height: 35px;
  margin: 6px 0;
  cursor: default;
  flex: 1;
}

.todo-list .list .text.completed {
  text-decoration: line-through;
}

.input__div {
  margin: 6px 0;
  position: relative;
  border: 1px solid #e4f5ef;
  flex: 1;
}

.input__div:focus-within .input__wrapper {
  background: #fff;
}

.input__div .input__wrapper {
  background: #f7fcfa;
  padding: 0 15px;
  transition: background 0.3s;
}

.input__div .input__wrapper input {
  height: 35px;
  background: 0 0;
  border: none;
  color: #2c3e50;
  display: block;
  font-family: inherit;
  font-size: 16px;
  line-height: 16px;
  outline: 0;
  padding: 0;
  position: relative;
  width: 100%;
  z-index: 1;
}

.input__div .border {
  background: #42b983;
  transition: all 0.18s;
  bottom: -1px;
  height: 2px;
  left: 30%;
  opacity: 0;
  pointer-events: none;
  position: absolute;
  right: 30%;
}

.input__div:focus-within .border {
  left: 0;
  right: 0;
  opacity: 1;
}

.material-checkbox {
  position: relative;
  display: inline-block;
  color: rgba(0, 0, 0, 0.87);
  cursor: pointer;
  font-size: 14px;
  line-height: 18px;
}

.material-checkbox > input {
  appearance: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  position: absolute;
  z-index: -1;
  left: -5px;
  top: -5px;
  display: block;
  margin: 0;
  border-radius: 50%;
  width: 18px;
  height: 18px;
  background-color: rgba(0, 0, 0, 0.42);
  outline: none;
  opacity: 0;
  transform: scale(1);
  -ms-transform: scale(0); /* Graceful degradation for IE */
  transition: opacity 0.5s, transform 0.5s;
}

.material-checkbox > input:checked {
  background-color: #2196f3;
}

.material-checkbox > input:disabled {
  opacity: 0;
}

.material-checkbox > input:disabled + span {
  cursor: initial;
}

.material-checkbox > span::before {
  content: "";
  display: inline-block;
  margin-right: 15px;
  border: solid 2px rgba(0, 0, 0, 0.42);
  border-radius: 2px;
  width: 14px;
  height: 14px;
  vertical-align: -4px;
  transition: border-color 0.5s, background-color 0.5s;
}

.material-checkbox > input:checked + span::before {
  border-color: #41b883;
  background-color: #41b883;
}

.material-checkbox > input:active + span::before {
  border-color: #41b883;
}

.material-checkbox > input:checked:active + span::before {
  border-color: transparent;
  background-color: rgba(0, 0, 0, 0.42);
}

.material-checkbox > span::after {
  content: "";
  display: inline-block;
  position: absolute;
  top: 0;
  left: 0;
  width: 5px;
  height: 10px;
  border: solid 2px transparent;
  border-left: none;
  border-top: none;
  transform: translate(5.5px, 1px) rotate(45deg);
  -ms-transform: translate(5.5px, 2px) rotate(45deg);
}

.material-checkbox > input:checked + span::after {
  border-color: #fff;
}
</style>
