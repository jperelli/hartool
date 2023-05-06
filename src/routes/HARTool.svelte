<script lang="ts">
	import type { Har } from 'har-format';

	let files: FileList;
	let har: Har;

	$: {
		if (files) {
			const reader = new FileReader();
			reader.onload = (e) => {
				har = JSON.parse(e.target?.result as string) as Har;
			};
			reader.readAsText(files[0]);
		}
	}

	// filter by url
	let filter = '';
	let filteredEntries: Har['log']['entries'];
	$: {
		filteredEntries = har?.log.entries.filter((entry) => entry.request.url.includes(filter)) || [];
	}

	// export filtered har to har
	const exportFilteredHAR = () => {
		const filteredHAR = { ...har, log: { ...har.log, entries: filteredEntries } };
		const blob = new Blob([JSON.stringify(filteredHAR)], {
			type: 'application/json'
		});
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = 'filtered.har';
		a.click();
		URL.revokeObjectURL(url);
	};
</script>

<section>
	<input id="upload-file" type="file" accept=".har" bind:files />
</section>

{#if har}
	<section>
		<input type="text" placeholder="filter by url" bind:value={filter} />
	</section>
	<section>
		<button on:click={exportFilteredHAR}>export filtered har</button>
	</section>
	<section>
		<h1>{har.log.pages?.[0].title}</h1>
	</section>
	<table>
		<thead>
			<tr>
				<th>status</th>
				<th>method</th>
				<th>url</th>
				<th>type</th>
				<th>size</th>
				<th>time</th>
				<th>startedAt</th>
			</tr>
		</thead>
		<tbody>
			{#each filteredEntries as entry}
				<tr>
					<td>{entry.response.status}</td>
					<td>{entry.request.method}</td>
					<td>{entry.request.url}</td>
					<td>{entry.response.content.mimeType}</td>
					<td>{entry.response.content.size}</td>
					<td>{entry.time}</td>
					<td>{entry.startedDateTime}</td>
				</tr>
			{/each}
		</tbody>
		<tfoot>
			<tr>
				<th>status</th>
				<th>method</th>
				<th>url</th>
				<th>type</th>
				<th>size</th>
				<th>time</th>
				<th>startedAt</th>
			</tr>
		</tfoot>
	</table>
{/if}

<style>
	td {
		max-width: 10rem;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
</style>
