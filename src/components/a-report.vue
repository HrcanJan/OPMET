<script setup>
import { computed } from 'vue'

const props = defineProps({
	reportData: {
		type: [Object, null],
		required: true,
	},
	isLoading: {
		type: Boolean,
		required: true
	}
})

function formatTime(time) {
	const date = new Date(time)
	
	const day = String(date.getDate()).padStart(2, '0')
	const month = String(date.getMonth() + 1).padStart(2, '0')
	const year = date.getFullYear()

	const hours = String(date.getHours()).padStart(2, '0')
	const minutes = String(date.getMinutes()).padStart(2, '0')
	const seconds = String(date.getSeconds()).padStart(2, '0')

	return `${day}.${month}.${year} ${hours}:${minutes}:${seconds}`
}

const groupedReportData = computed(() => {
	if (!props.reportData?.result) return {}

	return props.reportData.result.reduce((acc, row) => {
		if (!acc[row.stationId]) {
			acc[row.stationId] = []
		}
		acc[row.stationId].push(row)
		return acc
	}, {})
})
</script>

<template>
	<div v-if="props.isLoading">
		<div v-if="props.isLoading" class="spinner-container">
			<div class="spinner"></div>
		</div>
	</div>
	<div v-else>
		<div v-if="props.reportData?.result">
			<table v-for="(rows, stationId) in groupedReportData" :key="stationId">
				<thead>
					<tr>
						<th class="col-1">{{ stationId }}</th>
						<th class="col-2"></th>
						<th class="col-3"></th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="(row, index) in rows" :key="index">
						<td class="col-1">{{ row.queryType }}</td>
						<td class="col-2">{{ formatTime(row.reportTime) }}</td>
						<td v-if="row.textHTML" class="col-3" v-html="row.textHTML"></td>
						<td v-else class="col-3">{{ row.text }}</td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
</template>

<style scoped>
th {
	background-color: rgb(155, 155, 155);
	font-weight: bold;
}

td {
	background-color: rgb(220, 220, 220);
}

table {
	width: 800px;
	border-collapse: collapse;
	margin: auto auto;
	font-family: cursive;
	font-size: small;
	margin-bottom: -1px;
}

th, td {
	padding: 0 8px;
	text-align: left;
	border: 1px solid #000000;
	vertical-align: top;
}

.col-1 {
	width: 15%;
}

.col-2 {
	width: 20%;
}

.col-3 {
	width: 65%;
}

.spinner-container {
	display: flex;
	justify-content: center;
	align-items: center;
	height: 100px;
}

.spinner {
	border: 4px solid rgba(0, 0, 0, 0.1);
	width: 40px;
	height: 40px;
	border-radius: 50%;
	border-left-color: #000;
	animation: spin 1s ease infinite;
}

@keyframes spin {
	0% {
		transform: rotate(0deg);
	}
	100% {
		transform: rotate(360deg);
	}
}

@media (max-width: 805px) {
	table {
		width: 100%;
		font-size: smaller;
	}
}

@media (max-width: 500px) {
	table {
		width: 100%;
		font-size: x-small;
	}
}
</style>
