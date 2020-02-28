<template>
  <div class="versionFoot">{{ version }}</div>
</template>

<script>
import spy from "cep-spy";

export default {
  name: "version",
  data: () => ({
    isMounted: true
  }),
  mounted() {
    this.isMounted = true;
  },
  computed: {
    app() {
      return this.$root.$children[0];
    },
    isDev() {
      return spy.isDev;
    },
    version() {
      return spy.extVersion;
    }
  }
};
</script>

<style>
.versionFoot {
  cursor: default;
  position: absolute;
  bottom: 0px;
  right: 0px;
  padding: 0px 1rem 6px;
  color: var(--color-default);
  letter-spacing: 0.5ch;
  opacity: 0.5;
}
</style>
