<template>
	<div
		class="grid-item pilot-identity"
		:class="{ 'intro-animate': animate }"
		style="color: white !important"
	>
		<div class="header">
			<div class="col grow-max">
				<div class="heading h1">{{ pilot.callsign }}</div>
				<div class="heading h2">({{ pilot.name }})</div>
			</div>
			<div class="col"><img src="/faction-logos/union.svg" /></div>
		</div>
		<div class="body">
			<div class="add-padding">
				<Typer :values="[identLine]" :speed="12" :start-delay="300" :animate="animate" />
			</div>
			<div class="flex-container-rows">
				<div class="row add-padding">
					<Typer :values="[codeLine]" :speed="12" :start-delay="900" :animate="animate" />
				</div>
				<div class="row flex-container-cols add-padding">
					<div class="col grow-max flex-container-rows" style="padding-top: 5px">
						<div class="row flex-container-cols">
							<div class="col col-primary">
								<span class="flavor-text">
									Callsign:
									<b class="accent--text">
										<Typer
											:values="[capitalize(pilot.callsign)]"
											:speed="12"
											:start-delay="1800"
											:animate="animate"
											@typed="done.callsign = true"
									/></b>
									<br />
									Name (or legal alias):
									<b class="accent--text">
										<Typer
											:values="[pilot.name]"
											:speed="12"
											:start-delay="2000"
											:animate="animate"
											@typed="done.name = true"
									/></b>
									<br />
									Background:
									<b class="accent--text">
										<Typer
											:values="[pilot.background]"
											:speed="12"
											:start-delay="2200"
											:animate="animate"
											@typed="done.background = true"
									/></b>
									<template v-if="hasBond">
										<br />
										Bond:
										<b class="accent--text">
											<Typer
												:values="[bond.name]"
												:speed="12"
												:start-delay="2400"
												:animate="animate"
												@typed="done.bond = true"
										/></b>
									</template>
								</span>
							</div>
							<!-- each stamp flickers in as its matching field finishes typing -->
							<div class="col" :class="{ 'stamps-animate': animate }">
								<span class="stamp" :class="{ revealed: done.callsign }">CALLSIGN AVAILABLE</span>
								<br />
								<span class="stamp" :class="{ revealed: done.name }">IDENTITY VERIFIED</span>
								<br />
								<span class="stamp" :class="{ revealed: done.background }">
									PH/HR DATA REGISTERED
								</span>
								<template v-if="hasBond">
									<br />
									<span class="stamp" :class="{ revealed: done.bond }">BOND CHOSEN</span>
								</template>
							</div>
						</div>
						<div v-if="hasBond">
							<div style="padding-top: 5px" class="col flex-container-rows">
								<div class="row row-share">
									<span>PILOT BOND POWER AUDIT</span>
									<br />
									<div
										class="chip-container flicker-in"
										v-for="(power, index) in bondPowers"
										:key="power.name"
										:style="{ animationDelay: bondDelay(index) }"
									>
										<span class="chip"
											><i aria-hidden="true" class="notranslate cci cci-skill"></i
											>{{ getBondPower(power) }}</span
										>
									</div>
									<div v-if="!bondPowers.length" class="subtle--text">[ NONE SELECTED ]</div>
								</div>
							</div>
							<div class="row flex-container-cols pilot-tracks">
								<div class="col col-share">
									<span>BOND XP</span>
									<ProgressBar :max="8" :value="pilot.bond.xp || 0" color="#3cb043" />
									<span>STRESS</span>
									<ProgressBar
										:max="pilot.bond.maxStress || 8"
										:value="pilot.bond.stress || 0"
										color="#aa0000"
									/>
								</div>
								<div class="col col-share">
									<span>BURDENS</span>
									<div v-if="burdens.length" class="burdens">
										<Burden
											v-for="burden in burdens"
											:key="burden.id || burden.title"
											:burden="burden"
											:animate="animate"
										/>
									</div>
									<div v-else class="subtle--text">[ NONE RECORDED ]</div>
								</div>
							</div>
						</div>
						<div style="padding-top: 5px">
							FRAME CONFIGURATION OPTIONS
							<span class="subtle--text">("H.A.S.E" OMNINET VAULT REMIT)</span>
						</div>
						<div class="row" style="padding-top: 5px">
							<span style="font-size: 22px; line-height: 15px">
								[ HULL:
								<span
									class="stat-text accent--text flicker-in"
									style="font-size: 24px"
									:style="{ animationDelay: haseDelay(0) }"
								>
									{{ pilot.mechSkills[0] }}
								</span>
								AGI:
								<span
									class="stat-text accent--text flicker-in"
									style="font-size: 24px"
									:style="{ animationDelay: haseDelay(1) }"
								>
									{{ pilot.mechSkills[1] }}
								</span>
								SYS:
								<span
									class="stat-text accent--text flicker-in"
									style="font-size: 24px"
									:style="{ animationDelay: haseDelay(2) }"
								>
									{{ pilot.mechSkills[2] }}
								</span>
								ENG:
								<span
									class="stat-text accent--text flicker-in"
									style="font-size: 24px"
									:style="{ animationDelay: haseDelay(3) }"
								>
									{{ pilot.mechSkills[3] }}
								</span>
								]
							</span>
						</div>
						<div class="row flex-container-cols">
							<div class="col col-share">
								<span>PILOT SKILL TRIGGER AUDIT</span>
								<br />
								<div
									class="chip-container flicker-in"
									v-for="(skill, index) in pilot.skills"
									:key="skill.id"
									:style="{ animationDelay: auditDelay(index) }"
								>
									<span class="chip"
										><i aria-hidden="true" class="notranslate cci cci-skill"></i
										>{{ getSkill(skill) }}</span
									>
								</div>
							</div>
							<div class="col col-share">
								<span>PILOT TALENT AUDIT</span>
								<br />
								<div
									class="chip-container flicker-in"
									v-for="(talent, index) in pilot.talents"
									:key="talent.id"
									:style="{ animationDelay: auditDelay(skillCount + index) }"
								>
									<span class="chip"
										><i aria-hidden="true" class="notranslate cci cci-talent"></i
										>{{ getTalent(talent) }}</span
									>
								</div>
							</div>
						</div>
						<div v-if="pilot.level > 0" class="row flex-container-cols">
							<div class="col" style="padding-top: 5px">
								<span>PROCUREMENT LICENSE AUDIT: LEVEL {{ pilot.level }}</span>
								<br />
								<div
									class="chip-container flicker-in"
									v-for="(license, index) in pilot.licenses"
									:key="license.id"
									:style="{ animationDelay: auditDelay(skillCount + talentCount + index) }"
								>
									<span class="chip"
										><i aria-hidden="true" class="notranslate cci cci-license"></i
										>{{ getLicense(license) }}</span
									>
								</div>
							</div>
						</div>
					</div>
					<div class="col" :class="{ 'portrait-animate': animate }">
						<div class="pilot-image-container" :style="{ animationDelay: portraitDelay }">
							<div class="pilot-image-border">
								<img :src="pilotPortrait" class="portrait" />
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="flex-container-cols modal-buttons">
				<div class="row biometrics-container">
					<div class="biometrics flex-container-cols" @click="pilotModal">
						<div>
							<i
								aria-hidden="true"
								class="v-icon notranslate mdi mdi-fingerprint theme--dark grey--text text--darken-2"
								style="font-size: 36px; margin-top: 36px"
							></i>
						</div>
						<div style="width: 100%">
							BIOMETRIC RECORD VALID [[{{ randomNumber(14, 22) }}PB]]<br />
							OHM C//{{ timeStamp(pilot.save.lastModified) }}
						</div>
					</div>
				</div>
				<div class="row biometrics-container">
					<div class="mech-record flex-container-cols" @click="mechModal">
						<div style="width: 100%">
							MECHANICAL BLUEPRINT VALID [[{{ randomNumber(14, 22) }}TB]] <br />
							{{ activeMech.manufacturer.toUpperCase() }}-{{
								activeMech.frame_name.toUpperCase()
							}}
							:: "{{ activeMech.name.toUpperCase() }}"
						</div>
						<div>
							<i
								aria-hidden="true"
								class="v-icon notranslate cci cci-reserve-mech theme--dark grey--text text--darken-2 larger"
								style="font-size: 42px; margin-top: 1em"
							></i>
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
					matters regarding this record.  V-CDL//M-265-114-831 (A)
				</span>
			</div>
		</div>
	</div>
