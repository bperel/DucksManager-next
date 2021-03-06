<template>
  <b-navbar
    v-if="countryNames && hasPublicationNames"
    id="publication-list"
    toggleable="lg"
    type="dark"
    variant="dark"
    sticky
  >
    <b-navbar-toggle target="nav-publications" />
    <b-navbar-brand href="#">
      {{ $t('Collection') }}
    </b-navbar-brand>
    <b-collapse
      id="nav-publications"
      is-nav
    >
      <b-navbar-nav>
        <b-nav-item-dropdown
          v-for="country in sortedCountries"
          :key="country"
          :text="country"
        >
          <template #button-content>
            <Country
              :country="country"
              :country-name="countryNames[country]"
            >
              <template #default="props">
                {{ props.countryName }}
              </template>
            </Country>
          </template>
          <b-dropdown-item
            v-for="publicationCode in getSortedPublications(country)"
            :key="publicationCode"
            :href-l10n="`/collection/show/${publicationCode}`"
            :href="$r(`/collection/show/{publicationCode:${publicationCode}}`)"
          >
            {{ publicationNames[publicationCode] || publicationCode.split('/')[1] }}
          </b-dropdown-item>
        </b-nav-item-dropdown>
        <b-nav-item :href="$r('/collection/show/{publicationCode:new}')">
          {{ $t('Nouveau magazine') }}
        </b-nav-item>
      </b-navbar-nav>
    </b-collapse>
    <b-navbar-nav class="ml-auto">
      <b-nav-form>
        <IssueSearch :with-title="false" />
      </b-nav-form>
    </b-navbar-nav>
  </b-navbar>
  <div v-else>
    {{ $t('Chargement...') }}
  </div>
</template>

<script>
import Country from "./Country";
import l10nMixin from "../mixins/l10nMixin";
import {mapActions, mapGetters, mapState} from "vuex";
import IssueSearch from "./IssueSearch";

export default {
  name: "PublicationList",

  components: {
    IssueSearch,
    Country
  },

  mixins: [l10nMixin],

  data: () => ({
    hasPublicationNames: false
  }),

  computed: {
    ...mapGetters("collection", ["totalPerCountry", "totalPerPublication"]),
    ...mapState("coa", ["countryNames", "publicationNames"]),
    sortedCountries() {
      const vm = this
      return this.totalPerCountry && Object.keys(this.totalPerCountry)
          .sort((countryCode1, countryCode2) => vm.countryNames[countryCode1].localeCompare(vm.countryNames[countryCode2]))
    },
    publicationsPerCountry() {
      const vm = this
      return this.totalPerCountry && this.hasPublicationNames && Object.keys(this.totalPerCountry)
        .reduce((acc, country) => ({
          ...acc,
          [country]: Object.keys(vm.totalPerPublication).filter(publicationCode =>
            publicationCode.split('/')[0] === country
          )
        }), {})
    },
  },

  watch: {
    totalPerPublication: {
      immediate: true,
      async handler(newValue) {
        if (newValue) {
          await this.fetchPublicationNames(Object.keys(newValue))
          this.hasPublicationNames = true
        }
      }
    }
  },

  async mounted() {
    await this.fetchCountryNames()
  },

  methods: {
    ...mapActions("coa", ["fetchCountryNames", "fetchPublicationNames"]),
    getSortedPublications(country) {
      const vm = this
      return this.publicationsPerCountry && this.publicationsPerCountry[country]
        .sort((a, b) => vm.publicationNames[a] && vm.publicationNames[a].localeCompare(vm.publicationNames[b]))
    }
  }
}
</script>

<style scoped lang="scss">
.navbar {
  &#publication-list {
    margin-bottom: 20px;
  }

  .navbar-toggler {
    margin-right: 1rem;
  }

  .navbar-brand {
    min-width: 120px;
  }

  .navbar-nav {
    flex-wrap: wrap;

    ::v-deep ul {
      max-height: calc(100vh - 100px);
      z-index: 1030;
      overflow-y: auto;
    }
  }

  a {
    border: none;
  }
}
</style>
