<template>
  <div
    class="fullscreen bg-primary text-white text-center q-pa-md flex flex-center"
  >
    <div>
      <div style="font-size: 10vh">404</div>

      <div class="text-h4" style="opacity: 0.4">Oops, nothing here:/</div>

      <q-btn
        class="q-mt-xl"
        color="white"
        text-color="primary"
        size="md"
        style="width: 10rem"
        unelevated
        :to="{ name: 'login' }"
        label="Go to top"
      />
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "ErrorNotFoundPage",
});
</script>
