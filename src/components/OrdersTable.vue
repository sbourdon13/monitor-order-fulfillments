<template>
  <div>
    <b-table
      striped
      hover
      responsive
      small
      :items="items"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      class="p-2"
    >
      <!-- Display badges for the status column -->
      <template slot="status" slot-scope="row">
        <b-badge :variant="badgeVariants[row.item.status]">{{ row.item.statusLabel.toUpperCase() }}</b-badge>
      </template>

      <!-- create a column of toggle buttons to display details  -->
      <template slot="showDetails" slot-scope="row">
        <b-button
          size="sm"
          @click="row.toggleDetails"
          class="mr-2"
        >{{ row.detailsShowing ? 'Hide' : 'Show'}} Details</b-button>
      </template>

      <!-- show details of previous order events of the same reference  -->
      <template slot="row-details" slot-scope="row">
        <b-card>
          <b-row
            class="mb-2"
            v-for="(event, index) in ordersByReference[row.item.reference].history"
            :key="index"
          >
            <b-col sm="3" class="text-sm-right">
              <b>{{ event.localTime }}</b>
            </b-col>
            <b-col>{{ event.description }}</b-col>
            <b-col>
              <b-badge :variant="badgeVariants[event.status]">{{ event.statusLabel.toUpperCase() }}</b-badge>
            </b-col>
          </b-row>
        </b-card>
      </template>

      <!-- Table caption (number of references in the table) -->
      <template slot="table-caption">
        <div class="mx-4">{{ items.length }} reference(s).</div>
      </template>
    </b-table>

    <!-- pagination -->
    <b-row>
      <b-col>
        <b-pagination
          v-model="currentPage"
          :total-rows="items.length"
          :per-page="perPage"
          align="center"
          v-if="items.length > perPage"
        ></b-pagination>
      </b-col>
    </b-row>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fields: [
        { key: "reference", sortable: false },
        { key: "status", sortable: true },
        { key: "description", sortable: false },
        { key: "operator", sortable: true },
        { key: "localTime", sortable: true },
        { key: "showDetails", sortable: false }
      ],
      currentPage: 1,
      perPage: 20
    };
  },
  props: {
    items: Array,
    ordersByReference: Object,
    statusLabels: Object,
    badgeVariants: Object
  }
};
</script>

<style lang="scss" scoped>
</style>