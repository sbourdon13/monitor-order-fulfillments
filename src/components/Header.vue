<template>
  <div>
    <h1>Monitor your orders</h1>
    <b-card-group deck class="mx-2 my-4">
      <b-card
        v-for="statusData in cardsData"
        :key="statusData.status"
        :bg-variant="badgeVariants[statusData.status]"
        text-variant="white"
        :header="statusLabels[statusData]"
        class="text-center"
      >
        <h2>{{ statusData.number }}</h2>
        <p class="m-0">{{ statusData.text }}</p>
      </b-card>
    </b-card-group>
  </div>
</template>

<script>
export default {
  props: {
    ordersTableItems: Array,
    statusLabels: Object,
    badgeVariants: Object
  },
  computed: {
    cardsData() {
      return [
        {
          status: "TRANSMITTED",
          number: this.numberOfReferencesWithStatus("TRANSMITTED"),
          text: "transmitted order(s) must be prepared"
        },
        {
          status: "PREPARED",
          number: this.numberOfReferencesWithStatus("PREPARED"),
          text: "prepared order(s) must be shipped"
        },
        {
          status: "DELIVERY_EXCEPTION",
          number: this.numberOfReferencesWithStatus("DELIVERY_EXCEPTION"),
          text: "delivery exception(s)"
        },
        {
          status: "DELIVERED",
          number: this.numberOfReferencesWithStatus("DELIVERED"),
          text: "order(s) succesfully delivered"
        }
      ];
    }
  },
  methods: {
    numberOfReferencesWithStatus(status) {
      return this.ordersTableItems.filter(item => item.status === status).length;
    }
  }
};
</script>

<style lang="scss" scoped>
</style>