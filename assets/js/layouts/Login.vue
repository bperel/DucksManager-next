<template>
  <form method="post">
    <b-row>
      <b-col lg="6">
        <h1 class="h3 mb-3 font-weight-normal">
          {{ $t("Connexion") }}
        </h1>
        <b-alert
          v-if="error"
          show
          variant="danger"
        >
          {{ error }}
        </b-alert>
        <b-form-input
          id="username"
          name="username"
          type="text"
          required
          autofocus
          :value="lastUsername"
          :placeholder="$t(`Nom d'utilisateur`)"
        />
        <b-form-input
          id="password"
          name="password"
          type="password"
          required
          :placeholder="$t('Mot de passe')"
        />

        <input
          type="hidden"
          name="_csrf_token"
          :value="csrfToken"
        >

        <b-btn
          variant="primary"
          size="xl"
          type="submit"
        >
          {{ $t("Connexion") }}
        </b-btn>
        <div>
          <a :href="$r('/forgot')">{{ $t("Mot de passe oublié ?") }}</a>
        </div>
      </b-col>
    </b-row>
  </form>
</template>

<script>
import l10nMixin from "../mixins/l10nMixin";

export default {
  name: "Login",
  mixins: [l10nMixin],
  props: {
    error: { type: String, default: null },
    lastUsername: { type: String, default: null }
  },
  data() {
    return {
      csrfToken: document.getElementById("csrf").value
    };
  }
};
</script>

<style scoped>

</style>
