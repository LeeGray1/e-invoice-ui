<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      :clipped="$vuetify.breakpoint.lgAndUp"
      app
    >
      <NavDrawer />
    </v-navigation-drawer>

    <v-app-bar
      :clipped-left="$vuetify.breakpoint.lgAndUp"
      app
      color="blue darken-3"
      dark
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title style="width: 300px" class="ml-0 pl-4">
        <span class="hidden-sm-and-down">eInvoice Translator</span>
      </v-toolbar-title>
      <v-spacer />
      <v-col cols="5" sm="3" md="3">
        <v-select
          flat
          solo-inverted
          hide-details
          v-model="selectedLocaleCode"
          label="Select App Language"
          :items="nativelanguages"
          item-text="name"
          item-value="code"
          @change="localeCodeSelected"
        ></v-select>
      </v-col>

      <div v-if="showNav">
        {{ $auth.user.email }}
        <v-btn name="logout" icon tile @click="logout">
          <v-icon>mdi-logout</v-icon>
        </v-btn>
      </div>
      <v-btn color="primary" v-else @click="login">Login</v-btn>
    </v-app-bar>
    <v-main>
      <!--<v-label>{{$t('UploadXMLTxt')}}</v-label>-->
      <v-container fluid>
        <router-view />
      </v-container>
    </v-main>
    <v-footer app></v-footer>
  </v-app>
</template>

<script>
import NavDrawer from "@/components/nav-drawer";
import { call, get } from "vuex-pathify";

//import { LOCALES, Locales } from "@/i18n/locales";
//import { defaultLocale } from "@/i18n";

export default {
  components: {
    NavDrawer,
  },
  data: () => ({
    drawer: null,
    selectedLocaleCode: "",
  }),
  async created() {
    await this.setLocalCode();
    await this.loadNativeLanguages();
    await this.localeCodeSelected();
  },

  methods: {
    async setLocalCode() {
      console.log("Browser Language = " + navigator.language.substr(0, 2));
      if (localStorage.localeCode) {
        this.selectedLocaleCode = localStorage.localeCode;
      } else {
        //get browser locale
        this.selectedLocaleCode = navigator.language.substr(0, 2);
      }
      console.log("This selected code: " + this.selectedLocaleCode);
      let appId = 0;
      if (localStorage.appIdentifier) {
        appId = localStorage.appIdentifier;
        console.log(
          "App Identifier in local storage " + localStorage.appIdentifier
        );
      } else {
        localStorage.appIdentifier = Math.floor(Math.random() * 100000000 + 1);
      }
      console.log("App Identifier " + localStorage.appIdentifier);
    },

    ...call("languageStore", ["fillInvoiceWords"]),

    async SetInvoiceWords() {
      await this.fillInvoiceWords();
    },

    ...call("languageStore", [
      "loadNativeLanguages",
      "selectLocaleCode",
      "loadLanguages",
      "fillLocaleWords",
      "SetLocaleWordArray",
    ]),

    async localeCodeSelected() {
      await this.selectLocaleCode(this.selectedLocaleCode),
        await this.loadLanguages();
      await this.fillLocaleWords();
      await this.SetLocaleWordArray();
      await this.fillInvoiceWords();

      localStorage["localeCode"] = this.selectedLocaleCode;
    },

    login() {
      this.$auth.loginWithRedirect();
    },
    logout() {
      this.$auth.logout({
        returnTo: window.location.origin,
      });
    },
  },

  computed: {
    nativelanguages: get("languageStore/nativeLanguages"),
    localeCode: get("languageStore/selectedLocaleCode"),
    ...get("languageStore"),

    showNav() {
      if (this.$auth && this.$auth.isAuthenticated) return true;
      return false;
    },
  },
};
</script>