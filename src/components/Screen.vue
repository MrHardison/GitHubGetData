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
		<div class="table-responsive" v-show="showTable">
			<table class="table table-bordered table-hover table-striped">
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
		<div class="info-modal" :class="{visible: showModal}">
			<div class="close" @click="showModal = false">x</div>
			<div class="title">
				About commit
			</div>
			<div class="line">
				<div class="text">Author</div>
				<div class="value">{{subInfo ? subInfo.commit.author.name : ''}}</div>
			</div>
			<div class="line">
				<div class="text">Id</div>
				<div class="value">{{subInfo ? subInfo.author.id : ''}}</div>
			</div>
			<div class="line">
				<div class="text">Url</div>
				<div class="value">{{subInfo ? subInfo.commit.url : ''}}</div>
			</div>
			<div class="line">
				<div class="text">Additions</div>
				<div class="value">{{subInfo ? subInfo.stats.additions : ''}}</div>
			</div>
			<div class="line">
				<div class="text">Deletions</div>
				<div class="value">{{subInfo ? subInfo.stats.deletions : ''}}</div>
			</div>
			<div class="line">
				<div class="text">Total</div>
				<div class="value">{{subInfo ? subInfo.stats.total : ''}}</div>
			</div>
		</div>
	</div>
</template>

<script>
	import axios from 'axios'
	import Modal from './modal'
	
	export default {
		name: 'Screen',
		data() {
			return {
				name: '',
				repository: '',
				pattern: /^[a-zA-Z\.\-\_]{4,30}$/,
				commits: null,
				errorMessage: '',
				subInfo: null,
				nameError: '',
				repoError: '',
				showTable: false,
				showModal: false
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
				this.showTable = true
			},
			showError(error) {
				this.errorMessage = error
			},
			viewMore(index) {
				axios.get('https://api.github.com/repos/' + this.name + '/' + this.repository + '/commits/' + this.commits[index].sha)
				.then(response => {
					this.subInfo = response.data
					this.showModal = true
				})
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
	padding: 50px 0;
	position: relative;
	height: 100vh;
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
.info-modal {
	box-shadow: 0 2px 50px 0 rgba(0,0,0,.2);
	background: #fff;
	border-radius: 5px;
	box-sizing: border-box;
	padding: 30px;
	left: 50%;
	opacity: 0;
	transition: opacity .5s;
	position: absolute;
	top: 0;
	text-align: left;
	transform: translate(-50%, 50%);
	width: 500px;
	z-index: -1;
}
.info-modal .close {
	position: absolute;
	right: 20px;
	top: 10px;
	cursor: pointer;
	font-size: 20px;
}
.info-modal .title {
	text-align: center;
	font-size: 24px;
	margin-bottom: 20px;
}
.info-modal .line {
	padding: 10px 0;
	border-bottom: 1px solid rgba(0,0,0,.1);
}
.info-modal .line:last-child {
	border-bottom: none;
}
.info-modal .line .text {
	font-size: 12px;
	text-transform: uppercase;
}
.info-modal .line .value {
	font-size: 14px;
	font-weight: 700;
	overflow: hidden;
	text-overflow: ellipsis;
	width: 100%;
}
.info-modal.visible {
	opacity: 1;
	z-index: 2;
}
</style>
