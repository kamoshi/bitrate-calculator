<script lang="ts">
import { AudioBitrateUnit, FileSizeMode } from "./types";

	// inputs
	let inputFileSize = 8;
	let inputFileSizeUnit = FileSizeMode.MegaBytes;
    let inputVideoLength = '00:00:00';
	let inputAudioBitrate = 192;
	let inputAudioBitrateUnit = AudioBitrateUnit.KiloBits;

	$: [resultBytes, resultBits] = calculate(
		inputFileSize,
		inputFileSizeUnit,
		inputVideoLength,
		inputAudioBitrate,
		inputAudioBitrateUnit
	)
	
	function lengthAsNumber(value: string): number {
		if (!value)
			return 0;
		const [h, m, s] = value.split(':').map(x => parseInt(x));
		return ((h * 3600) + (m * 60) + s);
	}

	function getVideoModifier(value: FileSizeMode): number {
		switch (value) {
			case FileSizeMode.MegaBytes:
			return 1024;
			case FileSizeMode.KiloBytes:
			return 1;
		}
	}
		
	function getAudioModifier(value: AudioBitrateUnit, length: number): number {
		switch (value) {
			case AudioBitrateUnit.KiloBits:
			return 8;
			case AudioBitrateUnit.KiloBytes:
			return 1;
			case AudioBitrateUnit.TotalKB:
			return length;
			case AudioBitrateUnit.TotalMB:
			return length / 1024;
		}
	}

	function calculate(
		inputFileSize: number,
		inputFileSizeUnit: FileSizeMode,
		inputVideoLength: string,
		inputAudioBitrate: number,
		inputAudioBitrateUnit: AudioBitrateUnit,
	): [number, number] {
		const fileLength = lengthAsNumber(inputVideoLength);
		const fileSize = inputFileSize * getVideoModifier(inputFileSizeUnit);
		const audioCoefficient = inputAudioBitrate / getAudioModifier(inputAudioBitrateUnit, fileLength);
		const result = Math.max((fileSize / fileLength - audioCoefficient), 0);
		return [Math.floor(result), Math.floor(result * 8)];
	}
</script>


<div class="card">
	<h2>Video properties</h2>
	<form>
		<label for="inputFileSize">Final file size:</label>	
		<input id="inputFileSize" type="number" bind:value={inputFileSize}>
		
		<label for="inputFileSizeUnit">Filesize unit:</label>
		<select id="inputFileSizeUnit" bind:value={inputFileSizeUnit}>
			<option value={FileSizeMode.KiloBytes}>{FileSizeMode.KiloBytes}</option>
			<option value={FileSizeMode.MegaBytes}>{FileSizeMode.MegaBytes}</option>
		</select>
		
		
		<label for="inputVideoLength">Video length:</label>
		<input id="inputVideoLength" type="time" step="1" bind:value={inputVideoLength}/>
		
		<label for="inputAudioBitrate">Bitrate:</label>
		<input id="inputAudioBitrate" type="number" bind:value={inputAudioBitrate}>
		
		<label for="inputAudioBitrateUnit">Bitrate unit:</label>
		<select id="inputAudioBitrateUnit" bind:value={inputAudioBitrateUnit}>
			<option value={AudioBitrateUnit.KiloBits}>{AudioBitrateUnit.KiloBits}</option>
			<option value={AudioBitrateUnit.KiloBytes}>{AudioBitrateUnit.KiloBytes}</option>
		</select>
			
	</form>
</div>

<div class="card">
	<h2>Encoder target video bitrate</h2>
	<p>{resultBytes} kBytes/s</p>
	<p>{resultBits} kBits/s</p>
</div>


<style lang="scss">
	h2 {
		margin-top: 0;
	}

	form {
		display: grid;
		grid-template-columns: auto 1fr;
		grid-gap: 0.5em 1em;
		margin-right: auto;
	}

	label {
		display: flex;
  		justify-content: center;
  		align-items: center;
	}

	input,
	select {
		margin-bottom: 0;
	}

	.card {
		margin-bottom: 1em;
		padding: 1em;
		background-color: white;
		box-shadow: rgba(99, 99, 99, 0.2) 0px 2px 8px 0px;
	}
</style>