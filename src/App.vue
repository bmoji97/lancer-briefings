<template>
	<div class="page-wrapper">
		<Header :planet-path="planetPath" :class="{ animate: animate }" :header="header" />
		<Sidebar :animate="animate" :class="{ animate: animate }" />
	</div>
	<div id="router-view-container">
		<router-view
			:animate="animate"
			:initial-slug="initialSlug"
			:missions="missions"
			:events="events"
			:pilots="pilots"
			:clocks="clocks"
			:reserves="reserves"
			:contacts="contacts"
			:factions="factions"
		/>
	</div>
	<svg
		style="visibility: hidden; position: absolute"
		width="0"
		height="0"
		xmlns="http://www.w3.org/2000/svg"
		version="1.1"
	>
		<defs>
			<filter id="round">
				<feGaussianBlur in="SourceGraphic" stdDeviation="5" result="blur" />
				<feColorMatrix
					in="blur"
					mode="matrix"
					values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 19 -5"
					result="goo"
				/>
				<feComposite in="SourceGraphic" in2="goo" operator="atop" />
			</filter>
		</defs>
	</svg>
	<audio autoplay>
		<source src="/startup.ogg" type="audio/ogg" />
	</audio>
</template>

<script>
import Header from "./components/layout/Header.vue";
import Sidebar from "./components/layout/Sidebar.vue";
import Config from "@/assets/info/general-config.json";

