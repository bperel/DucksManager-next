<template>
  <div v-if="collection && l10n">
    <b-alert variant="info">
      {{ l10n.EXPLICATION_NOTATION_AUTEURS1 }}
      <a href="/stats/authors">{{ l10n.EXPLICATION_NOTATION_AUTEURS2 }}</a>
      {{ l10n.EXPLICATION_NOTATION_AUTEURS3 }}
      <br><br>
      {{ l10n.SUGGESTIONS_ACHATS_QUOTIDIENNES }}
    </b-alert>
    <div v-if="watchedAuthors.length && watchedAuthorsWithNotation.length">
      {{ l10n.MONTRER_MAGAZINES_PAYS }}
      <b-select
        v-if="countryNames"
        v-model="countryCode"
        :options="countryNames"
      />
      <SuggestionList
        :countrycode="countryCode"
        :since-last-visit="false"
      />
    </div>
    <b-alert
      v-else-if="!watchedAuthorsWithNotation.length"
      variant="warning"
    >
      {{ l10n.AUTEURS_NON_NOTES }}
    </b-alert>
    <b-alert
      v-else
      variant="warning"
    >
      {{ l10n.AUCUN_AUTEUR_NOTE_1 }}
      <span
        v-html="$t('AUCUN_AUTEUR_NOTE_2_REDIRECTION', [
          `<a href='/stats/authors'>${l10n.AUTEURS_COURT}</a>`
        ])"
      />
      {{ l10n.AUCUN_AUTEUR_NOTE_3 }}
    </b-alert>
  </div>
</template>

<script>
import collectionMixin from "../mixins/collectionMixin";
import l10nMixin from "../mixins/l10nMixin";
import {mapActions, mapGetters, mapState} from "vuex";
import SuggestionList from "./SuggestionList";

export default {
  name: "Expand",
  components: {SuggestionList},
  mixins: [collectionMixin, l10nMixin],

  data: () => ({
    countryCode: null
  }),

  computed: {
    ...mapState("collection", ["watchedAuthors", "suggestions"]),
    ...mapState("coa", ["countryNames"]),
    ...mapGetters("collection", ["hasSuggestions"]),

    imagePath: () => window.imagePath,

    watchedAuthorsWithNotation() {
      return this.watchedAuthors && this.watchedAuthors.filter(({notation}) => notation > 0)
    }
  },

  watch: {
    async watchedAuthors(newValue) {
      if (newValue && newValue.length) {
        await this.fetchCountryNames()
      }
    }
  },

  async mounted() {
    await this.loadWatchedAuthors()
  },

  methods: {
    ...mapActions("collection", ["loadWatchedAuthors", "loadSuggestions"]),
    ...mapActions("coa", ["fetchCountryNames"])
  }
}
</script>

<style scoped lang="scss">
  select {
    width: 300px;
  }
</style>