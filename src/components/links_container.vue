<template>
<div class="links-container">
  <article :id="index + 'folder'" class="links-folder" @mouseover="hover = true"  @mouseleave="hover = false">
    <h3 class="links-folder-name font--light" >{{ name }}</h3>
    <div class="links-folder--edit" v-show="hover" @click="isEditing = !isEditing;checkEditorPosition(this);">
      <svg width="15" height="15" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M54.7132 24.0924L24.0596 54.7461L15.3606 46.0471L46.0142 15.3934L54.7132 24.0924Z" fill="white" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
        <path d="M63.695 15.3934L54.6064 6.30492L55.0829 5.82843C56.645 4.26633 59.1777 4.26633 60.7398 5.82843L64.1715 9.2601C65.7336 10.8222 65.7336 13.3549 64.1715 14.917L63.695 15.3934Z" fill="white" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
        <path d="M14.8443 61.8689L8.13115 54.9743L4 66L14.8443 61.8689Z" fill="white" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </div>
    <div class="links-folder--delete" v-show="hover" @click="deleteContainer()">
      <svg width="13" height="13" viewBox="0 0 39 39" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M4 4L35 35" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
        <path d="M35 4L4 35" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </div>
    <section id="folderEditor" class="folder-editor" v-show="isEditing">
      <input class="input-name" v-model="name" type="text" maxlength="20">
      <link_editor v-on:deleteLink="deleteLink" v-for="link in links" :link=link />
      <div class="editor-options">
        <button class="add-link" id="addLink" @click="addLink">
          <svg width="18" height="18" viewBox="0 0 43 43" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M22 4V39" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M39 22L4 22" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
        <button class="editor--save" @click="saveOptions">
          <svg class="save" width="20" height="20" viewBox="0 0 51 35" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path  d="M4 13.9L20.6735 31L47 4" stroke="white" stroke-width="8" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
      </div>
    </section>
  </article>
  <article class="links font--light">
    <p v-for="link in links" class="link"><a target="_blank" :href="link.url">{{ link.name }}</a></p>
  </article>
</div>
</template>

<script>
import link_editor from "./link_editor.vue";

export default {
  props: {
    index: Number,
    name: String,
    links: Array,
  },
  data() {
    const self = this;
    return {
      hover: false,
      isEditing: false,
      removePopupListener: function(event) {
        self.closePopup(event);
      },
    }
  },
  methods: {
    saveOptions() {
      this.isEditing = false;
      this.$emit('modifyShortcuts', this.index, this.name, this.links)
    },
    deleteLink(link) {
      this.links.splice(this.links.indexOf(link), 1);
    },
    addLink() {
      if (this.links.length <= 10) {
        this.links.push({
          name: "...",
          url: "https://..."
        });
      } else {
        alert("There's a maximum amount of 10 links per section, sorry:))");
      }
    },
    prepareToClosePopup() {
      document.body.addEventListener("mousedown", this.removePopupListener);
      document.addEventListener('keydown', (event) => {
        if (event.key == 'Enter' && this.isEditing == true) {
          this.saveOptions();
          this.isEditing = false;
        }
      });
    },
    closePopup(event) {
      let popup = document.getElementById(`${this.index}folder`);
      if (popup) {
        if (event.target === popup || popup.contains(event.target)) {
          return;
        }
        this.isEditing = false;
      }
    },
    deleteContainer() {
      document.body.removeEventListener("mousedown", this.removePopupListener);
      this.isEditing = false;
      this.$emit('deleteContainer', this.index);
    },
    checkEditorPosition(e) {
      console.log(e)
    }
  },
  mounted() {
    this.prepareToClosePopup();
  },
  components: {
    link_editor,
  }
}
</script>

<style scoped>
button {
  background: none;
  border: none;
}

.link {
  padding: 0.25em;
}

a {
  color: var(--fontColor);
  text-decoration: none;
}


a:hover {
  text-decoration: line-through;
}

.links-container {
  display: flex;
  flex-direction: column;
}

.links-folder {
  padding: 1rem 2.75rem;
  display: flex;
  position: relative;
}

.links-folder-name {
  color: var(--primaryColor);
}

.links-folder--edit {
  position: absolute;
  right: 1.3rem;
  bottom: 18px;
  cursor: pointer;
  opacity: 30%;
}

.links-folder--edit:hover {
  opacity: 80%;
}

.links-folder--delete {
  position: absolute;
  right: 0;
  bottom: 18px;
  cursor: pointer;
  opacity: 30%;
}

.links-folder--delete:hover {
  opacity: 80%;
}

.folder-editor {
  border: 1px solid var(--fontColor);
  background: var(--backgroundColor);
  position: absolute;
  left: 80%;
  z-index: 10;
  box-shadow: 0px 0px 20px 0px rgba(61, 36, 36, 0.2);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 1.5rem;
  gap: 0.5rem;
}

.editor--save {
  cursor: pointer;
  align-self: center;
  opacity: 20%;
  transition: opacity 0.08s ease-in-out;
}

.add-link {
  cursor: pointer;
  opacity: 20%;
  transition: opacity 0.08s ease-in-out;
  align-self: flex-start;
}

.add-link:hover, .editor--save:hover {
  opacity: 80%;
}

.input-name {
  font-size: 1rem;
  padding: 0.25rem 0.25rem 0.25rem 0;
  background: none;
  border: none;
  border-bottom: 1px solid var(--fontColor);
  color: var(--fontColor);
  margin-bottom: 1rem;
}

.input-name:focus {
  outline: none;
  background: rgba(0, 0, 0, 0.1);
}

.editor-options {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

</style>
