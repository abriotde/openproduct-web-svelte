<script>
import Input from './lib/Input.svelte';
import Categories from './lib/Categories.svelte';
import {fly, fade } from 'svelte/transition';
// import { writable } from 'svelte/store';


const urlParams = new URLSearchParams(window.location.search);
const producer_id = urlParams.get('p');

let name = "";
let firstname = "";
let lastname = "";
let categories = "";
let address = "";
let email = "";
let phoneNumber = "";
let postCode = "";
let city = "";
let website = ""
let json = "";
$: producer = {
	name,
	firstname,
	lastname,
	categories,
	address,
	email,
	phoneNumber,
	postCode,
	city,
	website,
	json
};
function loadProducer(producer) {
	name = producer.name;
	firstname = producer.firstname;
	lastname = producer.lastname;
	categories = producer.categories;
	address = producer.address;
	email = producer.email;
	phoneNumber = producer.phoneNumber;
	postCode = producer.postCode;
	city = producer.city;
	website = producer.website;
}
const request = new XMLHttpRequest();
try {
	const url = "/producers/get/"+producer_id;
	request.open("GET", url);
	request.responseType = "json";
	request.onload = function() {
		producer = request.response;
		console.log("Response: ",producer);
		loadProducer(producer);
    }
	request.onerror = function () {
		console.error("Erreur XHR");
	}
	console.log("Get:",url);
	request.send();
} catch (error) {
	console.error(`Erreur XHR ${request.status}`);
}

	let hasError = false;
	let isSuccessVisible = false;
	let submitted = false;
	
	const errMessageDefault = "All the fields are mandatory";
	let errMessage = errMessageDefault;
	
	function handleSubmit(e) {
		let surveyForm = document.getElementById('producerEditForm');
		let invalid = surveyForm.querySelectorAll('input:invalid');
		hasError = invalid.length != 0;
		if (!hasError) {
			const url = "/producers/save/"+producer_id;
			request.open("POST", url);
			producer.name = firstname;
			producer.firstname = firstname;
			producer.lastname = lastname;
			producer.categories = categories;
			producer.address = address;
			producer.email = email;
			producer.phoneNumber = phoneNumber;
			producer.postCode = postCode;
			producer.city = city;
			producer.website = website;
			producer.json = "1";
			request.responseType = "json";
			request.onload = function() {
				console.log(request.response);
				if (request.response && request.response.ok) {
					isSuccessVisible = true;
					loadProducer(request.response.vals);
					setTimeout(function(){
						isSuccessVisible = false;
					}, 4000);
				} else {
					hasError = true;
					errMessage = request.response.error;
					setTimeout(function(){
						hasError = false;
						errMessage = errMessageDefault;
					}, 4000);
				}
			}
			request.onerror = function () {
				console.error("Erreur XHR");
			}
			console.log("POST:", url, producer);
			request.send(JSON.stringify(producer));
		}
	}
	let emailRegexp = /^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/
	let phoneRegexp = /^\+?[0-9 ]{5,15}$/
	let postCodeRegexp = /^[0-9]{5}$/
	let stringRegexp = /^[a-zA-Zéèêàù -]{2,64}$/
	let addressRegexp = /^[a-zA-Z0-9éèêàù -]{2,64}$/
	let websiteRegexp = /^https?:\/\/[a-zA-Z0-9.]{2,64}$/
</script>


{#if hasError == true}
		<p class="error-alert">{errMessage}</p>
{:else}
	{#if isSuccessVisible}
		<p class="sucess-alert" transition:fade={{duration:150}}>Data updated successfully</p>
	{/if}
{/if}
<p>Producer : {JSON.stringify(producer)}</p>
<div class="container">
	<form id="producerEditForm" class="mt-4" class:submitted on:submit|preventDefault={handleSubmit}>
		// https://svelte.dev/repl/c399ebf9700b4b25bf6fce7d0b3d4ffd?version=4.2.9
		<h2>Edit producer</h2>
		<div class="form-group">
			<h3>Identité</h3>
			<Input label="Company name" name="name" bind:value={name} />
			<Input label="First name" name="firstname" bind:value={firstname} />
			<Input label="Last name" name="lastname" bind:value={lastname} />
			<Categories label="Categories" name="categories" bind:value={categories} />
		</div>

		<div class="form-group">
			<h3>Coordonnées</h3>
			<Input label="Phone number" name="phoneNumber" bind:value={phoneNumber} regexp={phoneRegexp} errorMsg="Must contain only digit and space (And eventually a + at begening)" />
			<Input label="Email" name="email" bind:value={email} regexp={emailRegexp} errorMsg="Must have an @." />
			<Input label="Address" name="address" bind:value={address} regexp={addressRegexp} errorMsg="Contains invalid characters." />
			<Input label="Post code" name="postCode" bind:value={postCode} regexp={postCodeRegexp} errorMsg="Must have 5 digit. Use 0 if needed." />
			<Input label="City" name="city" bind:value={city} regexp={stringRegexp} errorMsg="Contains invalid characters." />
			<Input label="Website" name="website" bind:value={website} regexp={websiteRegexp} errorMsg="This is not a valid web Url." />
		</div>

		<button class="btn btn-submit" on:click={() => submitted = true}>Continue</button>
	</form>
</div>


<style>
	#producerEditForm {
		max-width: 1200px;
		margin: 0 auto;
		text-align: right;
		background: #00000012;
	}
	.form-group {
		margin: 1em;
		background: #00000034;
		border-radius: 1em;
	}
	h2 {
		margin-top: 0;
	}
	.form-group h3 {
		text-align:left;
	}
	.btn-submit {
		margin: 1em;
		border-radius: 3px;
		vertical-align: baseline;
	}

	.error-alert,
	.sucess-alert {
		padding: 6px;
		text-align: center;
		border-radius: 3px;
	}
	.error-alert {
		border: 1px solid #c00 !important;
		color: #c00;
	}
	.sucess-alert {
		border: 1px solid #060 !important;
		color: #060;
		padding: 6px;
		text-align: center;
		border-radius: 3px;
	}
	:global(.form-control) {
		border-radius: 3px;
		vertical-align: baseline;
	}
	:global(.error-text) {
		border: 1px solid #c00 !important;
		color: #c00;
	}
</style>
