<template>
  <div class="gml">
    <div class="uk-margin">
      <input v-model="model.name" class="uk-input uk-border-rounded uk-width-1-1" placeholder="name"/>
    </div>
    <div class="uk-margin">
      <input class="uk-input uk-border-rounded uk-width-1-1" placeholder="search for location..." @input="searchForAddress"/>
      <label v-if="typing" class="uk-text">{{ typing }}</label>
      <ul class="uk-list" v-if="!typing">
        <li v-for="(address, addressKey) in addresses" :key="addressKey" @click="chooseAddress(address)">
          {{address.formatted_address}}
        </li>
      </ul>
    </div>
    <div class="uk-margin">
      <input v-model="model.lat" class="uk-input uk-border-rounded uk-width-1-1" placeholder="lat" type="number" min="-90" max="90" />
    </div>
    <div class="uk-margin">
      <input v-model="model.lng" class="uk-input uk-border-rounded uk-width-1-1" placeholder="lng" type="number" min="-180" max="180"/>
    </div>
    <div class="uk-margin">
      <textarea v-model="model.description" placeholder="description" class="uk-textarea uk-border-rounded"></textarea>
    </div>
    <div class="uk-margin">
      <div v-if="model.image">
        <img :src="model.image" class="uk-display-blok"/>
        <a class="link uk-margin-small-top uk-display-blok" href="#" @click="model.image = null">Remove</a>
      </div>
      <sb-asset-selector :uid="uid" field="image" v-else/>
    </div>
  </div>
</template>

<script>
export default {
  mixins: [window.Storyblok.plugin],
  data() {
    return {
      open: [],
      searchTerm: '',
      debounce: null,
      addresses: [],
      typing: null,
    }
  },
  methods: {
    initWith() {
      return {
        // needs to be equal to your storyblok plugin name
        plugin: 'google-map-locations',
        lat: null,
        lng: null,
        image: null,
        name: null,
        description: null,
      }
    },
    pluginCreated() {
      // eslint-disable-next-line
      console.log('View source and customize: https://github.com/storyblok/storyblok-fieldtype')
    },
    searchForAddress(event) {
      this.searchTerm = null;
      this.typing = 'typing..';
      clearTimeout(this.debounce);
      this.debounce = setTimeout(() => {
        this.typing = null;
        this.addresses = [];
        this.searchTerm = event.target.value;
      }, 600);
    },
    chooseAddress(address) {
      this.model.lat = address.geometry.location.lat;
      this.model.lng = address.geometry.location.lng;
      this.resetAddresses();
      this.resetSearchTerm();
    },
    resetAddresses() {
      this.addresses = [];
    },
    resetSearchTerm() {
      this.searchTerm = null;
    }
  },
  watch: {
    'model': {
      handler: function (value) {
        this.$emit('changed-model', value);
      },
      deep: true
    },
    searchTerm() {
      if(!this.searchTerm) {
        return;
      }

      if(!this.options.apiKey || this.options.apiKey === '') {
        throw new Error('please define an API Key in your options!');
      }

      fetch(
        `https://maps.googleapis.com/maps/api/geocode/json?address=${encodeURIComponent(this.searchTerm)}&key=${this.options.apiKey}`
      ).then(async (response) => {
        const {status, results } = await response.json();
        if (status === 'OK') {
          this.addresses = results;
        }
      });
    }
  },
}
</script>

<style>
  /* gml == google maps location */
  .gml .uk-list li:not(:last-of-type){
    border-bottom: 1px solid rgba(0,0,0,0.25);
  }
  .gml .uk-list li {
    display: block;
    cursor: pointer;
    padding: 0.25rem 0.5rem;
  }
  .gml .uk-list li:hover {
    background-color: #00b3b0;
    color: white;
  }
  .gml .uk-list li:nth-child(2n):not(:hover) {
    background-color: rgba(0,0,0,0.1);
  }
</style>
