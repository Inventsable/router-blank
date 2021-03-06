<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link>|
      <router-link to="/about">About</router-link>
    </div>
    <router-view />
    <identity />
    <menus />
    <version />
  </div>
</template>

<script>
// Dynamic CSS variables that automatically handle all app themes and changes:
// https://github.com/Inventsable/starlette
import starlette from "starlette";

// Dynamic identification object that reports all panel and host information:
// https://github.com/Inventsable/CEP-Spy
import spy from "cep-spy";

// Utility components
// https://github.com/Inventsable/cep-vue-cli-router2x#components
import menus from "./components/main/menus.vue";
import version from "./components/main/version.vue";

export default {
  name: "App",
  components: {
    menus,
    version
  },
  computed: {
    storage() {
      return window.localStorage;
    }
  },
  data: () => ({
    csInterface: null,
    identity: null,
    menus: null,
    isMounted: false
  }),
  mounted() {
    console.clear();
    this.csInterface = new CSInterface();
    this.csInterface.addEventListener("console", this.consoler);

    // Utility components are already mounted prior to this
    console.log(
      `${spy.extName} ${spy.extVersion} : ${spy.isDev ? "DEV" : "BUILD"}`
    );
    this.isMounted = true;

    this.loadUniversalScripts();

    // Dynamic color handling library, see
    // https://github.com/Inventsable/starlette
    // for details and list of available CSS variables.
    starlette.init();
  },
  methods: {
    dispatchEvent(name, data) {
      var event = new CSEvent(name, "APPLICATION");
      event.data = data;
      this.csInterface.dispatchEvent(event);
    },
    loadScript(path) {
      // Correctly loads a script regardless of whether Animate or regular CEP app
      if (!/FLPR/.test(spy.appName))
        this.csInterface.evalScript(`$.evalFile('${path}')`);
      else
        this.csInterface.evalScript(
          `fl.runScript(FLfile.platformPathToURI("${path}"))`
        );
    },
    loadUniversalScripts() {
      // Preloads any script located inside ./src/host/universal
      let utilFolder = window.cep.fs.readdir(
        `${spy.path.root}/src/host/universal/`
      );
      if (!utilFolder.err) {
        let valids = utilFolder.data.filter(file => {
          return /\.(jsx|jsfl)$/.test(file);
        });
        valids.forEach(file => {
          this.loadScript(`${spy.path.root}/src/host/universal/${file}`);
        });
      }
      // Preloads any script located in ./src/host/[appName]/
      let hostFolder = window.cep.fs.readdir(
        `${spy.path.root}/src/host/${spy.appName}/`
      );
      if (!hostFolder.err) {
        let valids = hostFolder.data.filter(file => {
          return /\.(jsx|jsfl)$/.test(file);
        });
        valids.forEach(file => {
          this.loadScript(`${spy.path.root}/src/host/${spy.appName}/${file}`);
        });
      } else {
        console.log(
          `${spy.path.root}/src/host/${spy.appName} has no valid files or does not exist`
        );
      }
    },
    consoler(msg) {
      // Catches all console.log() usage in .jsx files via CSEvent
      console.log(`${spy.appName}: ${msg.data}`);
    },
    getCSS(prop) {
      // Returns current value of CSS variable
      // prop == typeof String as name of variable, with or without leading dashes:
      // this.getCSS('color-bg') || this.getCSS('--scrollbar-width')
      return window
        .getComputedStyle(document.documentElement)
        .getPropertyValue(`${/^\-\-/.test(prop) ? prop : "--" + prop}`);
    },
    setCSS(prop, data) {
      // Sets value of CSS variable
      // prop == typeof String as name of variable, with or without leading dashes:
      // this.setCSS('color-bg', 'rgba(25,25,25,1)') || this.setCSS('--scrollbar-width', '20px')
      document.documentElement.style.setProperty(
        `${/^\-\-/.test(prop) ? prop : "--" + prop}`,
        data
      );
    }
  }
};
</script>

<style>
#nav {
  padding: 10px;
  display: flex;
  justify-content: center;
  color: var(--color-default);
}

#nav a {
  padding: 0px 5px;
  font-weight: bold;
  color: var(--color-default);
}

#nav a.router-link-exact-active {
  color: var(--color-selection);
}

/* Various helper styles to match host application's theme */
@import url("https://fonts.googleapis.com/css?family=Open+Sans&display=swap");
:root {
  --quad: cubic-bezier(0.48, 0.04, 0.52, 0.96);
  --quart: cubic-bezier(0.76, 0, 0.24, 1);
  --quint: cubic-bezier(0.84, 0, 0.16, 1);

  background-color: var(--color-bg);
  color: var(--color-default);
  font-family: "Open Sans", sans-serif;
  font-size: 10px;
}

body::-webkit-scrollbar {
  width: 0px;
}
#app::-webkit-scrollbar {
  display: block;
}
::-webkit-scrollbar {
  background-color: var(--color-scrollbar);
  width: var(--scrollbar-width);
}
::-webkit-scrollbar-thumb {
  width: var(--scrollbar-width);
  background: var(--color-scrollbar-thumb);
  border-radius: var(--scrollbar-thumb-radius);
}
::-webkit-scrollbar-thumb:hover {
  background: var(--color-scrollbar-thumb-hover);
}
::-webkit-scrollbar-resizer {
  display: none;
}
::-webkit-scrollbar-button {
  height: 0px;
}
</style>
