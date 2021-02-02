<script context="module">
  export async function load(r) {
    return {
      props: {
        vid: r.page.params.vid
      }
    }
  }
</script>

<script>
	import { onMount } from "svelte";
  export let vid;
	let audio, cover_image;
	let progress, max;

	onMount(() => {
		const url = "https://" +
				vid +
				"-focus-opensocial.googleusercontent.com/gadgets/proxy?container=none&url=https%3A%2F%2Fwww.youtube.com%2Fget_video_info%3Fvideo_id%3D" +
				vid;
		fetch(url).then((response) => {
			if (response.ok) {
				response.text().then((ytData) => {
					ytData = parse_str(ytData);
					console.log(ytData);
					const player_response = JSON.parse(ytData.player_response);
					console.log(player_response);
					const { thumbnails } = player_response.videoDetails.thumbnail;
					let getAdaptiveFormats = player_response.streamingData.adaptiveFormats;
					let findAudioInfo = getAdaptiveFormats.findIndex(obj => obj.audioQuality);
					
					// get the URL for the audio file
					let audioURL = getAdaptiveFormats[findAudioInfo].url;
					
					// update the <audio> element src
					audio.src = audioURL;
					cover_image.src = thumbnails[3].url;
					max = getAdaptiveFormats[findAudioInfo].approxDurationMs;
				});
			}
		});
	});

	function parse_str(str) {
		return str.split('&').reduce(function(params, param) {
			var paramSplit = param.split('=').map(function(value) {
				return decodeURIComponent(value.replace('+', ' '));
			});
			params[paramSplit[0]] = paramSplit[1];
			return params;
		}, {});
	}

	let isPaused = true;

	const playPause = () => {
		if (audio.paused) {
			audio.play();
		}
		else {
			audio.pause();
		}
		isPaused = !isPaused;
	};

	let currentTime = 0;

	const handleTimeUpdate = () => currentTime = audio.currentTime / audio.duration;

	const getElapsed = (ct) => {
		let currentTime = audio?.currentTime ?? 0;
		let m = (currentTime / 60).toFixed(0).padStart(2, "0");
		let s = (currentTime % 60).toFixed(0).padStart(2, "0");
		return `${m}:${s}`;
	};
	
	const getDuration = (ct) => {
		let dur = audio?.duration ?? 0;
		let m = (dur / 60).toFixed(0).padStart(2, "0");
		let s = (dur % 60).toFixed(0).padStart(2, "0");
		return `${m}:${s}`;
	};

	$: elapsed = getElapsed(currentTime);
	$: duration = getDuration(currentTime);
</script>

<div class="player">
	<audio 
		bind:this="{audio}"
		on:timeupdate="{handleTimeUpdate}"
	/>
	<div class="cover">
		<img bind:this="{cover_image}"/>
		<span>
			<span>{elapsed}</span>
			<progress max="1" value="{currentTime}"/>
			<span>{duration}</span>
		</span>
	</div>
	<footer>
		<div on:click="{playPause}" id="playPause">
			{#if isPaused}
				<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
					<path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM9.555 7.168A1 1 0 008 8v4a1 1 0 001.555.832l3-2a1 1 0 000-1.664l-3-2z" clip-rule="evenodd" />
				</svg>
			{:else}
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 9v6m4-6v6m7-3a9 9 0 11-18 0 9 9 0 0118 0z" />
				</svg>
			{/if}
		</div>
	</footer>
</div>

<style>
	.player {
		display: grid;
		grid-template-columns: 1fr;
		grid-template-rows: 2fr 1fr;
		place-items: center;
		width: 100%;
		border: 1px solid #242F40;
	}
	.cover {
		display: grid;
		place-items: center;
	}
	#playPause {
		width: 50px;
		height: 50px;
	}
</style>
