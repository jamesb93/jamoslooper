<script>
	export let progress = 0;
	export let totalTime = 100;
	let progressWidth = 0;
	$: progressWidth = (progress / totalTime) * 100;
</script>

<svg class="progress-bar" width="80%" height="20">
	<rect class="progress-bar__background" width="100%" height="100%" fill="lightgray" />
	<rect class="progress-bar__fill" width="{progressWidth}%" height="100%" fill="blue" />
</svg>

<style>
	.progress-bar__fill {
		/* transition: width 0.1s linear; */
	}
</style>