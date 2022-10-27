<svelte:options tag="njv-scroll" />
<script>
	import {onMount} from "svelte"

	let listFirstEvents;
	let pagination = {};
	let ovserbables = [];
	let root ;


let tipoDeEvento = 'EX8bnCFDag94CDkYjtqc';
let orderBy = 'titulo';
let limit = 5;
let ultimoEvento;
let lastEventSelcetd;

	const loadindFirstEvents = async () => {
		try{
			const firestEvents = await fetch(`https://us-central1-appgoldeoro.cloudfunctions.net/api/eventos/tipo/${tipoDeEvento}/ordernar-por/${orderBy}/limite/${limit}`,{
			method: 'GET',
			headers: {
				"Content-Type": "application/json"
			}
		})
			const resultFirestEvents = await firestEvents.json();
	
		if(resultFirestEvents.status != 200){
		throw Error('Error ', resultFirestEvents.status)
		}
		// console.log(resultFirestEvents)
		pagination = resultFirestEvents.paginacion;
		listFirstEvents = resultFirestEvents.data;
		ultimoEvento = listFirstEvents[listFirstEvents.length-1].id;
		}catch(error){
			console.error(error)
		}
	}

	const cargarEventos = async () => {

		try{
	
				if(!ultimoEvento.length) return;
				const firestEvents = await fetch(`https://us-central1-appgoldeoro.cloudfunctions.net/api/eventos/tipo/${tipoDeEvento}/ordernar-por/${orderBy}/limite/${limit}/pagina/${ultimoEvento}`,{
					method: 'GET',
					headers: {
						"Content-Type": "application/json"
					}
				})
				const resultFirestEvents = await firestEvents.json();
			
				if(resultFirestEvents.status != 200){
						throw Error('Error ', resultFirestEvents.status)
				}
				// console.log(resultFirestEvents)
				pagination = resultFirestEvents.paginacion;
				listFirstEvents = [...listFirstEvents, ...resultFirestEvents.data];
				ultimoEvento = listFirstEvents[listFirstEvents.length-1].id;

				}catch(error){
					console.error(error)
				}
		
	}
	$: {
	
		if(ovserbables.length){
			
			requestIdleCallback(() => {
				const observer = new IntersectionObserver((entries, rootObserver) => {
				    console.log(entries)
						if(entries[0].isIntersecting){
										cargarEventos()
										observer.disconnect()
									
							}
				},{
				root,
				rootMargin: '0px 0px 700px 0px',
				threshold: 1
			});
		
			observer.observe(ovserbables[ovserbables.length-1]);
			
			},{timeout: 1000});	
		}
	}
	onMount(() => {
		loadindFirstEvents();
	})
</script>


{#if listFirstEvents && listFirstEvents.length > 0}
<section bind:this={root} class="list-events">
	{#each listFirstEvents as event,id}
	<article bind:this={ovserbables[id]} class="event">
		<h2>{event.titulo}</h2>
	</article>
	{/each}
</section>
{/if}
<style>
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	.list-events {
		background-color: gray;
		width: 50vw;
		height: 50vh;
		margin: 20px auto 20px auto;
		display: grid;
		grid-auto-flow: row;
		grid-auto-rows: 50vh;
		overflow: hidden scroll;

	}

	.event {
		background-color: white;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 1em;
		margin: 1em;
	}
</style>