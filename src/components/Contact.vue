<template>
	<div class="grid-item pilot-identity" style="color: white !important">
		<div class="header">
			<div class="col grow-max">
				<div class="heading h1">{{ contact.name }}</div>
				<div class="heading h2">({{ contact.alias }})</div>
			</div>
			<div class="col"><img src="/faction-logos/union.svg" /></div>
		</div>
		<div class="body">
			<div class="add-padding">
				<Typer :values="[identLine]" :speed="12" :start-delay="300" :animate="animate" />
			</div>
			<div class="flex-container-rows">
				<div class="row add-padding contact-code">
					<Typer :values="[contactCode]" :speed="12" :start-delay="900" :animate="animate" />
				</div>
				<div class="row flex-container-cols add-padding">
					<div class="col grow-max flex-container-rows" style="padding-top: 5px">
						<div class="row flex-container-cols">
							<div class="col col-primary">
								<span class="flavor-text">
									Alias:
									<b class="accent--text">
										<Typer
											:values="[contact.alias]"
											:speed="12"
											:start-delay="1800"
											:animate="animate"
											@typed="done.alias = true"
									/></b>
									<br />
									Name:
									<b class="accent--text">
										<Typer
											:values="[contact.name]"
											:speed="12"
											:start-delay="2000"
											:animate="animate"
											@typed="done.name = true"
									/></b>
									<br />
									Background:
									<b class="accent--text">
										<Typer
											:values="[contact.note]"
											:speed="12"
											:start-delay="2200"
											:animate="animate"
											@typed="done.note = true"
									/></b>
									<br />
									Pronouns:
									<b class="accent--text">
										<Typer
											:values="[contact.pronouns]"
											:speed="12"
											:start-delay="2400"
											:animate="animate"
											@typed="done.pronouns = true"
									/></b>
								</span>
							</div>
							<!-- each stamp flickers in as its matching field finishes typing -->
							<div class="col" :class="{ 'stamps-animate': animate }">
								<span class="stamp" :class="{ revealed: done.alias }">ALIAS REGISTERED</span> <br />
								<span class="stamp" :class="{ revealed: done.name }">IDENTITY VERIFIED</span> <br />
								<span class="stamp" :class="{ revealed: done.note }">UOB RECORDS VERIFIED</span>
								<br />
								<span class="stamp" :class="{ revealed: done.pronouns }">PH/HR DATA VERIFIED</span>
							</div>
						</div>
						<div style="padding-top: 5px">
							ASSIGNMENT <span class="subtle--text">(OMNINET PERSONNEL REMIT)</span>
						</div>
						<div class="row" style="padding-top: 5px">
							<span class="flavor-text">
								<Typer
									:values="[assignmentLine]"
									:speed="12"
									:start-delay="2600"
									:animate="animate"
								/>
							</span>
						</div>
						<template v-for="(item, index) in textItems" :key="item.header">
							<div style="padding-top: 5px">
								{{ item.header.toUpperCase() }}
								<span class="subtle--text"></span>
							</div>
							<div class="row narrative-text">
								<!-- bodies are HTML; TypeIt types markup (its `html` option defaults true) -->
								<Typer
									:values="[item.body]"
									:speed="16"
									:start-delay="3000 + index * 400"
									:animate="animate"
								/>
							</div>
						</template>
					</div>
					<div class="col" :class="{ 'portrait-animate': animate }">
						<div class="pilot-image-container" :style="{ animationDelay: portraitDelay }">
							<div class="pilot-image-border">
								<img :src="contactPortrait" class="portrait" @error="portraitFailed = true" />
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="flex-container-cols modal-buttons">
				<div class="row biometrics-container">
					<div class="biometrics flex-container-cols" @click="contactModal">
						<div>
							<i
								aria-hidden="true"
								class="v-icon notranslate mdi mdi-fingerprint theme--dark grey--text text--darken-2"
								style="font-size: 36px; margin-top: 36px"
							></i>
						</div>
						<div style="width: 100%">
							BIOMETRIC RECORD VALID [[{{ randomNumber(14, 22) }}PB]]<br />
							OHM C//{{ timeStamp(contact.save.lastModified) }}
						</div>
					</div>
				</div>
			</div>
			<hr role="separator" aria-orientation="horizontal" class="ma-2 v-divider theme--dark" />
			<div class="row row--dense">
				<span class="overline" style="line-height: 13px !important; opacity: 0.4">
					Improper use of this IDENT record and/or its constituent data by the record holder or any
					other persons is punishable under the DoJ/HR A-645-c. This record is the property of the
					Union Administrative Office and the information herein must be transmitted on request
					under NDL-C-DISCORDANT-BREATH encryption protocols. This RM-4 record must be updated every
					five (5) Cradle Standard Years of objective time to retain GMS licensing rights. Far-field
					operatives that anticipate deployments lasting longer than five Cradle Standard Years that
					have not been issued a man-portable Omninet Hook should apply for the RM-11-B IDENT
					Supplemental (b) Extension. Contact your local Union Adminstrative Officer for any other
					matters regarding this record. V-CDL//M-265-114-831 (A)
				</span>
			</div>
		</div>
	</div>
