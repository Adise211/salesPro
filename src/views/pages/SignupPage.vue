<template>
  <v-container fluid class="fill-height pa-0">
    <v-row class="fill-height">
      <v-spacer></v-spacer>
      <v-col md="3" class="my-auto">
        <div class="text-h5 text-center mb-5 font-weight-bold">
          Start closing
          <span class="text-success">more deals</span>
        </div>
        <div class="subtitle-1 text-center mb-5">It's for free!</div>

        <v-form ref="signupForm" class="d-flex flex-column justify-center align-center">
          <!-- Email -->
          <v-text-field
            v-model="userEmail"
            append-inner-icon="mdi-email"
            placeholder="Email"
            :rules="[signupRules.required]"
            validate-on="blur"
            variant="outlined"
            density="comfortable"
            style="width: 80%"
            color="primary"
          ></v-text-field>
          <!-- Password -->
          <v-text-field
            v-model="userPassword1"
            class="mt-1"
            placeholder="Password"
            :rules="[signupRules.required, signupRules.passwordLength]"
            :type="showPassword1 ? 'text' : 'password'"
            :append-inner-icon="showPassword1 ? 'mdi-eye' : 'mdi-eye-off'"
            validate-on="blur"
            variant="outlined"
            density="comfortable"
            style="width: 80%"
            color="primary"
            @click:appendInner="showPassword1 = !showPassword1"
          ></v-text-field>
          <!-- Verify password -->
          <v-text-field
            v-model="userPassword2"
            class="mt-1"
            placeholder="Verify password"
            :rules="[signupRules.required, signupRules.verifyPassword, signupRules.passwordLength]"
            :type="showPassword2 ? 'text' : 'password'"
            :append-inner-icon="showPassword2 ? 'mdi-eye' : 'mdi-eye-off'"
            validate-on="blur"
            variant="outlined"
            density="comfortable"
            style="width: 80%"
            color="primary"
            @click:appendInner="showPassword2 = !showPassword2"
          ></v-text-field>
          <!-- Fisrt name -->
          <v-text-field
            v-model="userFirstName"
            class="mt-1"
            placeholder="Name"
            :rules="[signupRules.required]"
            validate-on="blur"
            variant="outlined"
            density="comfortable"
            style="width: 80%"
            color="primary"
          ></v-text-field>
          <!-- Last name -->
          <v-text-field
            v-model="userLastName"
            class="mt-1"
            placeholder="Last name (optinal)"
            variant="outlined"
            density="comfortable"
            style="width: 80%"
            color="primary"
          ></v-text-field>
        </v-form>
        <div class="signup-actions d-flex flex-column align-center justify-center mt-5">
          <v-btn color="primary" variant="flat" @click="onSignupClick" :loading="isLoading"
            >Signup</v-btn
          >
          <a
            class="text-decoration-none mt-3 ml-4"
            rel="noopener noreferrer"
            target="_self"
            @click="signinAnchorHandler"
          >
            Have an account already? <span class="text-primary">click here</span>
          </a>
        </div>
      </v-col>
      <v-spacer></v-spacer>
      <v-col md="4" class="pa-0">
        <!-- BG Image -->
        <!-- <div class="logo-img">
          <v-img :src="logoURL" max-height="90vh"></v-img>
        </div> -->
        <div class="side-line d-flex flex-column justify-center">
          <div class="text-h3 text-white text-center font-italic mb-5">SalesPro</div>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { createNewUser } from "../../firebase/services/user";
import { mapActions } from "pinia";
import { useGeneralStore } from "@/stores/general";

export default {
  name: "SignupPage",
  components: {},
  props: {},
  data: () => ({
    showPassword1: false,
    showPassword2: false,
    userEmail: "",
    userPassword1: "",
    userPassword2: "",
    userFirstName: "",
    userLastName: "",
    isLoading: false
  }),
  created() {},
  mounted() {},
  methods: {
    ...mapActions(useGeneralStore, ["setToastMessage"]),
    async onSignupClick() {
      this.isLoading = true;
      const { valid } = await this.$refs.signupForm.validate();
      if (valid) {
        const newUserData = {
          Email: this.userEmail,
          Password: this.userPassword1,
          FirstName: this.userFirstName,
          LastName: this.userLastName
        };
        const response = await createNewUser(newUserData);
        console.log("response in client:", response);
        if (response?.createdNewUser) {
          this.isLoading = false;
          this.setToastMessage({
            type: "success",
            message: "Signed up successfully!"
          });
          setTimeout(() => {
            this.$router.push({
              name: "LoginPage"
            });
          }, 500);
        }
      }
    },
    signinAnchorHandler() {
      this.$router.push({
        name: "LoginPage"
      });
    }
  },
  computed: {
    logoURL() {
      return "/images/login-image2.jpg";
    },
    signupRules() {
      const emailRegex = new RegExp(
        /^[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?$/g
      );
      return {
        required: (value) => !!value || "Required.",
        // TO DO: make the email regex work (not in use right now!)
        emailMatch: (value) => emailRegex.test(value) || "Invalid e-mail.",
        verifyPassword: (value) => value === this.userPassword1 || "Not the same password",
        passwordLength: (value) => value >= 6 || "insert ore than 6 cha..."
      };
    }
  },
  watch: {}
};
</script>

<style scoped>
/* .logo-img {
  position: relative;
  bottom: 14vh;
} */
.side-line {
  border: 1px solid black;
  background-color: rgb(38, 50, 56);
  height: 100%;
  width: 100%;
}
</style>
