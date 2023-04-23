<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>

    <div v-else>
        <button :class="signingUp ? 'button-outline' : ''"  @click = "signingUp = false">Logowanie</button>
        <button :class="!signingUp ? 'button-outline' : ''" @click = "signingUp = true">Rejestracja</button>

      <div v-if="" class ="alert">
        {{ message }}
      </div>
      <LoginForm v-if="!signingUp" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="Zaloz konto"></LoginForm>
    </div>
  </div>
</template>



<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from 'axios';
export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      signingUp: false,
      authenticatedUsername: '',
      message: '',
      isPositive: false
    }
  },
  methods: {
    logMeIn(user) {
      axios.post('/api/tokens', user)
          .then(response => {
            // jestes zalogowany
            this.authenticatedUsername = user.login;
            const token = response.data.token;
            axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;
            axios.get('/api/meetings')
                .then(response => console.log(response.data));
          })
          .catch(response => {
            // niepoprawne haslo
            this.message = 'Nieudane logowanie'
          });
     // this.authenticatedUsername = user.login;
    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
    },

    register(user) {
      axios.post('/api/participants', user)
          .then(response => {
            this.message = 'Twoje konto zostalo zalozone'
          })
          .catch(response => {
            // nie udało sie
            this.message = 'Taki uzytkownik juz istnieje'
          });
    }
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}

.alert {
  padding: 5px;
  margin: 30px;
  background: lightgreen;
  font-size: 2em;
  border: 3px darkgreen;
}
</style>
