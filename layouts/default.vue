<template>
  <v-app dark>
    <v-app-bar elevation="0" fixed app color="#0E131F">
      <v-spacer />
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-main style="background-color: #111827">
      <Nuxt />
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "DefaultLayout",
  data() {
    return {
      title: "A simple API tester",
    };
  },
};
</script>
