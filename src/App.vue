<template>
  <div id="app">
    <b-container fluid>
      <app-header
        :ordersTableItems="ordersTableItems"
        :statusLabels="statusLabels"
        :badgeVariants="badgeVariants"
      ></app-header>
      <orders-table
        :items="ordersTableItems"
        :statusLabels="statusLabels"
        :badgeVariants="badgeVariants"
        :ordersByReference="ordersByReference"
      ></orders-table>
      <b-alert
        :variant="connectionVariant[eventSourceStatus]"
        show
      >Connection status: {{ connectionStatus[eventSourceStatus] }}</b-alert>
    </b-container>
  </div>
</template>

<script>
import OrdersTable from "./components/OrdersTable.vue";
import AppHeader from "./components/Header.vue";

import ReconnectingEventSource from "reconnecting-eventsource"; //implements autoreconnection
// see https://github.com/fanout/reconnecting-eventsource

export default {
  name: "app",
  data() {
    return {
      ordersByReference: {},
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
      badgeVariants: {
        CREATED: "primary",
        TRANSMITTED: "secondary",
        IN_PREPARATION: "warning",
        PREPARED: "info",
        SHIPPED: "dark",
        DELIVERY_EXCEPTION: "danger",
        DELIVERED: "success",
        UPDATE_DATA: "light"
      },
      eventSourceStatus: null,
      connectionStatus: [
        "Connecting...",
        "Connected",
        "No connection",
        "Error, trying to reconnect"
      ],
      connectionVariant: ["primary", "success", "warning", "danger"]
    };
  },
  computed: {
    ordersTableItems() {
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
      let orderEventSource = new ReconnectingEventSource(
        "http://localhost:8080",
        {
          withCredentials: false,
          max_retry_time: 3000 // maximum time to wait before attempting to reconnect, in ms
        }
      );
      this.eventSourceStatus = orderEventSource.readyState; // CONNECTING (0)

      orderEventSource.onopen = event => {
        this.eventSourceStatus = event.target.readyState; // OPEN (1)
      };

      orderEventSource.addEventListener(
        "order_event",
        event => {
          let data = JSON.parse(event.data);
          this.getOrderData(data);
        },
        false
      );

      orderEventSource.onerror = event => {
        this.eventSourceStatus = 3; // custom error status
        if (event.target.readyState == EventSource.CLOSED) {
          this.eventSourceStatus = event.target.readyState; // CLOSED (2)
        }
      };
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

      this.addOrderToReferenceData(order, reference);
    },
    addOrderToReferenceData(order, reference) {
      if (this.ordersByReference[reference]) {
        this.ordersByReference[reference].history.push(order);
        this.ordersByReference[reference].lastEvent = order;
      } else {
        this.$set(this.ordersByReference, reference, { // $set is necessary to keep the ordersTableItems computed property reactive
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
