<template>
  <div class="gml">
    <div class="uk-margin">
      <input class="uk-input uk-border-rounded uk-width-1-1" placeholder="search for location..." v-model="searchTerm" />
      <label v-if="typing" class="uk-text typing">{{ typing }}</label>
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
      addressWasChosen: false,
    }
  },
  methods: {
    /**
     * Storyblok Specific Initialize with Function.
     * Data returned will be saved in storyblok, and served via wpi
     */
    initWith() {
      return {
        // needs to be equal to your storyblok plugin name
        plugin: 'google-map-locations',
        lat: null,
        lng: null,
      }
    },

    /**
     * Choose Address from list of addresses returned by google geocode api
     *
     * @param {array} address
     */
    chooseAddress(address) {
      // makes sure we don't trigger mutliple updates
      this.addressWasChosen = true;
      this.model.lat = address.geometry.location.lat;
      this.model.lng = address.geometry.location.lng;
      this.searchTerm = address.formatted_address;
      this.resetAddresses();
      setTimeout(() => {
        this.addressWasChosen = false;
      }, 500);
    },
    /**
     * Reset Addresses
     */
    resetAddresses() {
      this.addresses = [];
    },
    /**
     * Reset search Term
     */
    resetSearchTerm() {
      this.searchTerm = null;
    },
    /**
     * Searches for Addresses via Google Geocode Api
     */
    searchForAddresses() {
      this.typing = null;

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
  watch: {
    'model': {
      handler: function (value) {
        this.$emit('changed-model', value);
      },
      deep: true
    },
    searchTerm() {
      if(!this.searchTerm || this.addressWasChosen) {
        return;
      }
      this.typing = 'typing...';
      clearTimeout(this.debounce);
      this.debounce = setTimeout(() => {
        this.searchForAddresses();
      }, 300);
    }
  },
}
</script>

<style>
  /* gml == google maps location */
  .gml .uk-list li:not(:last-of-type){
    border-bottom: 1px solid rgba(0,0,0,0.25);
  }
  .gml .typing {
    padding: 0.25rem 0.5rem;
    display: block;
    margin-top: 15px;
    color: grey;
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
    background-color: #f6f8f9;
  }
</style>
