<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="UCAB Logo"
          class="shrink mr-2"
          contain
          src="@/assets/logoUCAB.png"
          transition="scale-transition"
          width="40"
        />

        <v-toolbar-title transition="scale-transition"
          >Manejador de objetos</v-toolbar-title
        >
      </div>
    </v-app-bar>

    <v-main>
      <HelloWorld />
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from "./components/HelloWorld";

export default {
  name: "App",

  components: {
    HelloWorld,
  },

  data: () => ({
    //
  }),
};
</script>
