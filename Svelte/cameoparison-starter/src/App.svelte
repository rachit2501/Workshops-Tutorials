<script>
	import Welcome from './screens/Welcome.svelte';
	import Game from './screens/Game.svelte';
	import {onMount} from 'svelte';
	import {select} from './select.js';
	import {loadImage} from './utils.js';

	let state = 'welcome';
	let celebsPromise;
	let selection;

	const start = async (e)=>{
		const {celebs, lookup} = await celebsPromise;
		selection = select(celebs,lookup,e.detail.category.slug)
		state="playing"
	}

	const loadCelebs = async ()=>{
		const res = await fetch('https://cameo-explorer.netlify.app/celebs.json');
		const data = await res.json();
		const lookup = new Map();

		data.forEach(element => {
			lookup.set(element.id, element);			
		});

		const subset = new Set();

		data.forEach(c=>{
			if(c.reviews >= 50){
				subset.add(c);
				c.similar.forEach(id=>{
					subset.add(lookup.get(id))
				})
			}
		})

		return {
			celebs : Array.from(subset),
			lookup
		}

	}

	onMount(()=>{
		celebsPromise = loadCelebs();

		loadImage('/icons/right.svg');
		loadImage('/icons/wrong.svg')
	})
</script>

<main>
	{#if state==='welcome'}
		<Welcome on:select={start}/>
	{:else if state==="playing"}
		<Game {selection}/>
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 800px;
		margin: 0 auto;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}
</style>