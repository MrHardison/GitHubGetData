<template>
	<div class="page">
		<form class="page__form">
			<div class="input-box">
				<div class="title">Enter user name:</div>
				<input type="text" name="user" @input="userName($event.target.value)">
				<div class="input-error">{{nameError}}</div>
			</div>
			<div class="input-box">
				<div class="title">Enter repository name:</div>
				<input type="text" name="repo" @input="repoName($event.target.value)">
				<div class="input-error">{{repoError}}</div>
			</div>
			<button 
				class="submit btn btn-primary"
				@click.prevent="getData">
				Get data
			</button>
			<div class="error">{{errorMessage}}</div>
		</form>
		<div class="table-responsive">
			<table class="table table-bordered table-hover table-striped" v-if="!showTable">
				<tr>
					<th>Sha</th>
					<th>Url</th>
					<th>Author</th>
					<th>Email</th>
					<th>Date</th>
					<th>Action</th>
				</tr>
				<template v-for="(commit, index) in commits">
					<tr :key="index">
						<td>{{commit.sha}}</td>
						<td>{{commit.url}}</td>
						<td>{{commit.commit.author.name}}</td>
						<td>{{commit.commit.author.email}}</td>
						<td>{{commit.commit.author.date}}</td>
						<td><div class="btn btn-primary" @click="viewMore(index)">View more</div></td>
					</tr>
				</template>
			</table>
		</div>
	</div>
</template>

<script>
	import axios from 'axios'
	export default {
		name: 'Screen',
		data() {
			return {
				name: '',
				repository: '',
				pattern: /^[a-zA-Z]{4,30}$/,
				showTable: false,
				commits: null,
				errorMessage: '',
				nameError: '',
				repoError: ''
			}
		},
		methods: {
			getData() {
				if(this.nameError || this.repoError) {
					return this.errorMessage = 'Username or repository name is incorrent'
				} else if(!this.name || !this.repository) {
					return this.errorMessage = 'Username or repository name is empty'
				}
				axios.get('https://api.github.com/repos/' + this.name + '/' + this.repository + '/commits')
				.then(response => this.showData(response.data))
				.catch(error => this.showError(error.response.data.message))
			},
			userName(value) {
				if(this.pattern.test(value)) {
					this.name = value
					this.nameError = ''
					this.errorMessage = ''
				} else if(!this.pattern.test(value)) {
					this.nameError = 'Please, enter a valid username'
				}
			},
			repoName(value) {
				if(this.pattern.test(value)) {
					this.repository = value
					this.repoError = ''
					this.errorMessage = ''
				} else if(!this.pattern.test(value)) {
					this.repoError = 'Please, enter a valid repository name'
				}
			},
			showData(response) {
				this.commits = response
			},
			showError(error) {
				this.errorMessage = error
			},
			viewMore(index) {
				axios.get('https://api.github.com/repos/' + this.name + '/' + this.repository + '/commits/' + this.commits[index].sha)
				.then(response => console.log('response.data: ', response.data))
				.catch(error => this.showError(error.response.data.message))
			}
		}
	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
	margin: 0;
	padding: 0;
}
.page {
	margin: 0 auto;
}
.page__form {
	display: inline-block;
	margin: 0 auto;
	text-align: center;
	width: 300px;
}
.input-box {
	margin: 0 auto;
	margin-bottom: 50px;
	position: relative;
	text-align: left;
	width: 300px;
}
.input-error {
	bottom: -18px;
	color: red;
	font-size: 12px;
	left: 0;
	position: absolute;
}
.title {
	font-size: 16px;
}
input {
	width: 100%;
}
.error {
	margin-top: 20px;
	font-size: 16px;
	color: red;
	min-height: 30px;
}
.table {
	margin: 0 auto;
	margin-top: 100px;
	width: 1000px
}
th {
	text-transform: uppercase;
}
td {
	max-width: 30%;
	vertical-align: middle;
}
</style>
