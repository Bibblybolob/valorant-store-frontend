<template>
  <v-container>
    <h1 class="vtitle">
      {{ $t('navbar-link-manager') }}
    </h1>
    <v-row>
      <v-col cols="12" sm="6">
        <v-card
          elevation="2"
          class="pa-5"
          width="100vh"
          height="100%"
        >
          <h1>
            {{ $t('link-link-account') }}
          </h1>
          <v-alert
            type="warning"
            outlined
          >
            {{ $t('link-warning') }}
          </v-alert>
          <v-form ref="form">
            <v-text-field
              v-model="riotusername"
              label="Riot Account Name"
              :rules="[required]"
              filled
              clearable
              color="error"
            />
            <v-text-field
              v-model="riotuserpassword"
              label="Riot Account Password"
              :rules="[required]"
              filled
              clearable
              :append-icon="toggle.icon"
              :type="toggle.type"
              color="error"
              @click:append="show = !show"
            />
            <v-select
              v-model="riotregion"
              item-text="label"
              item-value="value"
              :items="regions"
              filled
              label="Select Riot Account Region"
              :rules="[required]"
              color="error"
            />
            <v-checkbox
              v-model="relogin"
              label="Enable or disable auto-login"
              color="error"
            ></v-checkbox>
            <v-btn
              elevation="2"
              color="error"
              large
              @click="link()"
            >
              Submit
            </v-btn>
          </v-form>
        </v-card>
      </v-col>
      <v-col cols="12" sm="6">
        <v-card
          elevation="2"
          class="pa-5"
          width="100vh"
          height="100%"
        >
          <h1>
            {{ $t('link-unlink-account') }}
          </h1>
          <v-alert
            type="warning"
            outlined
          >
            {{ $t('link-warning2') }}
          </v-alert>
          <v-btn
            elevation="2"
            block
            color="error"
            @click="unlink()"
          >
            Submit
          </v-btn>
        </v-card>
      </v-col>
    </v-row>
    <adsbygoogle
      :ad-slot="'9526693890'"
      :ad-format="'auto'"
    />
  </v-container>
</template>

<script>
export default {
  name: 'LinkManagerPage',
  middleware: ['auth', 'maintenance'],
  data () {
    return {
      regions: [
        {
          label: 'Asia_Pacific',
          value: 'ap'
        },
        {
          label: 'Europa',
          value: 'eu'
        },
        {
          label: 'North_America / Latin_America / Brazil',
          value: 'na'
        },
        {
          label: 'Korea',
          value: 'kr'
        }
      ],
      show: false,
      riotusername: null,
      riotuserpassword: null,
      riotregion: null,
      relogin: false,
      required: value => !!value || 'Please be sure to fill out the form or select.'
    }
  },

  head () {
    return {
      title: 'Link Manager'
    }
  },

  computed: {
    toggle () {
      const icon = this.show ? 'mdi-eye' : 'mdi-eye-off'
      const type = this.show ? 'text' : 'password'
      return { icon, type }
    }
  },

  methods: {
    async link () {
      this.$swal.showLoading()
      if (this.$refs.form.validate()) {
        const reg = /^[0-9a-zA-Z_]+$/
        if (!reg.test(this.riotusername)) {
          this.$swal({
            icon: 'error',
            title: 'Oops...',
            text: 'User name contains an unusable string or spaces.'
          })
          return
        }
        // Success
        const register = await this.$axios.get(`${this.$config.API_BASE}/valorant/login`, { headers: { password: this.riotuserpassword, username: this.riotusername, region: this.riotregion, discordid: this.$store.state.auth.user.id, relogin: this.relogin } })
        if (register.data.status === 'OK') {
          // Success
          this.$swal({
            icon: 'success',
            title: 'Success!',
            text: 'Link is complete!'
          })
        } else if (register.data.status === 'FAILED') {
          this.$swal({
            icon: 'error',
            title: 'Oops...',
            text: 'Already linked.'
          })
        } else if (register.data.status === 'auth_failure') {
          this.$swal({
            icon: 'error',
            title: 'Oops...',
            text: 'Wrong password or username.'
          })
        } else if (register.data.status === '2fa') {
          this.$swal({
            icon: 'error',
            title: 'Oops...',
            text: 'If two-factor authentication is enabled, you will not be able to login via the web; you must login via the Discord bot.'
          })
        } else {
          this.$swal({
            icon: 'error',
            title: 'Unknown Error',
            text: register.data.status
          })
        }
      } else {
        this.$swal({
          icon: 'error',
          title: 'Oops...',
          text: 'Please fill out the form and make your selection.'
        })
      }
    },

    async unlink () {
      this.$swal.showLoading()
      const unlink = await this.$axios.get(`${this.$config.API_BASE}/valorant/logout`, { headers: { discordid: this.$store.state.auth.user.id } })
      if (unlink.data.status === 'OK') {
        // Success
        this.$swal({
          icon: 'success',
          title: 'See you again!',
          text: 'Unlink is complete!'
        })
      } else {
        // Failed
        this.$swal({
          icon: 'error',
          title: 'Oops...',
          text: unlink.data.status
        })
      }
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
