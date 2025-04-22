<script lang="ts">
	import Dropzone from 'svelte-file-dropzone';
	let files = $state<FileList | null>(null);
	let apiKey = $state<string>('');
	let transcription = $state<string>('');
	let loading = $state<boolean>(false);
	let error = $state<string>('');

	async function handleUpload() {
		if (!files || files.length === 0 || !apiKey) {
			error = 'Please select a file and enter an API key.';
			return;
		}

		const file = files[0];
		if (!file) {
			error = 'No file found.';
			return;
		}

		error = '';
		transcription = '';
		loading = true;

		const formData = new FormData();
		formData.append('file', file);

		try {
			const res = await fetch('https://informed-collie-previously.ngrok-free.app/transcribe', {
				method: 'POST',
				headers: {
					'x-api-key': apiKey
				},
				body: formData
			});

			if (!res.ok) {
				throw new Error(`Server responded with status ${res.status}`);
			}

			const data = await res.json();
			transcription = data.transcription;
		} catch (err: any) {
			error = err.message;
		} finally {
			loading = false;
		}
	}
</script>

<main>
	<h1 style="color: white;">🎧 Upload and Transcribe</h1>

	<input type="text" placeholder="API Key" bind:value={apiKey} />
	<input style="color: white;" type="file" accept="audio/*" bind:files />

	<button onclick={handleUpload} disabled={loading} class="myButton">
		{loading ? 'Transcribing...' : 'Transcribe Audio'}
	</button>

	{#if error}
		<p class="error">{error}</p>
	{/if}

	{#if transcription}
		<div class="result">
			<h2>Transcription:</h2>
			<p>{transcription}</p>
		</div>
	{/if}
</main>

<style>
	main {
		/* max-width: 600px; */
		margin: 0;
		padding: 2rem;
		height: 100vh;
		font-family: system-ui, sans-serif;
		background-color: #1f2f47;
	}

	input,
	button {
		margin-bottom: 1rem;
		display: block;
	}

	.result {
		margin-top: 1rem;
		background: #f9f9f9;
		padding: 1rem;
		border-radius: 6px;
	}

	.error {
		color: red;
	}

	.myButton {
		box-shadow: inset 0px 0px 15px 2px #23395e;
		background: linear-gradient(to bottom, #2e466e 5%, #415989 100%);
		background-color: #2e466e;
		border-radius: 15px;
		border: 3px solid #1f2f47;
		display: inline-block;
		cursor: pointer;
		color: #ffffff;
		font-family: Arial;
		font-size: 15px;
		padding: 20px 56px;
		text-decoration: none;
		text-shadow: 0px 6px 34px #263666;
	}
	.myButton:hover {
		background: linear-gradient(to bottom, #415989 5%, #2e466e 100%);
		background-color: #415989;
	}
	.myButton:active {
		position: relative;
		top: 1px;
	}
</style>
