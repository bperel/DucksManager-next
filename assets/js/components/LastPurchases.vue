<template>
  <Accordion
    v-if="collectionPerPurchaseDate"
    id="last-purchases"
    accordion-group-id="last-purchases"
  >
    <template #header>
      {{ $t("Derniers achats") }}
    </template>
    <template #content>
      <Accordion
        v-for="({purchase, issues}, purchaseIndex) in collectionPerPurchaseDate"
        :id="`purchase-accordion-${purchaseIndex}`"
        :key="`purchase-accordion-${purchaseIndex}`"
        :accordion-group-id="`purchase-accordion-${purchaseIndex}`"
        :visible="false"
      >
        <template #header>
          <b>{{ purchase.date }}</b> <i v-if="purchase.description">{{ purchase.description }} </i>{{ issues.length }} {{ $tc("numéro | numéros", issues.length) }}
        </template>
        <template #content>
          <Issue
            v-for="({publicationCode, issueNumber}, issueIndex) in issues"
            :key="`purchase-${purchaseIndex}-issue-${issueIndex}`"
            :publicationcode="publicationCode"
            :publicationname="publicationNames[publicationCode]"
            :issuenumber="issueNumber"
            :no-wrap="false"
          />
        </template>
      </Accordion>
    </template>
  </Accordion>
</template>
<script>
import { mapState } from "vuex";
import collectionMixin from "../mixins/collectionMixin";
import Accordion from "./Accordion";
import Issue from "./Issue";

export default {
  name: "LastPurchases",
  components: { Accordion, Issue },
  mixins: [collectionMixin],
  computed: {
    ...mapState("coa", ["publicationNames"]),
    ...mapState("collection", ["collection", "purchases"]),

    collectionPerPurchaseDate() {
      const vm = this;
      return this.collection && this.purchases && this.collection.reduce((acc, issue) => {
        const purchase = (issue.purchaseId > 0 && vm.purchases.find(({ id }) => id === issue.purchaseId)) || { date: issue.creationDate };
        let purchaseIndex = acc.findIndex(({ purchase: currentPurchase }) => JSON.stringify(currentPurchase) === JSON.stringify(purchase));
        if (purchaseIndex === -1) {
          acc.push({ purchase, issues: [] });
          purchaseIndex = acc.length - 1;
        }
        acc[purchaseIndex].issues.push(issue);
        return acc;
      }, [])
        .sort(({ purchase: purchase1 }, { purchase: purchase2 }) => purchase1.date < purchase2.date ? 1 : -1)
        .slice(0, 5);
    }
  }
};
</script>

<style scoped lang="scss">
::v-deep .card-body {
  color: black;
}
</style>