</template>

<style scoped>
/* Only hide the stamps while the intro is animating; with animation off they just show. */
.stamps-animate .stamp {
	opacity: 0;
}

/* flickerIn is a global keyframe from _animations.css. */
.stamps-animate .stamp.revealed {
	animation: flickerIn 0.6s ease forwards;
}

/* HASE numbers and the audit chips. `both`, not `forwards`, so the 0% keyframe (opacity
   0) applies during the delay too — with `forwards` they would sit visible and then
   re-flicker. Only active while the intro is animating. */
.intro-animate .flicker-in {
	animation: flickerIn 0.6s ease both;
}

/* Portrait is last, after the final audit chip. Same `both` reasoning. */
.portrait-animate .pilot-image-container {
	animation: flickerIn 0.8s ease-in both;
}

.larger::before {
	margin-top: 9px;
}

.mdi::before {
	margin-top: 9px;
}

.mech-record {
	margin-left: auto;
	text-align: right;
}

.modal-buttons {
	margin-top: 5px;
}
</style>

<script>
import "external-svg-loader";
import lancerData from "@massif/lancer-data";
import ktbData from "lancer-ktb-data";
import nrfawData from "lancer-nrfaw-data";
import longrimData from "lancer-longrim-data";

import wallflowerData from "@/assets/LCPs/wallflower-data-2.0.5";
/*Append the datasets within computed if your LCP has new items.
EX:
pilotGear() {
  return [...lancerData.pilot_gear, ...wallflowerData.pilot_gear]
},
*/
import dustgraveData from "@/assets/LCPs/dustgrave-data-1.4.0";
import osrData from "@/assets/LCPs/osr-data-1.2.0";
import owsData from "@/assets/LCPs/ows-data-1.0.0";
import sotwData from "@/assets/LCPs/sotw-data-1.0.2";
import ssmrData from "@/assets/LCPs/ssmr-data-1.7.0";

