<script setup>
import { ref } from 'vue'
import AForm from './components/a-form.vue'
import AReport from './components/a-report.vue'

const reportData = ref(null)
let isLoading = ref(false)

function handleWeatherReport(data) {
	reportData.value = data
}

function toggleLoading(data) {
	isLoading.value = data
}
</script>

<template>
	<div class="header">
		<h1>ATC weather report</h1>
	</div>
	<div class="body">
		<AForm @isLoading="toggleLoading" @weatherReportReceived="handleWeatherReport"></AForm>
		<AReport :reportData="reportData" :isLoading="isLoading"></AReport>
	</div>
</template>

<style>
.header {
	height: 50px;
	width: 100%;
	background-color: rgb(239, 239, 239);
	border-bottom: 1px solid rgba(90, 90, 90, 0.5);
	display: flex;
	justify-content: center;
	font-family: cursive;
}

.body {
	margin-bottom: 50px;
}
</style>
