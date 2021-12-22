<script>
	import * as Tone from 'tone'
	import { Midi } from '@tonejs/midi'

	let input
	var currentScreen = 0

	var midiTracks = []
	var midiNotes = []
	var midiDuration = []
	var currentNote = ""

	async function getMidi(url) {
		return await Midi.fromUrl(url)
	}

	function uploadedFile() {
		currentScreen++
		getBase64(input.files[0]).then(result => {
			getMidi(result)
				.then(midi => {
					console.log(midi)
					midi.tracks.forEach(track => {
						midiTracks.push(track)
					})
					currentScreen++
				})
		})
	}

	async function getBase64(audioFile) {
		return new Promise((resolve, reject) => {
			let reader = new FileReader();
			reader.onerror = reject;
			reader.onload = (e) => resolve(e.target.result);
			reader.readAsDataURL(audioFile);
		});
	}

	async function selectTrack(index) {
		currentScreen++
		midiTracks[index].notes.forEach(note => {
			midiNotes.push(note.name)
			midiDuration.push(note.duration)
		})
		const synth = new Tone.Synth().toDestination()
		currentScreen++
		for(let i = 0; i < midiNotes.length; i++) {
			console.log(noteConverter(midiNotes[i]))
			synth.triggerAttackRelease(midiNotes[i], midiDuration[i] * 1000)
			currentNote = midiNotes[i]
			await new Promise(resolve => setTimeout(resolve, midiDuration[i] * 1000)); // 3 sec
		}
	}

	function noteConverter(note) {
		return note
			.replace("A", "La ")
			.replace("B", "Si ")
			.replace("C", "Do ")
			.replace("D", "Re ")
			.replace("E", "Mi ")
			.replace("F", "Fa ")
			.replace("G", "Sol ")
	}
</script>

{#if currentScreen == 0}
	<input type="file" on:change={uploadedFile} bind:this={input}>
{/if}

{#if currentScreen == 1 || currentScreen == 3}
	<p>Loading...</p>
{/if}

{#if currentScreen == 2}
	{#each midiTracks as track, index}
		{#if track.instrument.number > 72 && track.instrument.number < 81}
			<button style="color: green;" on:click={() => selectTrack(index)}>{track.name + ", " + (index + 1)}</button>
		{:else}
			<button on:click={() => selectTrack(index)}>{track.name + ", " + (index + 1)}</button>
		{/if}
	{/each}
{/if}

{#if currentScreen == 4}
	<p>{currentNote}</p>
	<p>{noteConverter(currentNote)}</p>
{/if}

