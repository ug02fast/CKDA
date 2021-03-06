<template>
  <v-container>
    <v-layout row>
      <v-flex xs8>
        <v-text-field
          name="eth-address"
          label="Enter An Ethereum Wallet Address to see CryptoKitties"
          id="eth-address"
          single-line
          @keydown.enter.native="searchKitty"
        ></v-text-field>
      </v-flex>
    </v-layout>
    <v-container>
      <v-layout row justify-space-around>
        <v-flex xs6>Key: <span style="color:blue">Blue = Rare</span> <span style="color:green">Green = Uncommon</span> <span style="color:red">Red = Common</span></v-flex>
        <v-flex xs6>Darker Color = Rarer | Rarity Calculated Off Total # of Attributes</v-flex>
      </v-layout>
    </v-container>
    <v-container v-if="!allYourKitties">
      <v-progress-linear :indeterminate="true"></v-progress-linear>
    </v-container>
    <v-container v-else fluid grid-list-xl>
      <v-layout row wrap>
        <v-flex v-if="allYourKitties && allYourKitties.length > 0" :key="y" v-for="(kitty, y) in allYourKitties" xs12 sm6 md4 offset-xs2 offset-sm0 offset-md0 offset-lg0>
          <v-card>
            <v-card-media
              :src="kitty.image_url"
              height="400px"
            ></v-card-media>
            <v-card-title primary-title>
              <v-container>
                <v-layout row wrap>
                  <div v-if="kitty && kitty.name" class="headline">{{ kitty.name }} - {{ kitty.id }}</div>
                  <div v-else class="headline">{{ kitty.id }}</div>
                </v-layout>
                <v-layout row>
                  <div class="grey--text">Generation: {{ kitty.generation }} - {{ kitty.status.cooldown_index | verbose }}</div>
                </v-layout>
              </v-container>
              <v-container>
                <v-layout v-show="kitty && kitty.cattributes" row wrap>
                  <v-flex>
                    <div slot="activator" class="cap" :key="j" v-for="(cattribute, j) in kitty.cattributes">
                      {{ cattribute.type }}:
                      <span v-if="cattribute && cattribute.total && (Number(cattribute.total) > 5000 && Number(cattribute.total) <= 50000)" :style="{ color: computedColor(cattribute.total, 'uncommon') }">
                        {{ cattribute.description }}
                      </span> 
                      <span v-else-if="cattribute && cattribute.total && (Number(cattribute.total) > 50000 && Number(cattribute.total) <= 100000)" :style="{ color: computedColor(cattribute.total, 'uncommon') }">
                        {{ cattribute.description }}
                      </span> 
                      <span v-else-if="cattribute && cattribute.total && Number(cattribute.total) <= 5000" :style="{ color: computedColor(cattribute.total, 'rare') }">
                        {{ cattribute.description }}
                      </span> 
                      <span v-else :style="{ color: computedColor(cattribute.total, 'common') }">
                        {{ cattribute.description }}
                      </span> 
                    </div>
                  </v-flex>
                  <v-flex>
                    <div v-if="kitty && kitty.auction && kitty.auction.status === 'open'"><h3>Current Auction:</h3>
                      <div>Time Duration: {{ kitty.auction.end_time | timeLeft }}</div>
                      <div>Auction Id: {{ kitty.auction.id }}</div>
                      <div>Start Price ETH: {{ kitty.auction.start_price | eth }}</div>
                      <div>End Price ETH: {{ kitty.auction.end_price | eth }}</div>
                      <div>Current Price ETH: {{ kitty.auction.end_price | eth }}</div>
                      <div>Seller Address: </div>
                      <div style="font-size: 12px;">{{ kitty.auction.seller.address }}</div>
                    </div>
                    <div v-else>No Auction in Progress</div>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card-title>
            <v-card-actions>
              <v-btn :href="'https://www.cryptokitties.co/kitty/' + kitty.id" target="_blank" flat>Official</v-btn>
              <v-btn :href="'https://cryptokittydex.com/kitties/' + kitty.id" target="_blank" flat color="purple">CryptoKittyDex</v-btn>
              <v-spacer></v-spacer>
              <v-btn icon @click.native="show = !show">
                <v-icon>{{ show ? 'keyboard_arrow_down' : 'keyboard_arrow_up' }}</v-icon>
              </v-btn>
            </v-card-actions>
            <v-slide-y-transition>
              <v-card-text v-show="show">
                {{ kitty.bio }}
              </v-card-text>
            </v-slide-y-transition>
            </v-card>
          </v-flex>
          <v-flex v-else>Hi</v-flex>
        </v-layout>
    </v-container>
  </v-container>
</template>

<script>
import axios from 'axios';
import moment from 'moment';

