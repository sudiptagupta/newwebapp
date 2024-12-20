<template>
  <q-page class="bg-dark row justify-center items-center">
    <div class="auth-container">
      <div class="text-center q-mb-md">
        <h4 class="text-white text-h5">Sign up</h4>
      </div>
      <q-form class="auth-form" @submit.prevent="handlerRegister" ref="myform">
        <div class="q-gutter-y-md">
          <q-input
            v-model="form.name"
            label="Name"
            dark
            outlined
            class="auth-input"
            :rules="[(val) => (val && val.length > 0) || 'Name is required!']"
          />

          <q-input
            v-model="form.email"
            label="Email"
            type="email"
            dark
            outlined
            class="auth-input"
            :rules="[
              (val) => (val && val.length > 0) || 'Email is required!',
              isValidEmail,
            ]"
          />

          <q-input
            v-model="form.password"
            label="Password"
            dark
            outlined
            class="auth-input"
            :type="visibility"
            :rules="[
              (val) => (val && val.length > 0) || 'Password is required.',
            ]"
          >
            <template v-slot:append>
              <q-icon
                :name="
                  visibility === 'password' ? 'visibility_off' : 'visibility'
                "
                class="cursor-pointer"
                @click="
                  visibility = visibility === 'password' ? 'text' : 'password'
                "
              />
            </template>
          </q-input>

          <q-btn
            label="Sign Up"
            type="submit"
            color="white"
            text-color="black"
            class="full-width"
            size="lg"
          />

          <div class="text-center text-grey q-my-md">OR</div>

          <q-btn
            color="dark"
            class="full-width google-btn"
            size="lg"
            flat
            @click="handleGoogleSignUp"
          >
            <template v-slot:default>
              <div class="row full-width items-center justify-between">
                <q-icon
                  name="img:https://www.google.com/favicon.ico"
                  size="18px"
                  class="q-ml-sm"
                />
                <span class="flex-grow text-center text-white"
                  >Continue with Google</span
                >
              </div>
            </template>
          </q-btn>

          <q-btn
            color="dark"
            class="full-width facebook-btn"
            size="lg"
            flat
            @click="facebookSignIn"
          >
            <template v-slot:default>
              <div class="row full-width items-center justify-between">
                <q-icon
                  name="img:https://www.facebook.com/favicon.ico"
                  size="18px"
                  class="q-ml-sm"
                />
                <span class="flex-grow text-center text-white"
                  >Continue with Facebook</span
                >
              </div>
            </template>
          </q-btn>

          <div class="text-center q-mt-lg">
            <span class="text-grey">Already have an account? </span>
            <router-link to="/login" class="text-primary">Sign in</router-link>
          </div>
        </div>
      </q-form>
    </div>
  </q-page>
</template>

<style lang="scss">
.auth-container {
  width: 100%;
  max-width: 500px;

  .text-h5 {
    font-weight: 500;
    padding: 8px 16px;
    border-radius: 4px;
    margin: 0 0 16px 0;
  }
}

.auth-form {
  width: 100%;
  padding: 1.5rem;
  border-radius: 8px;
  background: #1a1a1a;
}

.auth-input {
  .q-field__control {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 4px;

    &:hover {
      background: rgba(255, 255, 255, 0.08);
    }
  }

  .q-field__label {
    color: rgba(255, 255, 255, 0.7);
  }

  input {
    color: white !important;
    &::placeholder {
      color: rgba(255, 255, 255, 0.5);
    }
  }
}

.google-btn,
.facebook-btn {
  border: 1px solid rgba(255, 255, 255, 0.1);
  height: 40px;
  padding: 0 16px;

  &:hover {
    background: rgba(255, 255, 255, 0.05);
  }

  .flex-grow {
    flex-grow: 1;
    text-align: center;
    margin-right: 18px;
  }

  .q-icon {
    opacity: 0.9;
  }
}

.q-btn {
  border-radius: 4px;
  font-weight: 500;
  font-size: 14px;
}

a {
  text-decoration: none;
  font-weight: 500;
}
</style>

<script>
import { defineComponent, ref } from "vue";
import { useRouter } from "vue-router";
import useAuthUser from "src/composables/UserAuthUser";
import useNotify from "src/composables/UseNotify";
import useSupabase from "src/boot/supabase";

export default defineComponent({
  name: "RegisterPage",
  setup() {
    const router = useRouter();
    const { register, loginWithSocialProvider } = useAuthUser();
    const { notifyError, notifySuccess } = useNotify();
    const { supabase } = useSupabase();

    const form = ref({
      name: "",
      email: "",
      password: "",
    });

    const visibility = ref("password");

    const handleGoogleSignUp = async () => {
      try {
        const { error } = await loginWithSocialProvider("google");
        if (error) throw error;

        const session = supabase.auth.session();

        if (session) {
          notifySuccess("Successfully signed up with Google!");
          router.push("/me");
        }
      } catch (error) {
        notifyError(error.message); // Handle any error
      }
    };

    const handlerRegister = async () => {
      try {
        await register(form.value);
        router.replace({
          name: "email-confirmation",
          query: { email: form.value.email, name: form.value.name },
        });
        notifySuccess("Confirmation email sent!");
      } catch (error) {
        notifyError(error.message);
      }
    };

    return {
      form,
      handlerRegister,
      visibility,
      handleGoogleSignUp,
    };
  },
  methods: {
    isValidEmail(val) {
      const emailPattern =
        /^(?=[a-zA-Z0-9@._%+-]{6,254}$)[a-zA-Z0-9._%+-]{1,64}@(?:[a-zA-Z0-9-]{1,63}\.){1,8}[a-zA-Z]{2,63}$/;
      return emailPattern.test(val) || "Invalid email format!";
    },
  },
});
</script>
