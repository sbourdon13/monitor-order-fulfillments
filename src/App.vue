<template>
  <div id="app">
    <h1>Monitor your orders</h1>
    <!-- <b-table striped hover :items="orders"></b-table> -->
    <b-table striped hover responsive :items="items" :fields="fields">
      <!-- Display badges for the status column -->
      <template slot="status" slot-scope="row">
        <b-badge :variant="badgeColors[row.item.status]">{{ row.item.statusLabel }}</b-badge>
      </template>

      <!-- details of previous order events of the same reference -->
      <template slot="show_details" slot-scope="row">
        <b-button
          size="sm"
          @click="row.toggleDetails"
          class="mr-2"
        >{{ row.detailsShowing ? 'Hide' : 'Show'}} Details</b-button>
      </template>

      <template slot="row-details" slot-scope="row">
        <b-card>
          <b-row
            class="mb-2"
            v-for="(event, index) in ordersByReference[row.item.reference]"
            :key="index"
          >
            <b-col sm="3" class="text-sm-right">
              <b>{{ event.time.toLocaleString("en-GB") }}</b>
            </b-col>
            <b-col>{{ event.description }}</b-col>
            <b-col>{{ event.statusLabel }}</b-col>
          </b-row>
        </b-card>
      </template>
    </b-table>
  </div>
</template>

<script>
/* eslint-disable no-console */

export default {
  name: "app",
  data() {
    return {
      ordersByReference: {}, // keys are the references, values are arrays of all order events of this reference
      orderEvents: [], // all order events
      items: [], // orders to be displayed (one item per reference),
      fields: [
          { key: 'reference', sortable: false },
          { key: 'operator', sortable: true },
          { key: 'description', sortable: false },
          { key: 'status', sortable: true },
          { key: 'show_details', sortable: false }
        ],
      status: {
        CREATED: "CREATED",
        TRANSMITTED: "TRANSMITTED",
        IN_PREPARATION: "IN PREPARATION",
        PREPARED: "PREPARED",
        SHIPPED: "SHIPPED",
        DELIVERY_EXCEPTION: "DELIVERY EXCEPTION",
        DELIVERED: "DELIVERED"
      },
      badgeColors: {
        CREATED: "primary",
        TRANSMITTED: "secondary",
        IN_PREPARATION: "warning",
        PREPARED: "info",
        SHIPPED: "dark",
        DELIVERY_EXCEPTION: "danger",
        DELIVERED: "success",
        UPDATE_DATA: "light"
      }
    };
  },
  created() {
    this.setupStream();
  },
  methods: {
    setupStream() {
      let orderEventSource = new EventSource("http://localhost:8080");

      orderEventSource.addEventListener(
        "order_event",
        event => {
          let data = JSON.parse(event.data);
          this.getOrderData(data);
        },
        false
      );

      orderEventSource.addEventListener(
        "error",
        event => {
          console.log("EventSource failed.");
          if (event.readyState == EventSource.CLOSED) {
            console.log("Event was closed");
            console.log(EventSource);
          }
        },
        false
      );
    },
    getOrderData(data) {
      let status = data.payload.short;
      let statusLabel, description;
      const reference = data.payload.reference;
      const time = new Date(Date.parse(data.create_time));

      if (data.payload.subtype === "status_update") {
        description = data.payload.description;
        statusLabel = this.status[data.payload.short];
      } else {
        description = `Update ${data.payload.short}: ${
          data.payload.description
        }`;
        status = "UPDATE_DATA";
        statusLabel = "Updating data";
      }

      let order = {
        operator: data.payload.operator,
        time,
        reference,
        description,
        status,
        statusLabel
      };

      this.orderEvents.push(order);

      if (this.ordersByReference[reference]) {
        this.ordersByReference[reference].push(order);
        let index = this.items.findIndex(item => item.reference === reference);
        this.items[index] = order;
      } else {
        this.ordersByReference[reference] = [order];
        this.items.push(order);
      }
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
