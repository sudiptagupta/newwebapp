<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-dark text-white">
      <q-toolbar>
        <q-toolbar-title class="cursor-pointer" @click="goHome">
          Traddio
        </q-toolbar-title>

        <div class="q-mx-md">
          <q-btn
            flat
            label="Log in"
            to="/"
            class="text-white q-mb-md q-mr-md"
          />
          <q-btn
            flat
            label="Sign up"
            to="/register"
            class="bg-primary text-white q-mb-md"
          />
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "LoginLayout",

  setup() {
    // Navigate to home page when the title is clicked
    const goHome = () => {
      this.$router.push({ path: "/" });
    };

    return {
      goHome,
    };
  },
});
</script>

<style scoped>
.bg-dark {
  background-color: #121212; /* Dark background */
}

.text-white {
  color: #fff;
}

.bg-primary {
  background-color: #1976d2; /* Primary button color */
}

.q-toolbar-title {
  font-size: 1.5em;
}
</style>