export default {
	components: {
		Header,
		Sidebar,
	},

	data() {
		return {
			animate: Config.animate,
			initialSlug: Config.initialSlug,
			planetPath: Config.planetPath,
			header: Config.header,
			pilotSpecialInfo: Config.pilotSpecialInfo,
			clocks: [],
			events: [],
			missions: [],
			pilots: [],
			reserves: [],
			bonds: [],
			contacts: [],
			factions: [],
		};
	},
	created() {
		this.setTitleFavicon(Config.defaultTitle + " MISSION BRIEFING", Config.icon);
		this.importMissions(
			import.meta.glob("@/assets/missions/*.md", { query: "?raw", import: "default" })
		);
		this.importEvents(
			import.meta.glob("@/assets/events/*.md", { query: "?raw", import: "default" })
		);
		this.importClocks(import.meta.glob("@/assets/clocks/*.json"));
		this.importReserves(import.meta.glob("@/assets/reserves/*.json"));
		this.importPilots(import.meta.glob("@/assets/pilots/*.json"));
		this.importContacts(import.meta.glob("@/assets/contacts/*.json"));
		this.importFactions(
			import.meta.glob("@/assets/factions/*.md", { query: "?raw", import: "default" })
		);
	},
	mounted() {
		this.$router.push("/status");
	},
	methods: {
		setTitleFavicon(title, favicon) {
			document.title = title;
			let headEl = document.querySelector("head");
			let faviconEl = document.createElement("link");
			faviconEl.setAttribute("rel", "shortcut icon");
			faviconEl.setAttribute("href", favicon);
			headEl.appendChild(faviconEl);
		},
		// Content files are CRLF on Windows (git core.autocrlf converts on checkout, and
		// editors save that way), while these parsers read fixed line indices. Splitting on
		// "\n" alone leaves a trailing \r inside every field — mission.status became
		// "start\r", which matched none of Mission.vue's status cases, so the label
		// vanished and the icon URL gained a stray character.
		splitLines(content) {
			return content.replace(/\r\n?/g, "\n").split("\n");
		},
		async importMissions(files) {
			let filePromises = Object.keys(files).map(path => files[path]());
			let fileContents = await Promise.all(filePromises);
			fileContents.forEach(content => {
				let lines = this.splitLines(content);
				let mission = {};
				mission["slug"] = lines[0];
				mission["name"] = lines[1];
				mission["status"] = lines[2];
				mission["content"] = lines.slice(3).join("\n");
				this.missions = [...this.missions, mission];
			});
			this.missions = this.missions.sort(function (a, b) {
				return b["slug"] - a["slug"];
			});
		},
		async importEvents(files) {
			let filePromises = Object.keys(files).map(path => files[path]());
			let fileContents = await Promise.all(filePromises);
			fileContents.forEach(content => {
				let lines = this.splitLines(content);
				let event = {};
				event["title"] = lines[0];
				event["location"] = lines[1];
				event["time"] = lines[2];
				event["thumbnail"] = lines[3];
				event["content"] = lines.slice(4).join("\n");
				this.events = [...this.events, event];
			});
			this.events = this.events.reverse();
		},
		async importClocks(files) {
			let filePromises = Object.keys(files).map(path => files[path]());
			let fileContents = await Promise.all(filePromises);
			fileContents.forEach(content => {
				this.clocks = JSON.parse(JSON.stringify(content)).default;
			});
		},
		async importReserves(files) {
			let filePromises = Object.keys(files).map(path => files[path]());
			let fileContents = await Promise.all(filePromises);
			fileContents.forEach(content => {
				this.reserves = JSON.parse(JSON.stringify(content)).default;
			});
		},
		async importPilots(files) {
			let filePromises = Object.keys(files).map(path => files[path]());
			let fileContents = await Promise.all(filePromises);
			fileContents.forEach(content => {
				// COMP/CON v3 wraps the pilot in an envelope: { EXPORT_TYPE: "Save Pilot", data }.
				// Everything downstream expects the pilot record itself, so unwrap it here.
				// (vite's namedExports puts `data` on the module as well as under `default`.)
				let file = JSON.parse(JSON.stringify(content));
				let pilotFromJson = file.data ?? file.default?.data;

				// Old flat-schema exports have no envelope. Skip them loudly rather than
				// throwing here, which would abort the whole loader and blank every view.
				if (!pilotFromJson) {
					console.warn(
						`[pilots] Skipping "${file.callsign ?? file.default?.callsign ?? "unknown"}": ` +
							`not a COMP/CON v3 export (expected { EXPORT_TYPE, data }). Re-export it from COMP/CON.`
					);
					return;
				}
				// In case the pilot was added from a copy on compcon via sharecode, remove the "reference mark" symbol
				pilotFromJson.name = pilotFromJson.name.replace("※", "");
				pilotFromJson.callsign = pilotFromJson.callsign.replace("※", "");
				let pilotFromVue = this.pilotSpecialInfo[pilotFromJson.callsign.toUpperCase()];
				let pilot = {
					...pilotFromJson,
					...pilotFromVue,
				};
				this.pilots = [...this.pilots, pilot];
				// v3 moved the pilot's clocks under the bond block.
				(pilot.bond?.clocks || []).forEach(content => {
					let clock = {};
					clock["type"] = `Pilot Project // ${pilot.callsign}`;
					clock["result"] = "";
					clock["name"] = content.title;
					clock["description"] = content.description;
					clock["value"] = content.progress;
					clock["max"] = content.segments;
					clock["color"] = "#3CB043";
					this.clocks = [...this.clocks, clock];
				});

				(pilot.reserves || []).forEach(content => {
					let reserve = {};
					reserve["type"] = content.type;
					reserve["name"] = content.name;
					reserve["description"] = content.description;
					reserve["label"] = content.label;
					reserve["cost"] = content.cost;
					reserve["notes"] = content.notes;
					reserve["callsign"] = pilot.callsign.toUpperCase();
					this.reserves = [...this.reserves, reserve];
				});
			});
		},
		async importContacts(files) {
			let filePromises = Object.keys(files).map(path => files[path]());
			let fileContents = await Promise.all(filePromises);
			fileContents.forEach(content => {
				let contact = JSON.parse(JSON.stringify(content)).default;
				this.contacts = [...this.contacts, contact];
			});
			this.contacts = this.contacts.sort(function (a, b) {
				return a.name.localeCompare(b.name);
			});
		},
		async importFactions(files) {
			let filePromises = Object.keys(files).map(path => files[path]());
			let fileContents = await Promise.all(filePromises);
			fileContents.forEach(content => {
				let lines = this.splitLines(content);
				let faction = {};
				faction["title"] = lines[0];
				faction["location"] = lines[1];
				faction["disposition"] = lines[2];
				faction["thumbnail"] = lines[3];
				faction["content"] = lines.slice(4).join("\n");
				this.factions = [...this.factions, faction];
			});
			this.factions = this.factions.reverse();
		},
	},
};
</script>

<style>
#app {
	min-height: 100vh;
	overflow: hidden !important;
	/* border-right: 1px solid #ff0;
	border-bottom: 1px solid #ff0; */
}
</style>
