<template>
  <div id="app">
    <b-container fluid>
      <app-header :items="items" :statusLabels="statusLabels" :badgeColors="badgeColors"></app-header>
      <orders-table
        :items="items"
        :statusLabels="statusLabels"
        :badgeColors="badgeColors"
        :ordersByReference="ordersByReference"
      ></orders-table>
    </b-container>
  </div>
</template>

<script>
/* eslint-disable no-console */
import OrdersTable from "./components/OrdersTable.vue";
import AppHeader from "./components/Header.vue";

export default {
  name: "app",
  data() {
    return {
      ordersByReference: {}, // keys are the references, values are arrays of all order events of this reference
      orderEvents: [], // all order events (never used currently)
      statusLabels: {
        CREATED: "Created",
        TRANSMITTED: "Transmitted",
        IN_PREPARATION: "In preparation",
        PREPARED: "Prepared",
        SHIPPED: "Shipped",
        DELIVERY_EXCEPTION: "Delivery Exception",
        DELIVERED: "Delivered",
        UPDATE_DATA: "Updata data"
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
      },
      selectedStatus: "DELIVERY_EXCEPTION"
    };
  },
  computed: {
    items() {
      let items = [];
      for (let reference in this.ordersByReference) {
        items.push(this.ordersByReference[reference].lastEvent);
      }
      return items;
    }
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
      const localTime = time.toLocaleString("en-GB");

      if (data.payload.subtype === "status_update") {
        description = data.payload.description;
        statusLabel = this.statusLabels[data.payload.short];
      } else {
        description = `Update ${data.payload.short}: ${
          data.payload.description
        }`;
        status = "UPDATE_DATA";
        statusLabel = "Updating data";
      }

      let order = {
        operator: data.payload.operator,
        reference,
        description,
        status,
        statusLabel,
        localTime,
        time
      };

      this.orderEvents.push(order);
      this.addOrderToReferenceData(order, reference);
    },
    addOrderToReferenceData(order, reference) {
      if (this.ordersByReference[reference]) {
        this.ordersByReference[reference].history.push(order);
        this.ordersByReference[reference].lastEvent = order;
      } else {
        this.$set(this.ordersByReference, reference, {
          reference,
          history: [order],
          lastEvent: order
        });
      }
    }
  },
  components: {
    OrdersTable,
    AppHeader
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
