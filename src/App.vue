<template>
  <div id="app">
    <h1>Monitor your orders</h1>
    <!-- <b-table striped hover :items="orders"></b-table> -->
    <b-table striped hover :items="items"></b-table>
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
      items: [] // orders to be displayed (one item per reference)
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
      let status, description;
      let reference = data.payload.reference;

      if (data.payload.subtype === "status_update") {
        description = data.payload.description;
        status = data.payload.short;
      } else {
        description = `Update ${data.payload.short}: ${
          data.payload.description
        }`;
        status = "Updating data";
      }

      let order = {
        reference: reference,
        operator: data.payload.operator,
        description,
        status
      };

      this.orderEvents.push(order);

      if (this.ordersByReference[reference]) {
        this.ordersByReference[data.payload.reference].push(order);
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
