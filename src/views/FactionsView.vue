<template>
	<div
		id="factionsView"
		:class="{ animate: animateView }"
		:style="{ 'animation-delay': animationDelay }"
		class="content-container"
	>
		<section id="factions" :class="{ animate: animate }" class="section-container">
			<div class="section-header clipped-medium-backward">
				<img src="/icons/squad.svg" />
				<h1>FACTIONS</h1>
			</div>
			<div class="section-content-container">
				<div class="factions-list-container">
					<Faction
						v-for="item in factions"
						:key="item.title"
						:faction="item"
						:animate="animate"
						@select-faction="selectFaction(item)"
					/>
				</div>
			</div>
		</section>
		<section id="dossiers" :class="{ animate: animate }" class="section-container">
			<div style="height: 52px; overflow: hidden">
				<div class="section-header clipped-medium-backward-dossiers">
					<img src="/icons/orbital.svg" />
					<h1>DOSSIER</h1>
				</div>
				<div class="rhombus-back">&nbsp;</div>
			</div>
			<div class="section-content-container extra-margins">
				<div class="faction" v-if="selectedFaction.title">
					<div class="name">
						<h1>{{ selectedFaction.location }} // {{ selectedFaction.disposition }}</h1>
						<h2>{{ selectedFaction.title }}</h2>
					</div>
					<vue-markdown-it :source="selectedFaction.content" class="markdown" />
				</div>
			</div>
		</section>
	</div>
</template>

<script>
import { VueMarkdownIt } from "@f3ve/vue-markdown-it";
import Faction from "@/components/Faction.vue";

export default {
	components: {
		VueMarkdownIt,
		Faction,
	},
	props: {
		animate: {
			type: Boolean,
			required: true,
		},
		factions: {
			type: Array,
			required: true,
		},
	},
	data() {
		return {
			animateView: this.animate,
			animationDelay: "1.75s",
			selectedFaction: {
				type: Object,
			},
		};
	},
	created() {
		this.setAnimate();
	},
	methods: {
		setAnimate() {
			if (this.animate) {
				this.animateView = true;
			}
			let statusAnimated = window.sessionStorage.getItem("statusAnimated");
			if (statusAnimated) {
				this.animationDelay = "0s";
			}
			if (statusAnimated === null) {
				window.sessionStorage.setItem("statusAnimated", true);
			}
		},
		selectFaction(faction) {
			this.selectedFaction = faction;
		},
	},
};
</script>
