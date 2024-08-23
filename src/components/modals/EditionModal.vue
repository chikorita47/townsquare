<template>
  <Modal class="editions" v-if="modals.edition" @close="toggleModal('edition')">
    <div v-if="!isCustom">
      <h3>Select an edition:</h3>
      <ul class="editions">
        <li
          v-for="edition in editions"
          class="edition"
          :class="['edition-' + edition.id]"
          :style="{
            backgroundImage: `url(${require(
              '../../assets/editions/' + edition.id + '.png',
            )})`,
          }"
          :key="edition.id"
          @click="setEdition(edition)"
        >
          {{ edition.name }}
        </li>
        <li
          class="edition edition-custom"
          @click="isCustom = true"
          :style="{
            backgroundImage: `url(${require('../../assets/editions/custom.png')})`,
          }"
        >
          Custom Script / Characters
        </li>
      </ul>
    </div>
    <div class="custom" v-else>
      <h3>Load custom script / characters</h3>
      To play with a custom script, you need to select the characters you want
      to play with in the official
      <a href="https://script.bloodontheclocktower.com/" target="_blank"
        >Script Tool</a
      >
      and then upload the generated "custom-list.json" either directly here or
      provide a URL to such a hosted JSON file.<br />
      <br />
      To play with custom characters, please read
      <a
        href="https://github.com/bra1n/townsquare#custom-characters"
        target="_blank"
        >the documentation</a
      >
      on how to write a custom character definition file.
      <b>Only load custom JSON files from sources that you trust!</b>
      <h3>Some popular custom scripts:</h3>
      <ul class="scripts">
        <li
          v-for="(script, index) in scripts"
          :key="index"
          @click="handleURLOrJSON(script[1])"
        >
          {{ script[0] }}
        </li>
      </ul>
      <input
        type="file"
        ref="upload"
        accept="application/json"
        @change="handleUpload"
      />
      <div class="button-group">
        <div class="button" @click="openUpload">
          <font-awesome-icon icon="file-upload" /> Upload JSON
        </div>
        <div class="button" @click="promptURL">
          <font-awesome-icon icon="link" /> Enter URL
        </div>
        <div class="button" @click="readFromClipboard">
          <font-awesome-icon icon="clipboard" /> Use JSON from Clipboard
        </div>
        <div class="button" @click="isCustom = false">
          <font-awesome-icon icon="undo" /> Back
        </div>
      </div>
    </div>
  </Modal>
</template>

<script>
import editionJSON from "../../editions";
import { mapMutations, mapState } from "vuex";
import Modal from "./Modal";

