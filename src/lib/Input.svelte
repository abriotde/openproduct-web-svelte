<script>
	export let label = "";
	export let name = "";
	export let value = "";
	export let regexp = /.*/;
	export let errorMsg = "Invalid value.";
	export let validator = (val) => {
		// console.log("Regexp:",regexp);
		return regexp.test(val);
	};
	let error = "";
	function handleInput({ target: t }) {
		if (validator(t.value)) {
			value = t.value;
			t.setCustomValidity("");
		} else {
			error = "";
			t.setCustomValidity(errorMsg);
		}
	}
</script>
<div class="form-unit">
	{#if error}
		<span class="block error-text">{error}</span>
	{/if}
	<label for="{name}">{label} : </label>
	<input type="text" name="{name}" class="form-control" placeholder="{label}" bind:value={value} on:input={handleInput} on:input on:blur />
</div>

<style>
	.form-control {
		border-radius: 3px;
		vertical-align: baseline;
	}
	input:invalid {
		border: 1px solid #c00;
	}
	input:focus:invalid {
		outline: 1px solid #c00;
	}
	.error-text {
		border: 1px solid #c00 !important;
		color: #c00;
	}
</style>