</template>

<style scoped>
.narrative-text {
	padding-top: 2px;
	max-width: 46em;
	font-size: 13px;
	line-height: 1.35em;
	color: var(--text-pilot-value);
}

/* Typed content carries no scope attribute, so :deep is required to reach it. */
.narrative-text :deep(p) {
	margin-bottom: 0.4em;
}

/* Typer's root is a <span>, and the narrative bodies are <p> blocks. Without this the
   paragraphs are block-in-inline, which lays out unpredictably. */
.narrative-text .type-it {
	display: block;
}

/* TypeIt injects `[data-typeit-id]::before { content: "."; display: inline-block }` — a
   hidden period that keeps an empty target from collapsing. Here the <p> already supplies
   height, so all it does is add a blank line above the text. Measured 18px, exactly one
   line-height. Only cleared for the narrative blocks; inline fields still want it. */
.narrative-text .type-it::before {
	content: none;
}

/* Only hide the stamps while the intro is animating; with animation off they just show. */
.stamps-animate .stamp {
	opacity: 0;
}

/* flickerIn is a global keyframe from _animations.css. */
.stamps-animate .stamp.revealed {
	animation: flickerIn 0.6s ease forwards;
}

/* Portrait flickers in once every typed field is under way; the delay comes from
   portraitDelay so it tracks the number of narrative sections. `both`, not `forwards`,
   so the 0% keyframe (opacity 0) also applies during the delay — otherwise it sits fully
   visible and then re-flickers, the same trap the header's planet container hit. */
.portrait-animate .pilot-image-container {
	animation: flickerIn 0.8s ease-in both;
}

.mdi::before {
	margin-top: 9px;
}

.modal-buttons {
	margin-top: 5px;
}
</style>

<script>
import ContactModal from "@/components/modals/ContactModal.vue";
import Typer from "@/components/Typer.vue";

export default {
	components: {
		Typer,
	},
	props: {
		animate: {
			type: Boolean,
			required: true,
		},
		contact: {
			type: Object,
			required: true,
		},
	},
	data() {
		return {
			portraitFailed: false,
			// Which identity fields have finished typing; drives the status stamps opposite.
			done: { alias: false, name: false, note: false, pronouns: false },
		};
	},
	computed: {
		contactPortrait() {
			if (this.portraitFailed) return this.contact.img?.cloud_portrait || "";
			return `/contacts/${this.contact.name.toUpperCase()}.webp`;
		},
		assignment() {
			return this.contact.folder?.folder || "UNASSIGNED";
		},
		// The last Typer to *start* is the final narrative section (3000 + index * 400), so
		// derive from that rather than hardcoding — it stays correct as sections are added.
		// The extra 200ms keeps the portrait strictly after the last line begins.
		portraitDelay() {
			const lastTyperStart = 3000 + Math.max(0, this.textItems.length - 1) * 400;
			return `${lastTyperStart + 200}ms`;
		},
		// The line the Typer types out — same content the card showed statically before.
		contactCode() {
			return `${this.reverse(this.contact.name)}:${this.contact.id}//NDL-C-BLIND-REACH`;
		},
		identLine() {
			return `Union Administrative RM-4 Personnel Identification Protocol (IDENT) Record ${this.contact.id}`;
		},
		// Typed as one line including the "//" separator. It goes through as markup rather
		// than plain text so the assignment keeps its accent colour while the title does not,
		// which a single plain-text Typer could not express.
		assignmentLine() {
			const assignment = `<b class="accent--text">${this.assignment}</b>`;
			return this.contact.title ? `${assignment} // ${this.contact.title}` : assignment;
		},
		// Rendered generically rather than picking out Appearance/Personality by name, so
		// any other section a GM adds in COMP/CON still shows up. Empty bodies are dropped
		// (COMP/CON keeps the item with an empty string when the field is cleared).
		textItems() {
			return (this.contact.narrative?.textItems || []).filter(item => item.body?.trim());
		},
	},
	methods: {
		reverse(str) {
			return str.split(" ").reverse().join(".");
		},
		randomNumber(max, min) {
			const rand = Math.random() * (max - min) + min;
			const power = Math.pow(10, 2);
			return Math.floor(rand * power) / power;
		},
		timeStamp(ms) {
			let date = new Date(ms);
			return new Date(
				date.getFullYear() + 2990,
				date.getMonth(),
				date.getDate(),
				date.getHours(),
				date.getMinutes(),
				date.getSeconds(),
				date.getMilliseconds()
			).toISOString();
		},
		contactModal() {
			this.$oruga.modal.open({
				component: ContactModal,
				custom: true,
				trapFocus: true,
				props: {
					contact: this.contact,
				},
				class: "custom-modal",
				width: 1920,
			});
		},
	},
};
</script>
