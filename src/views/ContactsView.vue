<template>
	<div
		class="section-content-container"
		:class="{ animate: animateView }"
		:style="{ 'animation-delay': animationDelay }"
		id="contacts"
	>
		<div class="contact-list-container">
			<Contact v-for="item in contacts" :key="item.id" :contact="item" :animate="animate" />
		</div>
	</div>
</template>

<script>
import { VueMarkdownIt } from "@f3ve/vue-markdown-it";
import Contact from "@/components/Contact.vue";

export default {
	components: {
		VueMarkdownIt,
		Contact,
	},
	props: {
		animate: {
			type: Boolean,
			required: true,
		},
		contacts: {
			type: Array,
			required: true,
		},
	},
	data() {
		return {
			animateView: this.animate,
			animationDelay: "1.75s",
			clockAnimationDelay: "2500",
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
	},
};
</script>

<style scoped>
.contact-list-container {
	display: flex;
	flex-wrap: wrap;
	padding: 1em;
	gap: 1em;
}

.grid-item {
	flex: 45%;
}

.contact-list-container {
	height: calc(100vh - 96px);
}

.section-content-container {
	border: 0px solid transparent;
}

section.section-container#contacts {
	display: flex;
	flex-direction: column;
	margin: 50px 30px;
	width: 1755px;
}
</style>
