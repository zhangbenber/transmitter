<template>
	<div class="spectrum">
		<div
			class="bar"
			v-for="(v, i) in spectrum"
			:key="i"
			:style="{height: `${Math.pow(Math.abs(v), 1) * 20}px`}"
		></div>
	</div>
</template>

<script>
import FFT from 'fft'

export default {
	data: () => ({
		spectrum: new Array(2048).fill(0)
	}),
	mounted () {
		let getUserMedia = navigator.getUserMedia || navigator.mozGetUserMedia
		getUserMedia.call(navigator, { audio: true }, stream => {
			let context = new AudioContext()
			let fft = new FFT.complex(1024)
			let window = (N =>
				Array(N).fill().map((ignore, n) => {
					// Blackman window function
					let M = Math
					return 0.42
						- 0.5 * M.cos(2 * M.PI * n / (N - 1))
						+ 0.08 * M.cos(4 * M.PI * n / (N - 1))
				})
			)(1024)
			let output = new Array(2048)
			let source = context.createMediaStreamSource(stream)
			let processor = context.createScriptProcessor(1024, 1, 1)
			processor.onaudioprocess = e => {
				let buffer = e.inputBuffer.getChannelData(0)
				let windowed = window.map((w, i) => w * buffer[i])
				fft.simple(output, windowed, 'real')
				this.$set(this, 'spectrum', output.slice(0, 1024))
			}
			source.connect(processor)
			processor.connect(context.destination)
		}, err => {
			console.log(err)
		})
	}
}
</script>

<style>
	.spectrum {
		line-height: 300px;
	}
	.bar {
		display: inline-block;
		width: 1px;
		vertical-align: bottom;
		background: #000;
		max-height: 300px;
	}
</style>