export default {
  name: 'MyKitties',
  filters: {
    timeLeft(val) {
      const currentTime = moment().toDate().getTime();
      return `Approximately ${moment(Number(val)).diff(currentTime, 'hours')} hours`;
    },
    verbose(val) {
      if (val === 1) return 'Fast';
      else if (val === 2) return 'Swift';
      else if (val === 3) return 'Snappy';
      else if (val === 4) return 'Brisk';
      else if (val === 5) return 'Plodding';
      else if (val === 6) return 'Slow';
      else if (val === 7) return 'Sluggish';
      return 'Catatonic';
    },
    eth(val) {
      if (!val) return '';
      return val * 0.000000000000000001;
    },
  },
  data() {
    return {
      gArr: [],
      gKArr: [],
      show: false,
      ETH_ADDRESS: '',
      myKitties: [],
      myKittyAttr: [],
      myKittiesMap: new Map(),
      cattributes: [],
      allYourKitties: null,
      totalCats: 0,
      currentTime: 0,
      arr: [],
    };
  },
  methods: {
    computedColor(val, rarityCategory) {
      const colorPercent = (val / this.totalCats) * 100;
      if (rarityCategory === 'rare') {
        return `hsl(233, 100%, ${(colorPercent) + 30}%)`;
      } else if (rarityCategory === 'uncommon') {
        return `hsl(120, 100%, ${(colorPercent) + 30}%)`;
      } else if (rarityCategory === 'common') {
        return `hsl(7, 100%, ${(colorPercent) + 30}%)`;
      }
      return `hsl(7, 100%, ${(colorPercent) + 30}%)`;
    },
    searchKitty(val) {
      const LIMIT = 1000;
      const finalKArr = [];
      axios.get('https://api.cryptokitties.co/kitties').then((data) => {
        this.totalCats = data.data.total;
      }).then(() => {
        axios.get('https://api.cryptokitties.co/cattributes?total=true').then((data) => {
          this.cattributes = data.data;
        });
      }).then(() => {
        axios.all([
          axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${val.target.value}&limit=${LIMIT}&offset=0`),
          axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${val.target.value}&limit=${LIMIT}&offset=20`),
          axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${val.target.value}&limit=${LIMIT}&offset=40`),
          axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${val.target.value}&limit=${LIMIT}&offset=60`),
          axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${val.target.value}&limit=${LIMIT}&offset=80`),
        ])
      .then(axios.spread((...args) => {
        args.forEach((chunk) => {
          finalKArr.push(...chunk.data.kitties);
        });
        return finalKArr;
      }))
      .then((allYourKitties) => {
        allYourKitties.forEach((kitty) => {
          const cattributes = this.cattributes;
          axios.get(`https://api.cryptokitties.co/kitties/${kitty.id}`)
            .then((attributeData) => {
              Object.assign(kitty, attributeData.data);
              kitty.cattributes.forEach((cattribute) => {
                const filtered = cattributes.filter(match => (
                  match.description === cattribute.description));
                Object.assign(cattribute, { total: filtered[0].total });
              });
            })
            .catch((err) => {
              console.error('err', err);
            });
        });
        this.allYourKitties = allYourKitties;
      });
      });
    },
  },
  mounted() {
    const ETH_PUBLIC_TOKEN = window.ETH_ADDRESS; // eslint-disable-line
    const LIMIT = 1000;
    const finalKArr = [];
    axios.get('https://api.cryptokitties.co/kitties').then((data) => {
      this.totalCats = data.data.total;
    }).then(() => {
      axios.get('https://api.cryptokitties.co/cattributes?total=true').then((data) => {
        this.cattributes = data.data;
      });
    }).then(() => {
      axios.all([
        axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${ETH_PUBLIC_TOKEN}&limit=${LIMIT}&offset=0`),
        axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${ETH_PUBLIC_TOKEN}&limit=${LIMIT}&offset=20`),
        axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${ETH_PUBLIC_TOKEN}&limit=${LIMIT}&offset=40`),
        axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${ETH_PUBLIC_TOKEN}&limit=${LIMIT}&offset=60`),
        axios.get(`https://api.cryptokitties.co/kitties?owner_wallet_address=${ETH_PUBLIC_TOKEN}&limit=${LIMIT}&offset=80`),
      ])
      .then(axios.spread((...args) => {
        args.forEach((chunk) => {
          finalKArr.push(...chunk.data.kitties);
        });
        return finalKArr;
      }))
      .then((allYourKitties) => {
        allYourKitties.forEach((kitty) => {
          const cattributes = this.cattributes;
          axios.get(`https://api.cryptokitties.co/kitties/${kitty.id}`)
            .then((attributeData) => {
              Object.assign(kitty, attributeData.data);
              kitty.cattributes.forEach((cattribute) => {
                const filtered = cattributes.filter(match => (
                  match.description === cattribute.description));
                Object.assign(cattribute, { total: filtered[0].total });
              });
            })
            .catch((err) => {
              console.error('err', err);
            });
        });
        this.allYourKitties = allYourKitties;
      });
    });
  },
};
</script>

<style lang="scss" scoped>
  .cap {
    text-transform: capitalize;
  }
</style>
