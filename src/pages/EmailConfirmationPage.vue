<template>
  <q-page padding>
    <p class="text-body1">
      <strong>{{ $route.query.name }}</strong
      >, Thank you for registering! ❤️
    </p>
    <p class="text-body1">
      Click the button below to confirm your registration with your email:
      <strong>{{ $route.query.email }}</strong
      >.
    </p>
    <div class="row justify-center">
      <div class="col-md-4 col-sm-6 col-xs-10 q-gutter-y-md">
        <q-btn
          label="Voltar para login"
          color="primary"
          class="full-width"
          :to="{ name: 'login' }"
        />
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "EmailConfirmationPage",
});
</script>
