<template lang="html">
  <farm-main :paddingTop="['xl', 'xxl']" :paddingX="['l', 'xxl']">

    <farm-stack space="l">
      <farm-card backgroundColor="red">
        <h3 class="card-title">{{ $t('WARNING!')}}</h3>
        <p class="card-text">
          {{ $t('Logging out will permanently delete all logs stored on this device.')}}
          {{ $t('You will not be able to recover logs which have not been synced with')}}
          {{ $t('your farmOS server. Go back to sync any unsaved logs, or proceed to')}}
          {{ $t('logout and clear all data from this device.')}}
        </p>
      </farm-card>
      <farm-inline justifyContent="center" space="m">
        <button
          @click="goBack"
          class="btn btn-lg btn-success"
          type="button"
          name="button">
          {{ $t('Go Back')}}
        </button>
        <button
          @click="logout"
          class="btn btn-lg btn-primary"
          type="button"
          name="button">
          {{ $t('Logout')}}
        </button>
      </farm-inline>
      <p style="textAlign: center">
        Need to login again? <router-link to="/login">Login here</router-link>
      </p>
    </farm-stack>

  </farm-main>
</template>

<script>
export default {
  name: 'Logout',
  methods: {
    goBack() {
      this.$router.back();
    },
    logout() {
      // Call logout function from farmos.js
      this.$store.dispatch('logout');

      // Remove logs, assets, areas, user info & site info from store & local persistance
      this.$store.commit('deleteAllLogs');
      this.$store.dispatch('deleteAllCachedLogs');
      this.$store.commit('deleteAllAssets');
      this.$store.dispatch('deleteAllCachedAssets');
      this.$store.commit('deleteAllAreas');
      this.$store.dispatch('deleteAllCachedAreas');
      this.$store.commit('deleteAllUnits');
      this.$store.dispatch('deleteAllCachedUnits');
      this.$store.commit('deleteAllCategories');
      this.$store.dispatch('deleteAllCachedCategories');
      this.$store.dispatch('deleteCachedUserAndSiteInfo');

      // Set login status to false and return to login screen
      this.$store.commit('setLoginStatus', false);
      this.$router.push({ path: '/login' });
    },
  },
};
</script>

<style lang="css" scoped>
  a {
    color: inherit;
    text-decoration: underline;
  }
</style>
