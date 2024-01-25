<script>
	export let label = "";
	export let name = "";
	export let value = "";
	let error = "";
	function handleInput({ target: t }) {
	}
</script>
<div class="form-unit">
	{#if error}
		<span class="block error-text">{error}</span>
	{/if}
	<label for="{name}">{label} : </label>
	<textarea name="{name}" class="form-control" placeholder="{label}" rows="5" cols="33" bind:value={value} on:input={handleInput} on:input on:blur />
</div>

