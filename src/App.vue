<template>
  <div id="app">
    <div>
      <b-navbar type="dark" variant="primary" toggleable>
        <b-navbar-toggle target="nav_dropdown_collapse"></b-navbar-toggle>
        <b-collapse is-nav id="nav_dropdown_collapse">
          <b-navbar-nav>
            <b-nav-item to="/">Home</b-nav-item>
            <!-- <b-nav-item to="/territories">Territories</b-nav-item> -->
            <b-nav-item-dropdown class="group-codes" text="Territories">
              <b-dropdown-item v-for="group in groupCodes" v-bind:key="group" :to="`/territories/${group}`">
                <font-awesome-icon icon="check" v-if="group === groupCode" /> {{group}}
              </b-dropdown-item>
            </b-nav-item-dropdown>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </div>
    <router-view></router-view>
  </div>
</template>

<script>
import axios from 'axios';
import { uniqBy } from 'lodash';
import Home from './components/Home'
import Territories from './components/Territories';

export default {
  name: 'app',
  components: {
    Home,
    Territories,
  },
  watch: {
    '$route' (to) {
      this.groupCode = to.params.group;
    }
  },
  data() {
    return {
      congId: 1,
      groupCode: this.$route.params.group,
      groupCodes: [],
    };
  },
  methods: {
    async getGroupCodes() {
      const response = await axios({
        url: process.env.VUE_APP_ROOT_API,
        method: 'post',
        headers: {
          'Content-Type': 'application/json'
        },
        data: {
          query: `{ territories (congId: ${this.congId}) { group_code }}`
        }
      });

      const territories = response.data.data.territories;
      // const group = sessionStorage.getItem('group-code');
      // if (group) this.setGroupCode(group);

      return uniqBy(territories, 'group_code').map(g => g.group_code).sort();
    },

    async setGroupCode(value) {
      if(value != this.groupCode) {
        this.groupCode = value;
      }
      // this.territories = await this.getTerritories();
      // sessionStorage.setItem('group-code', value);
    },

  },

  async mounted() {
    this.groupCodes = await this.getGroupCodes();
  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #696969;
  margin-top: 60px;
  margin-right: 2px;
}
.dropdown-item {
  color: #696969;
}
.dropdown-item.active, .dropdown-item:active {
  padding-left: 0;
}
</style>
