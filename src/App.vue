<template>
  <div id="app">
    <!-- The Navbar below stays constant throughout the app.
         We've set up links on the navbar to different "routes",
         like the "Home" page and "About" page.

         There is also a right-aligned link to Login with GitHub.
         When logged in, this shows the username with a dropdown menu
         to see the profile or logout.
    -->
    <div class="content">

    <b-navbar toggleable="md" type="dark" variant="danger" sticky>

      <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>

      <b-navbar-brand to="/"></b-navbar-brand>

      <!-- If the viewport is small, the navbar collapses.
          Everything in b-collapse is what gets collapsed.
      -->
      <b-collapse is-nav id="nav_collapse">

        <!--  Here are links to different routes  -->
        <b-navbar-nav>
          <b-nav-item to="/" exact>Home</b-nav-item>
          <b-nav-item to="/leaderboard">Leaderboard</b-nav-item>
          <b-nav-item to="/play">Play</b-nav-item>
        </b-navbar-nav>

        <!-- Right aligned nav items -->

        <b-navbar-nav class="ml-auto">
          <!-- This part only displays if the user is authenticated -->
          <b-nav-item-dropdown right v-if="userInfo && !userInfo.isAnonymous">
            <template slot="button-content">
              <em>{{userInfo.displayName}}</em>
            </template>
            <b-dropdown-item to="/profile">Profile</b-dropdown-item>
            <b-dropdown-item @click="logout">Signout</b-dropdown-item>
          </b-nav-item-dropdown>

          <!-- The login option shows if the user is not authenticated -->

          <b-nav-item v-else to="login">Login</b-nav-item>

          <b-nav-text v-if="userInfo">
            <b-img v-if="currentLevel.img"
              rounded="circle" width="20"
              height="20"
              alt="img" class="m-1"
              :src="currentLevel.img"
              />
            {{userData.score}}
          </b-nav-text>


        </b-navbar-nav>

      </b-collapse>

      <b-navbar-nav is-nav-bar class="ml-auto pl-3 pr-2" v-show="$route.path.indexOf('/play') == 0">

        <b-nav-form>
          <!--<b-form-input size="sm" class="mr-sm-2" type="text" placeholder="Search"/>-->
          <b-button size="sm" class="my-2 my-sm-0" variant="default" v-b-modal.manualModal>
            Manual

            <!-- TODO: spinner here -->
          </b-button>

        </b-nav-form>

      </b-navbar-nav>

      <b-navbar-nav is-nav-bar class="ml-auto" v-show="$route.path.indexOf('/play') == 0">

        <b-nav-form>
          <!--<b-form-input size="sm" class="mr-sm-2" type="text" placeholder="Search"/>-->
          <b-button size="sm" class="my-2 my-sm-0" v-on:click="next">
            <span v-if="status === 'loading'">
              <i class="fa fa-spinner fa-spin"></i>
            </span>
            <span v-else>
              <span v-if="N">
                Submit {{N}}
              </span>
              <span v-else>
                Next
              </span>
            </span>

            <!-- TODO: spinner here -->
          </b-button>

        </b-nav-form>

      </b-navbar-nav>



    </b-navbar>

    <!-- The content is in the router view -->
    <!-- the modal -->
    <b-modal id="manualModal" title="Manual Input" hide-footer ref="manual">
      <b-form @submit="preventSubmit">
        <b-input v-model="N"></b-input>
      </b-form>
    </b-modal>

    <div class="router">
      <router-view :userInfo="userInfo"
                   :userData="userData"
                   :allUsers="allUsers"
                   :levels="levels"
                   :currentLevel="currentLevel"
                   v-on:taken_tutorial="setTutorial"
                   v-on:updatedN="updateN"
                   v-on:updatedStatus="updateStatus"
                   ref="routeView"
                   />
    </div>
  </div>
    <div class="footer bg-dark">
      <table style="height: 200px; width: 100%;">
        <tbody>
          <tr>
            <td class="align-middle text-center text-white">

            </td>
          </tr>
          <tr>
            <td class="align-middle text-center text-white">
              <router-link to="/about" class="text-white">About</router-link>
              <br>
            </td>
          </tr>
          <tr>
            <td class="align-middle text-center text-white">
            icons on this site were <a href='https://www.freepik.com/free-vector/cute-woodland-animals_1585897.htm'>Designed by Freepik</a>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

  </div>
</template>

<script>
import Vue from 'vue';
import BootstrapVue from 'bootstrap-vue';
import 'bootstrap/dist/css/bootstrap.css';
import 'bootstrap-vue/dist/bootstrap-vue.css';
import _ from 'lodash';
import VueFire from 'vuefire';
import firebase from 'firebase';
import { db } from './firebaseConfig';
import jelly from './assets/jelly.svg';
import giraffe from './assets/giraffe.svg';
import elephant from './assets/elephant.svg';
import narwhal from './assets/narwhal.svg';
import monkey from './assets/monkey.svg';
import jelly_grey from './assets/jelly_gray.svg';
import giraffe_grey from './assets/giraffe_gray.svg';
import elephant_grey from './assets/elephant_gray.svg';
import narwhal_grey from './assets/narwhal_gray.svg';
import monkey_grey from './assets/monkey_gray.svg';
import '../node_modules/font-awesome/css/font-awesome.min.css';

// explicit installation required in module environments
Vue.use(VueFire);
Vue.use(BootstrapVue);

