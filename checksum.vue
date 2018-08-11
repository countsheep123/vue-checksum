<template>
	<div class="generator">
		<label>
			<input type="file" @change="fileChanged" multiple hidden />
			<div class="button" :style="{backgroundColor: bgc}"  @dragenter="dragEnter" @dragleave="dragLeave" @dragover="dragover" @drop="drop">Click here or drop file here to generate checksum...</div>
		</label>

		<table v-if="results.length">
			<thead>
				<tr>
					<th>Filename</th>
					<th>Size</th>
					<th>MIME type</th>
					<th>Checksum</th>
				</tr>
			</thead>
			<tbody>
				<tr v-for="result in results">
					<td>{{result.filename}}</td>
					<td>{{result.size}} bytes</td>
					<td>{{result.type}}</td>
					<td>{{result.checksum}}</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<style scoped>
.generator {
	margin-top: 1rem;
}
.button {
	border: 0.4rem dashed #2d2d2d;
	cursor: pointer;
	height: 1rem;
	padding: 1rem;
	text-align: center;
}
</style>

<script>
import md5 from 'js-md5';

export default {
	data() {
		return {
			bgc: "#f3f5f7",
			results: [],
		}
	},
	methods: {
		calc: function(files) {
			if (!files) {
				return
			}

			Array.from(files).forEach((file, index) => {
				this.setResult(file, index);
			})
		},
		setResult: function(file, index) {
			var me = this;

			var reader = new FileReader();
			reader.onload = function(e) {
				var result = {
					filename: file.name,
					size: file.size,
					type: file.type,
					checksum: md5.base64(e.target.result),
				};

				me.results.splice(index, 0, result);
			}
			reader.readAsArrayBuffer(file);
		},
		changeBGColor: function(color) {
			this.bgc = color
		},
		dragover(e) {
			e.preventDefault();
		},
		dragEnter(e) {
			e.preventDefault();
			this.changeBGColor("#e8eaeb");
		},
		dragLeave(e) {
			e.preventDefault();
			this.changeBGColor("#f3f5f7");
		},
		drop(e) {
			e.preventDefault();
			this.changeBGColor("#f3f5f7");

			this.calc(e.dataTransfer.files);
		},
		fileChanged(e) {
			e.preventDefault();
			
			this.calc(e.target.files);
		}
	}
}
</script>
