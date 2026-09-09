<template>
	<span class="type-it"></span>
</template>

<script>
import TypeIt from "typeit";

export default {
	name: "Typer",
	// Fires when the line has finished typing, so callers can chain a follow-on reveal.
	emits: ["typed"],
	props: {
		values: {
			type: Array,
			required: true,
		},
		speed: {
			type: Number,
			default: 10,
		},
		loop: {
			type: Boolean,
			default: false,
		},
		startDelay: {
			type: Number,
			default: 0,
		},
		// Matches the `animate` prop Clock and Burden take, sourced from Config.animate.
		animate: {
			type: Boolean,
			default: true,
		},
	},
	computed: {
		// Watching `values` directly would rebuild on every parent re-render, because a
		// caller writing :values="[pilotCode]" hands over a new array each time. Compare the
		// contents instead so a rebuild only happens when the text actually changes.
		valuesKey() {
			return this.values.join(" ");
		},
	},
	watch: {
		valuesKey: "rebuild",
	},
	mounted() {
		this.build();
	},
	beforeUnmount() {
		this.teardown();
	},
	methods: {
		build() {
			// Gated on Config.animate rather than prefers-reduced-motion. _reset.css
			// deliberately lets the boot sequence play even when reduced motion is set (the
			// sequence is this dashboard's presentation, not decoration), and checking the
			// media query here would contradict that — it disabled typing entirely on any
			// machine with OS animations turned off. Flip Config.animate to opt out.
			// Both shortcuts still emit `typed`: the line is finished, just instantly. Without
			// it anything chained to completion never fires — an empty field left its status
			// stamp hidden permanently.
			if (!this.animate) {
				this.$el.innerHTML = this.values[this.values.length - 1] ?? "";
				this.$emit("typed");
				return;
			}

			// Nothing to type: skip the instance entirely. Otherwise TypeIt mounts a caret
			// that never completes, so retireCaret() never fires and it blinks forever on a
			// field the contact simply left blank.
			if (!this.valuesKey.trim()) {
				this.$el.textContent = "";
				// Honour startDelay even with nothing to type, so a blank field's follow-on
				// reveal still lands in sequence instead of firing at mount, ahead of fields
				// that come before it.
				this.emptyTimer = setTimeout(() => this.$emit("typed"), this.startDelay);
				return;
			}

			// Deliberately not held in data(): Vue would wrap the instance in a reactive
			// proxy, and TypeIt mutates its own internals.
			this.instance = new TypeIt(this.$el, {
				strings: this.values,
				speed: this.speed,
				lifeLike: false,
				cursorChar: "▮",
				cursor: {
					autoPause: true,
					autoPauseDelay: 500,
					animation: {
						frames: [0, 0, 1].map(n => {
							return { opacity: n };
						}),
						options: {
							iterations: Infinity,
							easing: "steps(2, start)",
							fill: "forwards",
						},
					},
				},
				nextStringDelay: 5,
				startDelay: this.startDelay,
				startDelete: false,
				loop: this.loop,
				loopDelay: 10000,
				afterComplete: () =>
					requestAnimationFrame(() => {
						this.retireCaret();
						this.$emit("typed");
					}),
			}).go();
		},
		// The caret blinks via an Infinity-iteration Web Animation that would otherwise run
		// forever on every card. Cancelling it in place does not hold: `autoPause` suspends
		// the blink while characters are being typed and *resumes* it once the queue drains,
		// so a cancel fired from afterComplete gets immediately undone (measured: running
		// animations went 0 -> 5 the moment typing finished). Removing the element instead
		// is not something a resume can reverse. Deferred a frame so it lands after TypeIt's
		// own post-completion cursor handling.
		retireCaret() {
			const caret = this.$el?.querySelector(".ti-cursor");
			if (!caret) return;
			caret.getAnimations().forEach(animation => animation.cancel());
			caret.remove();
		},
		teardown() {
			// Without this the queue keeps running after the card unmounts; with loop
			// enabled that timer would never stop.
			if (this.emptyTimer) {
				clearTimeout(this.emptyTimer);
				this.emptyTimer = null;
			}
			if (this.instance) {
				// destroy(false): retireCaret() may already have removed the cursor node, and
				// TypeIt's removeNode() dereferences node.parentNode without a null check —
				// which threw during unmount and left the router unable to leave the view.
				this.instance.destroy(false);
				this.instance = null;
			}
			if (this.$el) this.$el.textContent = "";
		},
		rebuild() {
			this.teardown();
			this.build();
		},
	},
};
</script>
