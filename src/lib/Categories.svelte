<script>
	export let label = "";
	export let name = "";
	export let value = "";
	let existsCategories = [
		{"val":"H", "text":"Habillement", "subcategories":[
				{"val":"HI", "text":"Tisserand industriel"},
				{"val":"HC", "text":"Chappelier"},
				{"val":"HB", "text":"Bottier/Chaussure"},
				{"val":"HA", "text":"Autres accessoire (Canne, Pipe"},
				{"val":"HT", "text":"Artisants (Costumie, Couture"},
				{"val":"HD", "text":"Décoration (Dentelle, Broderie"}
			]
		},
		{"val":"A", "text":"Allimentaire", "subcategories":[
				{"val":"AA", "text":"Apiculteurs (Miel)"},
				{"val":"AV", "text":"Eleveurs (Viande, ou oeufs)"},
				{"val":"AC", "text":"Céréaliers (Céréales, pain, pomme de terre."},
				{"val":"AM", "text":"Maraichers"},
				{"val":"AL", "text":"Fromagers (Ou lait)"},
				{"val":"AP", "text":"Poisson<"},
				{"val":"AF", "text":"Fruit (Jus, Confitures)"},
				{"val":"AB", "text":"Boissons avec ou sans alcool"},
				{"val":"AG", "text":"Grossiste/Epicerie/Restaurant (Ne produit pas, mais vends du local)"},
				{"val":"AO", "text":"Autres (Champignons, boulangers/patissier/charcutier, produits de beauté, Spiruline, Huiles, safran)"}
			]
		},
		{"val":"P", "text":"Produits de beauté/plantes (Fleurs, arbres, savon, parfums"},
		{"val":"J", "text":"Jeux/Jouets"},
		{"val":"O", "text":"Outils (Couteaux, Céramiques, Meubles, et autres artistes", "subcategories":[
				{"val":"OP", "text":"Céramique/Potiers/Emailleur"},
				{"val":"OC", "text":"Couteliers/Armuriers"},
				{"val":"OI", "text":"Vittre/vitraux"},
				{"val":"OF", "text":"Métal (Bronzier, Forgerons, Ferronerie"},
				{"val":"OL", "text":"Luthier/Instruments de musique"},
				{"val":"OT", "text":"Cuir (Tanneur, Taxidermiste, Sellier"},
				{"val":"OR", "text":"Verre (Verrier"},
				{"val":"OO", "text":"Bois/Vannerie"},
				{"val":"OZ", "text":"Pierre (Sculpteur, Tailleur"},
				{"val":"OB", "text":"Bijouterie/Dorure/Horlogerie"},
				{"val":"OM", "text":"Métiers liés à l'architecture, Maison/Véhicules"},
				{"val":"OE", "text":"Autres (Tapis, Cire, Orfèvre"},
				{"val":"OD", "text":"Arts graphiques (Dessinateur/Enlumineur/Calligraphe/Graveur/Graphiste/Imprimeur/Relieur"}
			]
		}
	];
	let selected;
	let subselect;
	let subval = "";
	let subcategories = [];
	let error = "";
	const noneObject = {};
	function init() {
		console.log("init(",value,")");
		for (var categorie of existsCategories) {
			console.log("categorie",categorie);
			if (categorie.val==value) {
				selected = categorie;
				console.log("init(",value,") => ", selected); 
				return;
			}
			if (("subcategories" in categorie) && categorie.subcategories!=undefined && categorie.subcategories.length>0) {
				console.log("has subcategories");
				subval = categorie;
				for (var subCategorie of existsCategories) {
					console.log("subcategorie:",subCategorie);
					if (subCategorie.val==value) {
						selected = categorie;
						subcategories = categorie.subcategories;
						console.log("init(",value,") => ", selected); 
						return;
					}
				}
			}
		}
	}
	init();
	function displaySubCategories() {
		if (("val" in selected) && selected.val!=undefined) {
			value = selected.val;
		}
		if (("subcategories" in selected) && selected.subcategories!=undefined && selected.subcategories.length>0) {
			subcategories = selected.subcategories;
			subval = selected;
		} else {
			subcategories = [];
		}
		console.log("selected:",selected);
		console.log("subcategories:",subcategories);
	}
</script>
<div class="form-unit">
	{#if error}
		<span class="block error-text">{error}</span>
	{/if}
	<label for="{name}">{label} : </label>
	<select name="{name}" bind:value={selected} on:change={() => (displaySubCategories())} class="form-control" >
		<option value={noneObject}>None</option>
		{#each existsCategories as categorie}
			<option value={categorie}>
				{categorie.text}
			</option>
		{/each}
	</select>
	{#if subcategories.length>0}
		<select name="{name}_" bind:value={subselect} on:change={() => (value = subselect.val)} class="form-control" >
			<option value={subval}>None</option>
			{#each subcategories as categorie}
				<option value={categorie}>
					{categorie.text}
				</option>
			{/each}
		</select>
	{/if}
</div>
