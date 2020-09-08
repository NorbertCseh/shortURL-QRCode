<template>
	<div id="app">
		<section>
			<div class="container">
				<div class="columns is-centered">
					<div class="column is-half">
						URL Shortener && QR Code generator
						<br />
						<div class="field is-horizontal">
							<input
								@change="errors = null"
								class="input"
								type="text"
								name="userURL"
								id="userURL"
								placeholder="https://www.google.com"
								v-model="oldUrl"
							/>

							<button
								class="button is-primary ml-4"
								v-on:click="sendRequest(oldUrl)"
								:disabled="loading"
							>
								<span v-if="loading">
									<progress
										class="progress is-small is-primary"
										max="100"
										>15%</progress
									>
								</span>
								<span v-else>Generate</span>
							</button>
						</div>
						<p v-if="errors" class="help is-danger mb-2">
							Error: {{ errors }}
						</p>
						<div class="field">
							<div v-if="newUrl" class="container mb-2">
								<span>New link:</span>
								<a :href="newUrl" target="_blank">
									{{ newUrl }}
								</a>
							</div>
						</div>
					</div>
				</div>
			</div>
		</section>

		<section>
			<div class="table-container">
				<table
					v-if="key !== 0"
					class="table container is-bordered is-striped is-hoverable is-narrow"
				>
					<thead>
						<th>Original Link</th>
						<th>Shortened Link</th>
						<th>QR Code</th>
					</thead>
					<tbody>
						<tr v-for="url in urls" v-bind:key="url[0]">
							<td>
								<a :href="url[1]" target="_blank">
									<span>{{ url[1] }}</span>
								</a>
							</td>
							<td>
								<a :href="url[2]" target="_blank">
									<span>{{ url[2] }}</span>
								</a>
							</td>
							<td><p v-html="url[3]"></p></td>
						</tr>
					</tbody>
				</table>
			</div>
		</section>
	</div>
</template>

<script lang="ts">
	import { Component, Emit, Prop, Vue } from 'vue-property-decorator';
	import axios from 'axios';
	import QRCode from 'qrcode-svg';

	@Component
	export default class App extends Vue {
		key = 0;
		oldUrl = '';
		loading = false;
		newUrl = '';
		urls: Array<[number, string, string, string]> = [];
		errors = '';

		data() {
			return {
				key: 0,
				oldUrl: '',
				loading: false,
				newUrl: '',
				errors: null,
			};
		}

		@Emit()
		async sendRequest(inputUrl: string) {
			this.loading = true;
			await axios
				.post('https://rel.ink/api/links/', {
					url: inputUrl,
				})
				.then((res) => {
					this.newUrl = `https://rel.ink/${res.data.hashid}`;
					this.urls.push([
						this.key,
						this.oldUrl,
						this.newUrl,
						new QRCode({
							content: this.oldUrl,
							predefined: true,
						}).svg(),
					]);
					this.key = this.key + 1;

					this.loading = false;
				})
				.catch(() => {
					this.errors = 'Not a valid URL address';
					this.loading = false;
				});
		}
	}
</script>

<style>
	#app {
		font-family: Share Tech Mono, Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
		margin-top: 60px;
	}
</style>
