<script>
	import {
		Button,
		Dialog,
		Grid,
		NumberStepper,
		Pagination,
		Stack,
		TextField,
		paginationStore
	} from 'svelte-ux'
	import { mdiMagnify, mdiArrowRight, mdiEyeOutline, mdiMovieOpen } from '@mdi/js'

	const apiKey = import.meta.env.VITE_API_KEY
	const pagination = paginationStore()

	let open = false
	let page = 1
	let episode = 1
	let season = 1
	let query = ''
	let imdbID = ''
	let type = ''
	let frameURL = ''
	let results = []
	let data = {}

	pagination.subscribe((value) => {
		if (value.page !== page) {
			search(value.page)
		}
	})

	const setFrameURL = () => {
		frameURL =
			type === 'series'
				? `https://www.2embed.stream/embed/tv/${imdbID}/${season}/${episode}`
				: `https://www.2embed.stream/embed/movie/${imdbID}`
	}

	const pressEnter = (e) => {
		if (['Enter', 'NumpadEnter'].includes(e.code)) {
			search()
		}
	}

	const viewIMDB = (movie) => {
		window.open(`https://www.imdb.com/title/${movie.imdbID}`, '_blank').focus()
	}

	const viewMovie = async (movie) => {
		imdbID = movie.imdbID
		type = movie.Type
		const response = await fetch(`https://omdbapi.com?apikey=${apiKey}&i=${movie.imdbID}`)
		data = await response.json()
		setFrameURL()
		open = true
	}

	const search = async (pageNumber = 1) => {
		if (!query) {
			return
		}

		page = pageNumber
		const response = await fetch(`https://omdbapi.com?apikey=${apiKey}&s=${query}&page=${page}`)
		const data = await response.json()
		if (data.Error) {
			results = []
			return
		}

		pagination.setPage(page)
		pagination.setTotal(data.totalResults)
		results = data.Search
	}
</script>

<svelte:head>
	<title>Svelte Movie</title>
</svelte:head>

<Dialog bind:open class="p-4">
	<h2 class="mb-4 text-3xl text-center">{data.Title}</h2>
	{#if type === 'series'}
		<div class="flex items-center justify-center mb-4">
			<span class="mr-4">Season:</span>
			<NumberStepper class="mr-4" mix={1} max={data.totalSeasons} bind:value={season} />
			<span class="mr-4">Episode:</span>
			<NumberStepper class="mr-4" min={1} bind:value={episode} />
			<Button icon={mdiArrowRight} class="flex-row-reverse p-2 text-blue" on:click={setFrameURL}
				>Go</Button
			>
		</div>
	{/if}
	<iframe width="1280" height="720" src={frameURL} title="Svelte Movie" allowfullscreen />
</Dialog>

<h1 class="text-4xl text-center">Svelte Movie</h1>

<TextField class="m-8" label="" on:keydown={pressEnter} bind:value={query}>
	<span slot="append">
		<Button icon={mdiMagnify} class="p-2 text-surface-content/50" on:click={search} />
	</span>
</TextField>

{#if results.length}
	<Grid autoColumns="240px" justifyItems="center">
		{#each results as item}
			<div class="relative my-4 bg-red-500 rounded shadow-lg w-60 h-80 group shadow-black">
				<div class="absolute w-full h-full one">
					{#if item.Poster !== 'N/A'}
						<img class="w-full h-full rounded" src={item.Poster} alt={item.Title} />
					{:else}
						<img
							class="w-full h-full rounded"
							src={`https://placehold.co/240x320/f4c932/000000?text=${item.Title}`}
							alt={item.Title}
						/>
					{/if}
				</div>
				<div
					class="box-border absolute w-full h-full px-3 text-white two bg-black/[.8] transition-transform delay-400 transform scale-0 group-hover:scale-100"
				>
					<Stack vertical template="auto 1fr auto auto" gap={8} class="h-full p-2">
						<h2 class="text-xl">{item.Title} ({item.Year})</h2>
						<div></div>
						<Button
							class="w-full text-white hover:text-gray-950 bg-fire-engine-red"
							variant="fill"
							icon={mdiEyeOutline}
							on:click={() => viewMovie(item)}
						>
							Watch
						</Button>
						<Button
							class="w-full text-white hover:text-gray-950 bg-fire-engine-red"
							variant="fill"
							icon={mdiMovieOpen}
							on:click={() => viewIMDB(item)}
						>
							IMDB
						</Button>
					</Stack>
				</div>
			</div>
		{/each}
	</Grid>
	<Pagination
		{pagination}
		show={['firstPage', 'prevPage', 'pagination', 'nextPage', 'lastPage']}
		classes={{ pagination: 'flex-1 text-center' }}
		format={(pagination) => `Page ${pagination.page} of ${pagination.totalPages}`}
	/>
{/if}