export default {
  components: {
    Modal,
  },
  data: function () {
    return {
      editions: editionJSON,
      isCustom: false,
      scripts: [
        [
          "No Greater Joy (Teensyville)",
          '[{"id":"_meta","name":"No Greater Joy"},{"id":"clockmaker"},{"id":"investigator"},{"id":"empath"},{"id":"chambermaid"},{"id":"artist"},{"id":"sage"},{"id":"drunk"},{"id":"klutz"},{"id":"scarlet_woman"},{"id":"baron"},{"id":"imp"}]',
        ],
        [
          "Over the River (Teensyville)",
          '[{"id":"_meta","name":"Over the River","author":"Andrew Nathenson"},{"id":"grandmother"},{"id":"clockmaker"},{"id":"innkeeper"},{"id":"snake_charmer"},{"id":"professor"},{"id":"slayer"},{"id":"lunatic"},{"id":"recluse"},{"id":"godfather"},{"id":"spy"},{"id":"imp"}]',
        ],
        [
          "Laissez un Faire (Teensyville)",
          '[{"id":"_meta","name":"Laissez un Faire"},{"id":"balloonist"},{"id":"savant"},{"id":"amnesiac"},{"id":"fisherman"},{"id":"artist"},{"id":"cannibal"},{"id":"mutant"},{"id":"lunatic"},{"id":"widow"},{"id":"goblin"},{"id":"leviathan"}]',
        ],
        [
          "Strings Pulling",
          '[{"id":"_meta","name":"Strings Pulling","author":"Ben Burns"},{"id":"washerwoman"},{"id":"investigator"},{"id":"librarian"},{"id":"chef"},{"id":"empath"},{"id":"fortune_teller"},{"id":"undertaker"},{"id":"monk"},{"id":"ravenkeeper"},{"id":"virgin"},{"id":"slayer"},{"id":"soldier"},{"id":"mayor"},{"id":"butler"},{"id":"saint"},{"id":"recluse"},{"id":"drunk"},{"id":"poisoner"},{"id":"spy"},{"id":"baron"},{"id":"scarlet_woman"},{"id":"marionette"},{"id":"imp"}]',
        ],
        [
          "Shadow Puppets",
          '[{"id":"_meta","name":"Shadow Puppets"},{"id":"washerwoman"},{"id":"investigator"},{"id":"librarian"},{"id":"chef"},{"id":"empath"},{"id":"fortune_teller"},{"id":"undertaker"},{"id":"monk"},{"id":"ravenkeeper"},{"id":"virgin"},{"id":"slayer"},{"id":"soldier"},{"id":"mayor"},{"id":"butler"},{"id":"saint"},{"id":"recluse"},{"id":"drunk"},{"id":"lunatic"},{"id":"poisoner"},{"id":"spy"},{"id":"baron"},{"id":"scarlet_woman"},{"id":"marionette"},{"id":"imp"}]',
        ],
        /*
        [
          "Deadly Penance Day",
          '[{"id":"_meta","name":"Deadly Penance Day","author":"TPI"},{"id":"bountyhunter"},{"id":"chef"},{"id":"cultleader"},{"id":"exorcist"},{"id":"fool"},{"id":"gambler"},{"id":"investigator"},{"id":"mayor"},{"id":"philosopher"},{"id":"preacher"},{"id":"sailor"},{"id":"soldier"},{"id":"town_crier"},{"id":"politician"},{"id":"barber"},{"id":"drunk"},{"id":"saint"},{"id":"assassin"},{"id":"witch"},{"id":"spy"},{"id":"baron"},{"id":"lilmonsta"}]',
        ],
        [
          "Catfishing 11.1",
          '[{"id":"_meta","name":"Catfishing11.1","author":"Emily","logo":"https://i.imgur.com/tkFhtDn.png"},{"id":"investigator"},{"id":"chef"},{"id":"grandmother"},{"id":"balloonist"},{"id":"dreamer"},{"id":"fortune_teller"},{"id":"snake_charmer"},{"id":"gambler"},{"id":"savant"},{"id":"philosopher"},{"id":"ravenkeeper"},{"id":"amnesiac"},{"id":"cannibal"},{"id":"drunk"},{"id":"recluse"},{"id":"sweetheart"},{"id":"mutant"},{"id":"lunatic"},{"id":"godfather"},{"id":"cerenovus"},{"id":"pit-hag"},{"id":"widow"},{"id":"imp"},{"id":"vigormortis"},{"id":"fang_gu"},{"id":"beggar"},{"id":"barista"},{"id":"apprentice"},{"id":"harlot"},{"id":"bone_collector"}]',
        ],
        [
          "On Thin Ice (Teensyville)",
          '[{"id":"sentinel"},{"id":"artist"},{"id":"philosopher"},{"id":"professor"},{"id":"seamstress"},{"id":"slayer"},{"id":"virgin"},{"id":"klutz"},{"id":"moonchild"},{"id":"eviltwin"},{"id":"mastermind"},{"id":"pukka"}]',
        ],
        [
          "Race To The Bottom (Teensyville)",
          '[{"id":"doomsayer"},{"id":"sentinel"},{"id":"clockmaker"},{"id":"empath"},{"id":"dreamer"},{"id":"slayer"},{"id":"courtier"},{"id":"mayor"},{"id":"lunatic"},{"id":"klutz"},{"id":"scarletwoman"},{"id":"spy"},{"id":"vortox"}]',
        ],
        [
          "Frankenstein's Mayor by Ted (Teensyville)",
          '[{"id":"sentinel"},{"id":"chef"},{"id":"undertaker"},{"id":"oracle"},{"id":"sage"},{"id":"ravenkeeper"},{"id":"mayor"},{"id":"recluse"},{"id":"lunatic"},{"id":"poisoner"},{"id":"spy"},{"id":"zombuul"}]',
        ],
        [
          "Vigormortis High School (Teensyville)",
          '[{"id":"sentinel"},{"id":"chef"},{"id":"empath"},{"id":"chambermaid"},{"id":"monk"},{"id":"ravenkeeper"},{"id":"mayor"},{"id":"saint"},{"id":"goon"},{"id":"poisoner"},{"id":"devils_advocate"},{"id":"vigormortis"}]',
        ],
        */
      ],
    };
  },
  computed: mapState(["modals"]),
  methods: {
    openUpload() {
      this.$refs.upload.click();
    },
    handleUpload() {
      const file = this.$refs.upload.files[0];
      if (file && file.size) {
        const reader = new FileReader();
        reader.addEventListener("load", () => {
          try {
            const roles = JSON.parse(reader.result);
            this.parseRoles(roles);
          } catch (e) {
            alert("Error reading custom script: " + e.message);
          }
          this.$refs.upload.value = "";
        });
        reader.readAsText(file);
      }
    },
    promptURL() {
      const url = prompt("Enter URL to a custom-script.json file");
      if (url) {
        this.handleURL(url);
      }
    },
    async handleURL(url) {
      const res = await fetch(url);
      if (res && res.json) {
        try {
          const script = await res.json();
          this.parseRoles(script);
        } catch (e) {
          alert("Error loading custom script: " + e.message);
        }
      }
    },
    async handleURLOrJSON(urlOrJson) {
      try {
        const script = JSON.parse(urlOrJson);
        this.parseRoles(script);
      } catch {
        await this.handleURL(urlOrJson);
      }
    },
    async readFromClipboard() {
      const text = await navigator.clipboard.readText();
      try {
        const roles = JSON.parse(text);
        this.parseRoles(roles);
      } catch (e) {
        alert("Error reading custom script: " + e.message);
      }
    },
    parseRoles(roles) {
      if (!roles || !roles.length) return;
      roles = roles.map((role) =>
        typeof role === "string" ? { id: role } : role,
      );
      const metaIndex = roles.findIndex(({ id }) => id === "_meta");
      let meta = {};
      if (metaIndex > -1) {
        meta = roles.splice(metaIndex, 1).pop();
      }
      this.$store.commit("setCustomRoles", roles);
      this.$store.commit(
        "setEdition",
        Object.assign({}, meta, { id: "custom" }),
      );
      // check for fabled and set those too, if present
      if (roles.some((role) => this.$store.state.fabled.has(role.id || role))) {
        const fabled = [];
        roles.forEach((role) => {
          if (this.$store.state.fabled.has(role.id || role)) {
            fabled.push(this.$store.state.fabled.get(role.id || role));
          }
        });
        this.$store.commit("players/setFabled", { fabled });
      }
      this.isCustom = false;
    },
    ...mapMutations(["toggleModal", "setEdition"]),
  },
};
</script>

<style scoped lang="scss">
ul.editions .edition {
  font-family: Dumbledor, PiratesBay, sans-serif;
  text-transform: uppercase;
  text-align: center;
  padding-top: 15%;
  background-position: center center;
  background-size: contain;
  background-repeat: no-repeat;
  width: 30%;
  margin: 5px;
  font-size: 120%;
  text-shadow:
    -1px -1px 0 #000,
    1px -1px 0 #000,
    -1px 1px 0 #000,
    1px 1px 0 #000,
    0 0 5px rgba(0, 0, 0, 0.75);
  cursor: pointer;
  &:hover {
    color: red;
  }
}

.custom {
  text-align: center;
  input[type="file"] {
    display: none;
  }
  .scripts {
    list-style-type: disc;
    font-size: 120%;
    cursor: pointer;
    display: block;
    width: 50%;
    text-align: left;
    margin: 10px auto;
    li:hover {
      color: red;
    }
  }
}
</style>
