<template>
  <v-container>
    <h1 class="vtitle">
      {{ $t('navbar-daily-store') }}
    </h1>
    <p>{{ $t('store-update') }}</p>
    <v-btn
      color="error"
      large
      outlined
      style="margin-left: 7 px;"
      @click="twitterShare()"
    >
      <v-icon>
        mdi-twitter
      </v-icon>
      {{ $t('store-share') }}
    </v-btn>
    <v-btn
      color="error"
      large
      text
      @click="viewdate()"
    >
      <v-icon left>
        mdi-clock-check-outline
      </v-icon>
      <h3 v-text="date" />
    </v-btn>
    <v-row class="pt-5">
      <v-col v-for="weapon in storeoffers" v-bind:key="weapon.name" cols="12" sm="3">
        <v-card
          class="mx-auto c1"
          max-width="100%"
          hover
          @click="viewskin(weapon.uuid)"
        >
          <v-img
            :src="/images/ + weapon.tierid + '.png'"
            height="250px"
            aspect-ratio="1.4"
            contain
          >
            <v-img
              :src="weapon.imgsrc"
              height="250px"
              class="align-end"
              aspect-ratio="1.4"
              contain
            >
              <v-card-title class="valofont">
                {{ weapon.name }}
              </v-card-title>
              <v-card-subtitle class="text-h6 text-decoration-underline text-right">
                {{ weapon.vp }} VP
              </v-card-subtitle>
            </v-img>
          </v-img>
        </v-card>
      </v-col>
    </v-row>
    <h1 v-if="bonusoffers[0] !== undefined" class="vtitle">
      {{ $t('store-night-market') }}
    </h1>
    <v-row v-if="bonusoffers[0] !== undefined">
      <v-col v-for="weapon in bonusoffers" v-bind:key="weapon.name" cols="6">
        <v-card
          class="mx-auto c2"
          max-width="100%"
          hover
          @click="viewskin(weapon.uuid)"
        >
          <v-img
            :src="/images/ + weapon.tierid + '.png'"
            height="250px"
            aspect-ratio="1.4"
            contain
          >
            <v-img
              :src="weapon.imgsrc"
              height="250px"
              class="align-end"
              aspect-ratio="1.4"
              contain
            >
              <v-card-title class="valofont">
                {{ weapon.name }}
              </v-card-title>
              <v-card-subtitle class="text-h6 text-decoration-underline text-right">
                {{ weapon.vpold }} VP -> {{ weapon.vp }} VP
              </v-card-subtitle>
            </v-img>
          </v-img>
        </v-card>
      </v-col>
    </v-row>
    <h2 v-else style="margin-top: 10px;">
      {{ $t('store-night-market-close') }}
    </h2>
    <adsbygoogle
      :ad-slot="'3401103391'"
      :ad-format="'autorelaxed'"
    />
  </v-container>
</template>

<script>
export default {
  name: 'StorePage',
  middleware: ['auth', 'maintenance'],

  data () {
    return {
      storeoffers: [],
      bonusoffers: [],
      date: undefined
    }
  },

  head () {
    return {
      title: 'Daily Store'
    }
  },

  mounted () {
    this.$swal.showLoading()
    this.loadStores()
  },

  methods: {
    async loadStores () {
      const response = await this.$axios.get(`${this.$config.API_BASE}/valorant/getshop`, { headers: { discordid: this.$store.state.auth.user.id } })
      if (response.data.status !== undefined) {
        if (response.data.status === 'FAILED') {
          this.$swal({
            icon: 'error',
            title: 'Oops...',
            text: this.$t('not-link')
          })
          this.$router.go(-1)
        } else {
          this.$swal({
            icon: 'error',
            title: 'Oops...',
            text: response.data.status
          })
        }
      } else {
        const response2 = await this.$axios.get(`${this.$config.API_BASE}/valorant/getnightmarket`, { headers: { discordid: this.$store.state.auth.user.id } })
        this.setStores(response.data)
        this.setNightMarket(response2.data)
      }
    },

    getDateFrom (date) {
      const diff = new Date().getTime() - date
      const elapsed = new Date(diff)

      if (date === undefined) {
        return 'Unknown'
      }

      if (elapsed.getUTCFullYear() - 1970) {
        return elapsed.getUTCFullYear() - 1970 + ' years ago'
      } else if (elapsed.getUTCMonth()) {
        return elapsed.getUTCMonth() + ' months ago'
      } else if (elapsed.getUTCDate() - 1) {
        return elapsed.getUTCDate() - 1 + ' days ago'
      } else if (elapsed.getUTCHours()) {
        return elapsed.getUTCHours() + ' hours ago'
      } else if (elapsed.getUTCMinutes()) {
        return elapsed.getUTCMinutes() + ' minutes ago'
      } else if (elapsed.getUTCSeconds()) {
        return elapsed.getUTCSeconds() + ' seconds ago'
      } else {
        return 'just now'
      }
    },

    setStores (stores) {
      for (const k in stores) {
        if ('offerleft' in stores[k]) {
          this.date = this.getDateFrom(stores[k].date * 1000)
          continue
        }
        this.storeoffers.push(
          { uuid: stores[k].uuid, vp: stores[k].vp, name: stores[k].name, imgsrc: stores[k].imgsrc, videosrc: stores[k].videosrc, tierid: stores[k].tierid }
        )
      }
      this.$swal.close()
    },

    setNightMarket (stores) {
      for (const k in stores) {
        if ('bonusleft' in stores[k]) {
          continue
        }
        this.bonusoffers.push(
          { uuid: stores[k].uuid, vp: stores[k].vp, vpold: stores[k].vpold, name: stores[k].name, imgsrc: stores[k].imgsrc, videosrc: stores[k].videosrc, tierid: stores[k].tierid }
        )
      }
    },

    viewskin (uuid) {
      window.open('/skin/' + uuid, '_self')
    },

    viewdate () {
      this.$swal({
        icon: 'info',
        title: 'Last updated time',
        text: this.date
      })
    },

    twitterShare () {
      const baseUrl = 'https://twitter.com/intent/tweet?'
      const text = ['text', 'Look at my store!' + '\n\n' + '#ValorantStoreChecker' + '\n']
      const url = ['url', 'https://valorantstore.net/stores/' + btoa(this.$store.state.auth.user.id)]
      const parameter = new URLSearchParams([text, url]).toString()
      const shareUrl = `${baseUrl}${parameter}`
      window.open(shareUrl, 'twitter', 'top=200,left=300,width=600,height=400')
    }
  }
}
</script>
<style scoped lang="css">
.valofont {
  color: #ff4655;
  font-family: 'VALORANT', sans-serif;
}

.vtitle {
  font-size: 5vh;
}
</style>
