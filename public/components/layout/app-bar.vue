<template>
  <v-app-bar
    :class="themeColorDark ? 'area--dark' : 'area--light'"
    color="primary"
    height="64"
    style="max-height: 64px;"
  >
    <xs-menu :pages="pages" :extra-menus="extraMenus" />
    <v-tabs
      v-show="$vuetify.breakpoint.mdAndUp"
      background-color="primary"
      height="64"
      :dark="themeColorDark"
      centered
    >
      <v-tabs-slider :color="themeColorDark ? 'white' : textDark" />
      <v-tab
        :to="{name: 'index'}"
        class="font-weight-bold"
      >
        Accueil
      </v-tab>
      <v-tab
        v-if="!config.datasetsPage || config.datasetsPage.type !== 'none'"
        :to="{name: 'datasets'}"
        class="font-weight-bold"
      >
        Les données
      </v-tab>
      <v-tab
        v-if="!config.reusesPage || config.reusesPage.type !== 'none'"
        :to="{name: 'reuses'}"
        class="font-weight-bold"
      >
        Visualisations
      </v-tab>
      <template v-if="pages">
        <v-tab
          v-for="page in pages.filter(p => p.navigation && p.navigation.type === 'direct')"
          :key="page.id"
          :to="{name: 'pages-id', params: {id: page.id}}"
          class="font-weight-bold"
        >
          {{ page.title }}
        </v-tab>
        <v-menu
          v-for="menu in extraMenus"
          :key="menu"
          offset-y
          nudge-left
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              text
              v-bind="attrs"
              :height="64"
              v-on="on"
            >
              {{ menu }}
              <v-icon right>
                mdi-menu-down
              </v-icon>
            </v-btn>
          </template>
          <v-list>
            <v-list-item
              v-for="page in pages.filter(p => p.navigation && p.navigation.type === 'menu' && p.navigation.title === menu)"
              :key="page.id"
              :to="{name: 'pages-id', params: {id: page.id}}"
            >
              <v-list-item-title>{{ page.title }}</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
      <v-tab
        v-if="config.contactEmail"
        to="/contact"
        class="font-weight-bold"
      >
        Contact
      </v-tab>
      <!-- <v-tab v-if="user && user.isAdmin" :to="{name: 'config'}" class="font-weight-bold">
              Configuration
            </v-tab> -->
    </v-tabs>
    <v-spacer />

    <v-toolbar-items>
      <notifications-queue v-if="user && notifyUrl" :notify-url="notifyUrl" />
      <template v-if="initialized && config.authentication !== 'none'">
        <v-btn
          v-if="!user"
          depressed
          color="primary"
          @click="login(url)"
        >
          Se connecter
        </v-btn>
        <v-menu
          v-else
          offset-y
          nudge-left
        >
          <template v-slot:activator="{on}">
            <v-btn
              text
              :height="64"
              v-on="on"
            >
              <v-avatar :size="36">
                <img :src="`${directoryUrl}/api/avatars/user/${user.id}/avatar.png`">
              </v-avatar>
                  &nbsp;
              {{ user.name }}
              <v-icon right>
                mdi-menu-down
              </v-icon>
            </v-btn>
          </template>
          <v-list outlined>
            <v-list-item :href="dataFairUrl + '/'" :disabled="embed">
              <v-list-item-title>Back-office</v-list-item-title>
            </v-list-item>
            <v-divider />
            <v-list-item :to="{name: 'me'}" :disabled="embed">
              <v-list-item-title>Mon compte</v-list-item-title>
            </v-list-item>
            <!--<v-list-item dense>
              <v-list-item-title style="overflow: visible;">
                <v-switch
                  v-model="$vuetify.theme.dark"
                  hide-details
                  class="mt-0"
                  label="mode nuit"
                  color="white"
                  @change="setDarkCookie"
                />
              </v-list-item-title>
            </v-list-item>-->
            <v-list-item :disabled="embed" @click="logout">
              <v-list-item-title>Se déconnecter</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
    </v-toolbar-items>
  </v-app-bar>
</template>

<script>
  import XsMenu from '~/components/layout/xs-menu'
  import NotificationsQueue from '~/components/notifications-queue'
  const { mapState, mapGetters, mapActions } = require('vuex')

  export default {
    components: { XsMenu, NotificationsQueue },
    async fetch() {
      this.pages = (await this.$axios.$get(process.env.publicUrl + `/api/v1/portals/${this.portal._id}/pages`, { params: { size: 1000, select: 'id,title,navigation', published: true } })).results
    },
    data: () => ({
      pages: null,
    }),
    computed: {
      ...mapState(['config', 'textDark', 'portal']),
      ...mapState('session', ['user', 'initialized']),
      ...mapGetters(['themeColorDark', 'embed']),
      directoryUrl() {
        return process.env.directoryUrl
      },
      dataFairUrl() {
        return process.env.dataFairUrl
      },
      notifyUrl() {
        return process.env.notifyUrl
      },
      extraMenus() {
        return (this.pages || []).filter(p => p.navigation && p.navigation.type === 'menu').map(p => p.navigation.title).filter((m, i, s) => s.indexOf(m) === i)
      },
      url () {
        return window.location.href
      },
    },
    methods: {
      ...mapActions('session', ['logout', 'login']),
      reload() {
        window.location.reload()
      },
      setDarkCookie(value) {
        const maxAge = 60 * 60 * 24 * 100 // 100 days
        this.$cookies.set('theme_dark', '' + value, { path: '/', domain: process.env.sessionDomain, maxAge })
        this.reload()
      },
    },
  }
</script>

<style lang="css" scoped>
</style>
