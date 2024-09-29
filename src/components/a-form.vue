<script setup>
import { ref } from 'vue'

const check1 = ref(false)
const check2 = ref(false)
const check3 = ref(false)
const airports = ref('')
const countriesList = ref('')

const missingCheckbox = ref(false)
const missingAirportCountry = ref(false)

const wrongAirportFormat = ref(false)
const wrongCountryFormat = ref(false)

let counter = 1

const emit = defineEmits(['weatherReportReceived', 'isLoading', 'stopLoading'])

function validateSubmit(event) {
	event.preventDefault()

	missingCheckbox.value = false
	missingAirportCountry.value = false
	wrongAirportFormat.value = false
	wrongCountryFormat.value = false

	if (!check1.value && !check2.value && !check3.value) {
		missingCheckbox.value = true
	}

	if (airports.value == '' && countriesList.value == '') {
		missingAirportCountry.value = true
	}

	const airportPattern = /^[A-Z]{4}( [A-Z]{4})*$/
	if (airports.value !== '' && !airportPattern.test(airports.value.toUpperCase())) {
		wrongAirportFormat.value = true
	}

	const countryPattern = /^[A-Z]{2}( [A-Z]{2})*$/
	if (countriesList.value !== '' && !countryPattern.test(countriesList.value.toUpperCase())) {
		wrongCountryFormat.value = true
	}

	if (!missingCheckbox.value && !missingAirportCountry.value && !wrongAirportFormat.value && !wrongCountryFormat.value) {
		handleSubmitForm()
	}
}

function handleSubmitForm() {
	const reportTypes = []
	if (check1.value) reportTypes.push('METAR')
	if (check2.value) reportTypes.push('SIGMET')
	if (check3.value) reportTypes.push('TAF')

	const stations = airports.value !== '' ? airports.value.split(' ') : []
	const countries = countriesList.value !== '' ? countriesList.value.split(' ') : []

	const prcMessage = {
		"id": "query" + counter,
		"method": "query",
		"params": [
			{
				"id": "briefing" + counter,
				"reportTypes": reportTypes,
				...(stations.length > 0 ? { stations } : {}),
				...(countries.length > 0 ? { countries } : {})
			}
		]
	}

	counter++
	createBriefing(prcMessage)
}

function createBriefing(prcMessage) {
	emit('isLoading', true)
	fetch('https://ogcie.iblsoft.com/ria/opmetquery', {
		method: 'POST',
		headers: {
			'Content-Type': 'application/json'
		},
		body: JSON.stringify(prcMessage)
	})
	.then(response => response.json())
	.then(data => {
		console.log('Weather report:', data)
		emit('weatherReportReceived', data)
		emit('isLoading', false)
	})
	.catch(error => {
		console.error('Error fetching weather report:', error)
		emit('isLoading', false)
	})
}
</script>

<template>
	<div>
		<form @submit="validateSubmit">
			<!-- Message types -->
			<div class="row">
				<span>Message types:</span>
				<div class="checkboxes full">
					<div class="custom-checkbox">
						<input type="checkbox" id="metar" name="METAR" v-model="check1">
						<label for="metar">METAR</label>
					</div>
					<div class="custom-checkbox">
						<input type="checkbox" id="sigmet" name="SIGMET" v-model="check2">
						<label for="sigmet">SIGMET</label>
					</div>
					<div class="custom-checkbox">
						<input type="checkbox" id="taf" name="TAF" v-model="check3">
						<label for="taf">TAF</label>
					</div>
				</div>
			</div>
			<div v-if="missingCheckbox" class="row error">
				<p>At least one message type must be selected</p>
			</div>

			<!-- Airports -->
			<div class="row">
				<span>Airports:</span>
				<input class="full" type="text" id="airports" name="airports" v-model="airports">
			</div>
			<div v-if="wrongAirportFormat" class="row error">
				<p>Should only contain airport codes (4 letters) separated by spaces</p>
			</div>

			<!-- Countries -->
			<div class="row">
				<span>Countries:</span>
				<input class="full" type="text" id="countries" name="countries" v-model="countriesList">
			</div>
			<div v-if="missingAirportCountry" class="row error">
				<p>At least one airport (ICAO code) or country (WMO code) is required</p>
			</div>
			<div v-if="wrongCountryFormat" class="row error">
				<p>Should only contain country codes (2 letters) separated by spaces</p>
			</div>

			<div class="button">
				<button type="submit">Create briefing</button>
			</div>
		</form>
	</div>
</template>

<style scoped>
form {
	background-color: white;
	font-size: small;
	width: 500px;
	padding: 20px 25px;
	border: 1px solid black;
	font-family: cursive;
	margin: 50px auto 50px auto
}

@media (max-width: 805px) {
	form {
		width: 100%;
		margin: 50px 0px 50px 0px;
		font-size: smaller;
	}
}

@media (max-width: 500px) {
	form {
		width: 100%;
		margin: 50px 0px 50px 0px;
		font-size: x-small;
	}
}

.row {
	display: flex;
	gap: 30px;
	margin-bottom: 10px;
}

.row span {
	width: 30%;
}

.error {
	color: red;
	font-size: smaller;
	margin-left: 28%;
	margin-top: -5px;
}

.custom-checkbox {
	position: relative;
	padding-left: 25px;
	cursor: pointer;
	user-select: none;
	display: inline-block;
}

.custom-checkbox input {
	position: absolute;
	opacity: 0;
	cursor: pointer;
	height: 0;
	width: 0;
}

.custom-checkbox label:before {
	content: "";
	position: absolute;
	left: 0;
	top: 0;
	width: 18px;
	height: 18px;
	border: 2px solid #000;
	border-radius: 0;
	background-color: white;
	transition: background-color 0.3s, transform 0.2s;
}

.custom-checkbox input:checked ~ label:before {
	background-color: #ffffff00;
	border-color: #000000;
}

.custom-checkbox label:after {
	content: "";
	position: absolute;
	left: 6px;
	top: 2px;
	width: 5px;
	height: 10px;
	border: solid rgb(0, 0, 0);
	border-width: 0 2px 2px 0;
	transform: rotate(45deg);
	opacity: 0;
	transition: opacity 0.3s ease;
}

.custom-checkbox input:checked ~ label:after {
	opacity: 1;
}

.custom-checkbox input:checked ~ label:before {
	transform: scale(1.1);
}

.custom-checkbox input:hover ~ label:before {
	border-color: #555;
}

.checkboxes {
	display: flex;
}

.checkboxes input {
	margin-right: 5px;
}

.checkboxes label {
	margin-right: 20px;
}

.full {
	width: 100%;
}

form button {
	font-family: cursive;
	font-size: small;
	margin-top: 15px;
	padding: 5px;
	border: 1px solid black;
	background-color: rgb(210, 200, 200);
}

input {
	font-family: cursive;
	font-size: small;
}

.button {
	width: 100%;
	display: flex;
	justify-content: flex-end;
}
</style>
