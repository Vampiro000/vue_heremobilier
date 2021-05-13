<template>
  <!-- <hello-world /> -->
  <div>
    <v-flex>
      <div style="position: fixed; bottom: 0px; width: 100%">
        <v-tabs v-model="tab" centered grow background-color="indigo" dark>
          <v-tab> <v-icon>mdi-home</v-icon> </v-tab>
          <v-tab> <v-icon>mdi-map-search</v-icon> </v-tab>
          <v-tab> <v-icon>mdi-map-marker-star</v-icon> </v-tab>
        </v-tabs>
      </div>
      <v-tabs-items v-model="tab">
        <v-tab-item>
          <div>Acceuil</div>
        </v-tab-item>
        <v-tab-item>
          <div><SearchPage /></div>
        </v-tab-item>
        <v-tab-item>
          <div></div>
        </v-tab-item>
      </v-tabs-items>
    </v-flex>
  </div>
</template>


<script>
import HelloWorld from "../components/HelloWorld";
import SearchPage from "../components/SearchPage";
export default {
  name: "Home",
  data: () => ({
    tab: null,
    text: "gello",
  }),
  components: {
    HelloWorld,
    SearchPage,
  },
};
</script>
