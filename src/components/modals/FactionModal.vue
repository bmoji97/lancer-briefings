<template>
	<div class="faction-modal">
		<div class="faction-header-container">
			<div class="section-header clipped-medium-backward-bio">
				<img src="/icons/events.svg" />
				<h1>DOSSIER</h1>
			</div>
			<div class="rhombus-back">&nbsp;</div>
		</div>
		<div class="faction">
			<div class="name">
				<h1>{{ faction.location }} // {{ faction.disposition }}</h1>
				<h2>{{ faction.title }}</h2>
			</div>
			<vue-markdown-it :source="faction.content" class="markdown" />
		</div>
	</div>
</template>

<script>
import { VueMarkdownIt } from "@f3ve/vue-markdown-it";

export default {
	name: "FactionModal",
	components: {
		VueMarkdownIt,
	},
	props: {
		faction: {
			type: Object,
			required: true,
		},
	},
};
</script>
