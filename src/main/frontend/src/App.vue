<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername" :meetings="meetings"></MeetingsPage>
    </div>

    <div v-else>
      <button :class="signingUp ? 'button-outline' : '' "@click="signingUp = false"> Logowanie </button>
      <button :class="!signingUp ? 'button-outline' : '' "@click="signingUp = true"> Rejestracja </button>

      <div v-if="message" class="['alert', 'alert-' + (this.isError ? 'error' : 'success')]">
      </div>
      <div :class="['alert', 'alert-' + (this.isError ? 'error' : 'success')]" v-if="message">
        {{ message }}

      </div>

      <LoginForm v-if="!signingUp" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="Załóż konto"></LoginForm>
    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from "axios"

export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      message: "",
      signingUp: false,
      authenticatedUsername: '',
      meetings: [],
      isError: false
    }
  },

  methods: {
    logMeIn(user) {
      axios.post("/api/tokens", user)
          .then(response => {
            this.authenticatedUsername = user.login;
            const token = response.data.token;
            axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;
            axios.get('/api/meetings')
                .then(response => {
                  this.meetings = response.data;
                })
                .catch(response => {
                  this.isError = true;
                  this.message = ("Nie udało się pobrać listy spotkań")
                })
            this.isError = false;
            this.message = ("Udało się zalogować")
          })
          .catch(response => {
            this.isError = true;
            this.message = ("Nie udało się zalogować")
          })
    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
    },

    register(user) {
      axios.post('/api/participants', user)
          .then(response => {
            this.isError = false;
            this.message = ("Udało się założyć konto")

          })
          .catch(response => {
            this.isError = true;
            this.message = ("Nie udało się założyć konta")
          });
    },
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}

.alert {
  border: 1px solid black;
  font-size: 20px;
  padding: 5px;
}

.alert-success {
  background: darkseagreen;
  border-color: green;
}
.alert-error {
  background: palevioletred;
  border-color: darkred;
}
</style>
