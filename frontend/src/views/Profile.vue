<template>
  <div class="profile">
    <h1>User Profile</h1>

    <div v-if="!isAuthenticated">
      <button @click="login">Log in</button>
    </div>

    <div v-else>
      <p><strong>Name:</strong> {{ user.name }}</p>
      <p><strong>Email:</strong> {{ user.email }}</p>
      <img :src="user.picture" alt="profile picture" v-if="user.picture" />

      <button @click="logout">Log out</button>
    </div>
  </div>
</template>

<script>
import createAuth0Client from "@auth0/auth0-spa-js";
import authConfig from "../../auth_config.json";

export default {
  name: "Profile",
  data() {
    return {
      auth0: null,
      isAuthenticated: false,
      user: {}
    };
  },
  async created() {
    this.auth0 = await createAuth0Client({
      domain: authConfig.domain,
      client_id: authConfig.clientId,
      redirect_uri: window.location.origin
    });

    // Handle redirect back from Auth0
    if (window.location.search.includes("code=") && window.location.search.includes("state=")) {
      await this.auth0.handleRedirectCallback();
      window.history.replaceState({}, document.title, "/");
    }

    this.isAuthenticated = await this.auth0.isAuthenticated();

    if (this.isAuthenticated) {
      this.user = await this.auth0.getUser();
    }
  },
  methods: {
    async login() {
      await this.auth0.loginWithRedirect();
    },
    async logout() {
      this.auth0.logout({
        returnTo: window.location.origin
      });
    }
  }
};
</script>

<style scoped>
.profile {
  padding: 20px;
}
button {
  margin-top: 10px;
  padding: 8px 12px;
  cursor: pointer;
}
</style>
