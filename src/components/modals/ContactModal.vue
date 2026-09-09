<template>
	<div class="pilot-modal">
		<div class="pilot-header-container">
			<div class="section-header clipped-medium-backward-bio">
				<img src="/icons/license.svg" />
				<h1>{{ contact.name }} [{{ contact.alias }}]</h1>
			</div>
			<div class="rhombus-back">&nbsp;</div>
		</div>
		<div class="pilot markdown">
			<div v-html="getHistory()" />
		</div>
	</div>
	<div class="pilot-modal portrait">
		<div class="pilot-header-container">
			<div class="section-header clipped-medium-backward-pilot">
				<img src="/icons/portrait.svg" />
				<h1>Contact Artwork</h1>
			</div>
			<div class="rhombus-back">&nbsp;</div>
		</div>
		<div class="pilot">
			<img :src="contactPortrait" class="portrait" @error="portraitFailed = true" />
		</div>
	</div>
</template>

<script>
export default {
	inheritAttrs: false,
	props: {
		contact: {
			type: Object,
			required: true,
		},
	},
	data() {
		return {
			portraitFailed: false,
		};
	},
	computed: {
		contactPortrait() {
			if (this.portraitFailed) return this.contact.img?.cloud_portrait || "";
			return `/contacts/${this.contact.name.toUpperCase()}.webp`;
		},
	},
	methods: {
		// narrative.textItems (Appearance / Personality) render on the card itself now;
		// the modal carries only the overview so the two don't duplicate each other.
		getHistory() {
			if (!this.contact.description) return `<h2> [ERR: REDACTED] </h2>`;

			return `<h2>OVERVIEW</h2> ${this.contact.description}`;
		},
	},
};
</script>
