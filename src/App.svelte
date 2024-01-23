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
let latitude = "";
let longitude = "";
let email = "";
let phoneNumber = "";
let postCode = "";
let city = "";
let website = ""
let json = "";
let coordinatesChecked = true;
$: producer = {
	name,
	firstname,
	lastname,
	categories,
	address,
	latitude,
	longitude,
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
	latitude = producer.latitude;
	longitude = producer.longitude;
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

	function getExpectedCoordinates(form) {
		var address = (form.address.value+" "+form.postCode.value+" "+form.city.value).trim();
		return new Promise((resolve) => {
			getCoordinateFromAddress(address, 
				(x, y) => { resolve([x, y]); }
			);
	    });
	}
	async function checkAddress(form) {
		if (!coordinatesChecked) {
			console.log("check coordinates");
			const coordinates = await getExpectedCoordinates(form);
			let x = coordinates[0], y = coordinates[1];
			let lat = form.latitude.value, long = form.longitude.value;
			console.log(coordinates," VS ",lat,long);
			if (lat>x+0.1 || lat<x-0.1 || long>y+0.1 || long<y-0.1) {
				displayError("Expected coordinates are [ "+x+" , "+y+" ]");
				return false;
			} else {
				coordinatesChecked = true;
			}
		} else {
			console.log("coordinates ever checked");
		}
		return true;
	}
	function checkInputs(form) {
		let invalid = form.querySelectorAll('input:invalid');
		if (invalid.length != 0) {
			displayError(errMessageDefault);
			return false;
		}
		return true;
	}
	function checkForm() {
		let form = document.getElementById('producerEditForm');
		hasError = false;
		return checkInputs(form) && checkAddress(form);
	}
	function displayError(err) {
		hasError = true;
		errMessage = err;
		setTimeout(function(){
			hasError = false;
		}, 4000);
	}
	function handleSubmit(e) {
		if (checkForm()) {
			const url = "/producers/save/"+producer_id;
			request.open("POST", url);
			producer.name = firstname;
			producer.firstname = firstname;
			producer.lastname = lastname;
			producer.categories = categories;
			producer.address = address;
			producer.latitude = latitude;
			producer.longitude = longitude;
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
					displayError(request.response.error);
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
	let latlongRegexp = /^[0-9]{1,2}\.[0-9]{6,15}$/
</script>

<svelte:head>
	<script src="/js/openproduct.js" ></script>
</svelte:head>

<div class="container">
	<form id="producerEditForm" class="mt-4" class:submitted on:submit|preventDefault={handleSubmit}>
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
			<Input label="Website" name="website" bind:value={website} regexp={websiteRegexp} errorMsg="This is not a valid web Url." />
		</div>

		<div class="form-group">
			<h3>Address</h3>
			<Input label="Latitude" name="latitude" bind:value={latitude} regexp={latlongRegexp} errorMsg="Must be a number with a precision of minimum 6 digit" on:change={e => coordinatesChecked=false} />
			<Input label="Longitude" name="longitude" bind:value={longitude} regexp={latlongRegexp} errorMsg="Must be a number with a precision of minimum 6 digit" on:change={e => coordinatesChecked=false} />
			<Input label="Address" name="address" bind:value={address} regexp={addressRegexp} errorMsg="Contains invalid characters." on:change={e => coordinatesChecked=false} />
			<Input label="Post code" name="postCode" bind:value={postCode} regexp={postCodeRegexp} errorMsg="Must have 5 digit. Use 0 if needed." on:change={e => coordinatesChecked=false} />
			<Input label="City" name="city" bind:value={city} regexp={stringRegexp} errorMsg="Contains invalid characters." on:change={e => coordinatesChecked=false} />
		</div>

{#if hasError == true}
		<p class="error-alert">{errMessage}</p>
{:else}
	{#if isSuccessVisible}
		<p class="sucess-alert" transition:fade={{duration:150}}>Data updated successfully</p>
	{/if}
{/if}
		<button class="btn btn-submit" on:click={() => submitted = true}>Continue</button>
	</form>
</div>

<p>Producer : {JSON.stringify(producer)}</p>

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
