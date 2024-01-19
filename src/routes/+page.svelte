<script>
	import * as RNBO from '@rnbo/js'
	import ProgressBar from './ProgressBar.svelte';

	let loaded = false
	let context = null
	let device = null
	let isRecording = false

	let progress = 0
	let totalTime = 0

	$: {
		if (device) {
			const parameter = device.parametersById.get('record')
			parameter.value = isRecording
		}
	}

	const createDeviceInstance = (devicePath, context, output) => {
		return new Promise((resolve, reject) => {
			fetch(devicePath)
				.then((response) => response.json())
				.then((response) => {
					const patcher = response;
					return RNBO.createDevice({ context, patcher });
				})
				.then((device) => {
					device.node.connect(output);
					device.messageEvent.subscribe((e) => {
						const { tag, payload } = e;
						const lookup = {
							'progress': (value) => { progress = value},
							'total_time': (value) => { totalTime = value}
						}
						try {
							lookup[tag](payload)
						} catch (e) {
						}
					});
					if (device) {
						resolve(device);
						reject('Error');
					}
				});
		});
	};

	async function start() {
		context = new (window.AudioContext || window.webkitAudioContext)();

		let output = context.createGain().connect(context.destination);
		createDeviceInstance('/jamoslooper.json', context, output)
		.then(async(response) => {
				device = response;
				loaded = true

				const stream = await navigator.mediaDevices.getUserMedia({
					audio: true,
				});

				const source = context.createMediaStreamSource(stream);
				source.connect(device.node)
			}
		);
	}
</script>

{#if !loaded}
<button on:click={start}>start</button>
{:else}
<div class='container'>

	<label for="">
		Record
		<input type="checkbox" bind:checked={isRecording}>
	</label>
	
	<ProgressBar {progress} {totalTime}/>
</div>
{/if}

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
</style>