export default {
  name: 'app',
  data() {
    return {
      userInfo: {},
      allUsers: [],
      Nusers: 50,
      N: 0,
      status: 'loading',
      levels: {
        0: {
          level: 0,
          min: 0,
          max: 100,
          character: null,
          img: null,
          img_grey: null,
        },
        1: {
          level: 1,
          min: 101,
          max: 750,
          character: 'jelly',
          img: jelly,
          img_grey: jelly_grey,
        },
        2: {
          level: 2,
          min: 751,
          max: 2000,
          character: 'giraffe',
          img: giraffe,
          img_grey: giraffe_grey,
        },
        3: {
          level: 3,
          min: 2001,
          max: 3500,
          character: 'elephant',
          img: elephant,
          img_grey: elephant_grey,
        },
        4: {
          level: 4,
          min: 3501,
          max: 6000,
          character: 'narwhal',
          img: narwhal,
          img_grey: narwhal_grey,
        },
        5: {
          level: 5,
          min: 6001,
          max: 8000,
          character: 'monkey',
          img: monkey,
          img_grey: monkey_grey,
        },
      },
    };
  },

  mounted() {
    firebase.auth().signInAnonymously().catch(function(error) {
      // Handle Errors here.
      var errorCode = error.code;
      var errorMessage = error.message;
      // ...
    });
  },

  firebase: {
    allUsers: db.ref('/users/').orderByChild('score'),
  },

  computed: {
    userData() {
      let data = {};
      if (!this.userInfo) {
        return data;
      }
      this.allUsers.forEach((val) => {
        if (val['.key'] === this.userInfo.displayName) {
          data = val;
        }
      });
      return data;
    },

    currentLevel() {
      let clev = {};
      _.mapValues(this.levels, (val) => {
        if (this.userData.score >= val.min && this.userData.score <= val.max) {
          clev = val;
        }
      });

      return clev;
    },

  },
  methods: {
    updateN(N) {
      this.N = N;
    },
    updateStatus(status) {
      console.log('updating status');
      this.status = status;
    },
    next() {
      this.$refs.routeView.N = this.N;
      this.$refs.routeView.next();
    },
    preventSubmit(e) {
      e.preventDefault();
      this.$refs.manual.hide();
    },
    logout() {
      firebase.auth().signOut().then(() => {
        this.userInfo = null;
        this.$router.replace('login');
      });
    },
    setUser(user) {
      console.log('setting user', this.userInfo);
      this.userInfo = user;
    },
    setTutorial(val) {
      db.ref(`/users/${this.userInfo.displayName}`).child('taken_tutorial').set(val);
    },
  },

  created() {
    this.userInfo = firebase.auth().currentUser;
    const self = this;
    firebase.auth().onAuthStateChanged((user) => {
      console.log('hi');
      self.userInfo = user;
    });
    console.log('app db is', db);
  },
};
</script>

<style>
  /*
    You can style your component here. Since this is a top level componentm
    the styles follow into child components.
  */

  html, body {
    height: 100%;
  }
  body {
    display: flex;
    flex-direction: column;
  }
  .content {
    flex: 1 0 auto;
    min-height: -webkit-fill-available;
    height: fit-content;
  }

  .footer {
    position: relative;
    height: 200px;
  }

  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    /*color: #2c3e50;*/
  }

  .router {
    padding-top: 40px;
  }

  #signupForm {
    max-width: 400px;
    padding: 20px;
    margin-top: 20px;
    box-shadow: 0px 0 7px 0px #80808036;
  }

  .inline {

  }

  /* The ribbons */

.corner-ribbon{
  width: 200px;
  background: #e43;
  position: absolute;
  top: 25px;
  left: -50px;
  text-align: center;
  line-height: 50px;
  letter-spacing: 1px;
  color: #f0f0f0;
  transform: rotate(-45deg);
  -webkit-transform: rotate(-45deg);
}

/* Custom styles */

.corner-ribbon.sticky{
  position: fixed;
}

.corner-ribbon.shadow{
  box-shadow: 0 0 3px rgba(0,0,0,.3);
}

/* Different positions */

.corner-ribbon.top-left{
  top: 25px;
  left: -50px;
  transform: rotate(-45deg);
  -webkit-transform: rotate(-45deg);
}

.corner-ribbon.top-right{
  top: 25px;
  right: -50px;
  left: auto;
  transform: rotate(45deg);
  -webkit-transform: rotate(45deg);
}

.corner-ribbon.bottom-left{
  top: auto;
  bottom: 25px;
  left: -50px;
  transform: rotate(45deg);
  -webkit-transform: rotate(45deg);
}

.corner-ribbon.bottom-right{
  top: auto;
  right: -50px;
  bottom: 25px;
  left: auto;
  transform: rotate(-45deg);
  -webkit-transform: rotate(-45deg);
}

/* Colors */

.corner-ribbon.white{background: #f0f0f0; color: #555;}
.corner-ribbon.black{background: #333;}
.corner-ribbon.grey{background: #999;}
.corner-ribbon.blue{background: #007aff;
z-index: 99}
.corner-ribbon.green{background: #2c7;}
.corner-ribbon.turquoise{background: #1b9;}
.corner-ribbon.purple{background: #95b;}
.corner-ribbon.red{background: #e43;}
.corner-ribbon.orange{background: #e82;}
.corner-ribbon.yellow{background: #ec0;}

</style>
