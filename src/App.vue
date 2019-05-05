<template>
  <div id="app">
    <h1>Monitor your orders</h1>
    <b-table striped hover :items="orders"></b-table>
  </div>
</template>

<script>
/* eslint-disable no-console */

export default {
  name: "app",
  data() {
    return {
      orders: []
    };
  },
  created() {
    this.setupStream();
  },
  methods: {
    setupStream() {
      let es = new EventSource("http://localhost:8080", {
        withCredentials: false
      });

      es.addEventListener(
        "order_event",
        event => {
          let data = JSON.parse(event.data);
          this.getOrderData(data);
        },
        false
      );

      es.addEventListener(
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
        reference: data.payload.reference,
        operator: data.payload.operator,
        description,
        status
      };
      this.orders.push(order);
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