import PilotModal from "@/components/modals/PilotModal.vue";
import MechModal from "@/components/modals/MechModal.vue";

import Typer from "@/components/Typer.vue";

// Intro cascade timings (ms). The identity fields finish typing around 2400; everything
// after that chains in the order the sections appear on the card — bond powers, then the
// HASE numbers, then the audit chips, then the portrait. Each stage's start is derived
// from the previous one so a pilot with more bond powers or chips pushes the rest back
// rather than overlapping them.
const BOND_BASE = 2500;
const CHIP_STEP = 120;
const HASE_STEP = 150;
const STAGE_GAP = 150;

import ProgressBar from "@/components/ProgressBar.vue";
import Burden from "@/components/Burden.vue";

export default {
	components: {
		Burden,
		ProgressBar,
		Typer,
	},
	props: {
		animate: {
			type: Boolean,
			required: true,
		},
		pilot: {
			type: Object,
			required: true,
		},
	},
	data() {
		return {
			activeMech: {},
			bond: {},
			// Which identity fields have finished typing; drives the status stamps opposite.
			done: { callsign: false, name: false, background: false, bond: false },
		};
	},
	computed: {
		pilotPortrait() {
			return `/pilots/${this.pilot.callsign.toUpperCase()}.webp`;
		},
		// XP, stress and burdens are all part of the Bonds subsystem, so the whole block is
		// gated on the pilot actually having one — otherwise every card shows two empty
		// tracks and a "none recorded" placeholder.
		hasBond() {
			return !!this.pilot.bond?.bondId;
		},
		identLine() {
			return `Union Administrative RM-4 Pilot Identification Protocol (IDENT) Record ${this.pilot.id}`;
		},
		codeLine() {
			return `${this.reverse(this.pilot.name)}:${this.pilot.id}//NDL-C-BLIND-REACH`;
		},
		skillCount() {
			return (this.pilot.skills || []).length;
		},
		talentCount() {
			return (this.pilot.talents || []).length;
		},
		licenseCount() {
			return this.pilot.level > 0 ? (this.pilot.licenses || []).length : 0;
		},
		bondPowerCount() {
			return this.hasBond ? this.bondPowers.length : 0;
		},
		// HASE picks up after the bond power chips have all flickered.
		haseStart() {
			return BOND_BASE + this.bondPowerCount * CHIP_STEP + STAGE_GAP;
		},
		// Audits pick up after the four HASE numbers.
		auditStart() {
			return this.haseStart + 4 * HASE_STEP + STAGE_GAP;
		},
		// Last in the sequence: the portrait waits for the final audit chip, so it scales
		// with how many bond powers, skills, talents and licences a pilot actually has.
		portraitDelay() {
			const chips = this.skillCount + this.talentCount + this.licenseCount;
			return `${this.auditStart + chips * CHIP_STEP + 300}ms`;
		},
		// Bond state lives under pilot.bond in the v3 schema; guard it anyway so a pilot
		// exported without the block doesn't take the card down.
		burdens() {
			return this.pilot.bond?.burdens || [];
		},
		// COMP/CON stores each selected power as a copy of the bond's power object, but
		// older exports store just the name. Normalise both, then re-resolve against the
		// bond so the full record (description, veteran/master flags) is available either
		// way rather than depending on how the pilot happened to be exported.
		bondPowers() {
			const powers = this.bond?.powers || [];
			return (this.pilot.bond?.bondPowers || []).map(entry => {
				const name = typeof entry === "string" ? entry : entry?.name;
				return powers.find(p => p.name === name) || { name: name || "UNKNOWN POWER" };
			});
		},
		mechPortrait() {
			return `/mechs/${this.pilot.callsign.toUpperCase()}.webp`;
		},
		pilotGear() {
			return [
				...lancerData.pilot_gear,
				...wallflowerData.pilot_gear,
				...dustgraveData.pilot_gear,
				...sotwData.pilot_gear,
				...ssmrData.pilot_gear,
			];
		},
		mechWeapons() {
			return [
				...lancerData.weapons,
				...ktbData.weapons,
				...nrfawData.weapons,
				...longrimData.weapons,
				...wallflowerData.weapons,
				...dustgraveData.weapons,
				...osrData.weapons,
				...owsData.weapons,
				...ssmrData.weapons,
			];
		},
		mechSystems() {
			return [
				...lancerData.systems,
				...ktbData.systems,
				...nrfawData.systems,
				...longrimData.systems,
				...wallflowerData.systems,
				...dustgraveData.systems,
				...osrData.systems,
				...owsData.systems,
				...sotwData.systems,
				...ssmrData.systems,
			];
		},
		talents() {
			return [
				...lancerData.talents,
				...ktbData.talents,
				...nrfawData.talents,
				...longrimData.talents,
				...wallflowerData.talents,
				...dustgraveData.talents,
				...osrData.talents,
				...ssmrData.talents,
			];
		},
		skills() {
			return [...lancerData.skills];
		},
		bonds() {
			return [...ktbData.bonds, ...sotwData.bonds];
		},
		frames() {
			return [
				...lancerData.frames,
				...ktbData.frames,
				...nrfawData.frames,
				...longrimData.frames,
				...wallflowerData.frames,
				...dustgraveData.frames,
				...osrData.frames,
				...owsData.frames,
				...sotwData.frames,
				...ssmrData.frames,
			];
		},
		mechManufacturerIcon() {
			if (this.activeMech.manufacturer)
				return `/faction-logos/${this.activeMech.manufacturer.toLowerCase()}.svg`;
			return "";
		},
		// v3 keeps an array of pilot loadouts with the selected one indexed by active_index,
		// replacing the single `loadout` object of the old schema.
		loadout() {
			const loadouts = this.pilot.loadouts || [];
			return loadouts[this.pilot.active_index || 0] || { id: "", armor: [], weapons: [], gear: [] };
		},
		pilotCode() {
			const identNameParts = this.pilot.name.split(" ");
			const identFirstName = identNameParts[0];
			const identLastNameParts = identNameParts.slice(1);
			let identName = "";
			identLastNameParts.forEach(part => {
				identName += `${part}.`;
			});
			identName += identFirstName;
			return `Union Administrative RM-4 Pilot Identification Protocol (IDENT) Record ${identName}: ${this.pilot.id} // ${this.pilot.background} // LOADOUT ${this.loadout.id} - MECH ${this.pilot.mechs[0]?.id} // HARDPOINTS ${this.pilot.mechs[0]?.loadouts[0]?.id}`;
		},
		pilotInfo() {
			const info = this.pilot;

			let resolveGear = (type, item, idx, arr) => {
				item = item || { id: "", flavorName: "" };
				const gear =
					this.pilotGear.find(obj => {
						return item.id === obj.id;
					}) || null;
				item.flavorName = gear?.name || "ERR: DATA NOT FOUND";
				arr[idx] = item;
			};

			this.loadout.armor.forEach((item, index, array) => resolveGear("armor", item, index, array));
			this.loadout.weapons.forEach((item, index, array) =>
				resolveGear("weapon", item, index, array)
			);
			this.loadout.gear.forEach((item, index, array) => resolveGear("gear", item, index, array));

			return info;
		},
	},
	created() {
		this.getActiveMech();
		this.getBond();
	},
	methods: {
		getBond() {
			// v3 embeds the whole bond (name, ideals, powers) under bond.data, so prefer that
			// and only fall back to the merged LCP list for exports that lack it.
			this.bond =
				this.pilot.bond?.data ||
				this.bonds.find(obj => {
					return obj.id === this.pilot.bond?.bondId;
				});
		},
		getActiveMech() {
			// v3 dropped state.active_mech_id — there is no "active mech" in the export any
			// more, just an ordered list — so the first mech stands in for it.
			this.activeMech = this.pilot.mechs?.[0] || {};

			// frameData is embedded per-mech in v3; fall back to the merged LCP frame list
			// for anything exported without it.
			let frame =
				this.activeMech.frameData ||
				this.frames.find(obj => {
					return obj.id === this.activeMech.frame;
				});

			if (!frame) frame = lancerData.frames[0];

			this.activeMech.frame_description = frame.description;
			this.activeMech.frame_name = frame.name;
			this.activeMech.manufacturer = frame.source;
			this.activeMech.mechtype = frame.mechtype.join(" // ");
		},
		getHistory() {
			if (this.pilot.history === "") {
				return `<p> <h2> [ERR: REDACTED] </h2> </p>`;
			}

			let response = "<p>";

			if (this.pilot.text_appearance !== "") {
				response += `<h2>APPEARANCE</h2> ${this.pilot.text_appearance} </hr>`;
			}

			if (this.pilot.history !== "") {
				response += `<h2>HISTORY</h2> ${this.pilot.history} </hr>`;
			}

			response += "</p>";

			return response;
		},
		getSkill(skill) {
			// v3 embeds the skill under .data; fall back to the merged LCP list otherwise.
			const record = skill.data || this.skills.find(x => x.id == skill.id);
			return `${record?.name ?? skill.id} +${(skill.rank || 0) * 2}`;
		},
		// Bond powers have no rank, so mark the tier instead — matching how getTalent and
		// getLicense append their rank numerals.
		getBondPower(power) {
			if (power.master) return `${power.name} // MASTER`;
			if (power.veteran) return `${power.name} // VETERAN`;
			return power.name;
		},
		getTalent(talent) {
			// v3 embeds the talent under .data; fall back to the merged LCP list otherwise.
			const record = talent.data || this.talents.find(x => x.id == talent.id);
			return `${record?.name ?? talent.id} ${"I".repeat(talent.rank || 0)}`;
		},
		getLicense(license) {
			// v3 carries a `stub` with the frame's name and source, which saves resolving the
			// frame out of the merged LCP arrays (and works for brews those arrays lack).
			const stub = license.stub || this.frames.find(x => x.id == license.id);
			const name = stub?.name ?? license.id;
			const source = stub?.source ? `${stub.source} ` : "";
			return `${source}${name} ${"I".repeat(license.rank || 0)}`;
		},
		// Bond power chips flicker first, straight after the identity fields finish typing.
		bondDelay(index) {
			return `${BOND_BASE + index * CHIP_STEP}ms`;
		},
		haseDelay(index) {
			return `${this.haseStart + index * HASE_STEP}ms`;
		},
		// One continuous cascade across the skill, talent and licence audits — callers pass
		// a running index so the three lists read as a single sweep rather than three.
		auditDelay(index) {
			return `${this.auditStart + index * CHIP_STEP}ms`;
		},
		capitalize(str) {
			return str
				.split(" ")
				.map(word => word[0].toUpperCase() + word.slice(1))
				.join(" ");
		},
		reverse(str) {
			const words = str.split(" ");
			const reversed = words.reverse();
			const reversedResult = words.join(".");
			return reversedResult;
		},
		randomNumber(max, min) {
			const rand = Math.random() * (max - min) + min;
			const power = Math.pow(10, 2);
			return Math.floor(rand * power) / power;
		},
		timeStamp(str) {
			let date = new Date(str);
			let y = date.getFullYear();
			let m = date.getMonth();
			let d = date.getDate();
			let h = date.getHours();
			let mi = date.getMinutes();
			let s = date.getSeconds();
			let ms = date.getMilliseconds();
			let tz = date.getTimezoneOffset();
			y += 2990;
			return new Date(y, m, d, h, mi, s, ms).toISOString();
		},
		pilotModal() {
			this.$oruga.modal.open({
				component: PilotModal,
				custom: true,
				trapFocus: true,
				props: {
					pilot: this.pilot,
					talents: this.talents,
					skills: this.skills,
					frames: this.frames,
				},
				class: "custom-modal",
				width: 1920,
			});
		},
		mechModal() {
			this.$oruga.modal.open({
				component: MechModal,
				custom: true,
				trapFocus: true,
				props: {
					animate: this.animate,
					mech: this.activeMech,
					systemsData: this.mechSystems,
					weaponsData: this.mechWeapons,
					pilot: this.pilot,
				},
				class: "custom-modal",
				width: 1920,
			});
		},
	},
};
</script>
