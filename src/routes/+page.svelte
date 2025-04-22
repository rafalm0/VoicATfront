<script lang="ts">
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
	<div class="container">
		<h1>🎧 Upload and Transcribe</h1>

		<input type="text" placeholder="API Key" bind:value={apiKey} class="input" />

		<label for="file-upload" class="upload-label">🎤 Choose Audio File</label>
		<input id="file-upload" type="file" accept="audio/*" bind:files class="file-hidden" />

		{#if files}
			<p style="color: white;">Selected file: {files[0]?.name}</p>
		{/if}

		<button on:click={handleUpload} disabled={loading} class="myButton">
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
	</div>
</main>

<style>
	main {
		background-color: #1f2f47;
		height: 100vh;
		padding: 2rem;
		font-family: system-ui, sans-serif;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 100%;
		/* max-width: 500px; */
		background: #2a3b5c;
		padding: 2rem;
		border-radius: 1rem;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
	}

	h1 {
		color: white;
		margin-bottom: 1.5rem;
	}

	.input {
		width: 100%;
		padding: 0.75rem;
		margin-bottom: 1rem;
		border-radius: 0.5rem;
		border: none;
	}

	.upload-label {
		display: inline-block;
		background-color: #4f46e5;
		color: white;
		padding: 0.75rem 1.5rem;
		border-radius: 0.5rem;
		cursor: pointer;
		margin-bottom: 1rem;
		font-weight: bold;
		transition: background-color 0.2s;
	}

	.upload-label:hover {
		background-color: #4338ca;
	}

	.file-hidden {
		display: none;
	}

	.result {
		margin-top: 1rem;
		background: #f9f9f9;
		padding: 1rem;
		border-radius: 6px;
		width: 100%;
		color: #1f2f47;
	}

	.error {
		color: red;
		margin-top: 0.5rem;
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
		padding: 15px 40px;
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
