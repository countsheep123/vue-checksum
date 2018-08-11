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
					<td>{{result.checksum}}
						<span class="copy">
							<font-awesome-icon :icon="['far', 'copy']" @click="copy(result.checksum)" />
						</span>
					</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<style scoped>
.generator {
	margin-top: 1rem;
}
.copy {
	color: #cccccc;
	font-size: 0.75rem;
	cursor: pointer;
	float: right;
}
.copy:hover {
	color: #e8eaeb;
}
.copy:active {
	color: #00ffff;
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
import fontawesome from '@fortawesome/fontawesome';
import FontAwesomeIcon from '@fortawesome/vue-fontawesome';
import fabrands from '@fortawesome/fontawesome-free-brands';
import fasolid from '@fortawesome/fontawesome-free-solid';
import faregular from '@fortawesome/fontawesome-free-regular';

fontawesome.library.add(fabrands, fasolid, faregular);

export default {
	components: {
		FontAwesomeIcon
	},
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
		},
		copy(text) {
			var tmp = document.createElement('div');

			tmp.appendChild(document.createElement('pre')).textContent = text;

			var s = tmp.style;
			s.position = 'fixed';
			s.left = '-100%';

			document.body.appendChild(tmp);
			document.getSelection().selectAllChildren(tmp);

			var result = document.execCommand('copy');

			document.body.removeChild(tmp);

			// true: success, false: fail
			return result;
		},
	}
}
</script>
