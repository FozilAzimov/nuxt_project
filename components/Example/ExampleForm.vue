<template>
  <div>
    <div class="w-full max-w-xs">
      <ValidationObserver ref="formValidation">
        <form @submit.prevent="submit" class="bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4">
          <div class="mb-4">
            <ValidationProvider rules="required" v-slot="{ errors }">
              <label class="block text-gray-700 text-sm font-bold mb-2" for="username">
                Username
              </label>
              <input v-model="form.username" :class="{'border-red-500': errors[0]}" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" id="username" type="text" placeholder="Username">
            </ValidationProvider>
          </div>
          <div class="mb-6">
            <ValidationProvider rules="required" v-slot="{ errors }">
              <label class="block text-gray-700 text-sm font-bold mb-2" for="password">
                Password
              </label>
              <input v-model="form.password" :class="{'border-red-500': errors[0]}" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline" id="password" type="password" placeholder="******************">
              <p v-if="errors[0]" class="text-red-500 text-xs italic">Please choose a password.</p>
            </ValidationProvider>
          </div>
          <div class="flex items-center justify-between">
            <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" type="submit">
              Кириш 
            </button>
            <a class="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" href="#">
              Forgot Password?
            </a>
          </div>
        </form>
      </ValidationObserver>
    </div>
    <button @click="postData">Abbos button</button>
    <button @click="drillingData">Abbos drilling button</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      form: {
        username: null,
        password: null,
      },
      tokenUrl: null
    }
  },
  methods: {
    submit () {
      this.$refs.formValidation.validate().then((success) => {
        if (success) {
          axios.post('http://192.168.1.58:8443/security/logIn', {
            username: this.form.username,
            password: this.form.password
          })
          .then(response => {
            const token = response.data.token;
            localStorage.setItem('token', token);
            console.log(response, token)
          })
          .catch(error => {
            console.error('Login failed:', error);
          });
        }
      })
    },
    postData(){
      this.tokenUrl = localStorage.getItem('token');
      console.log(this.tokenUrl)
      axios.get('http://192.168.1.58:8080/rightMap', {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
      })
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
    },
    drillingData(){
      this.tokenUrl = localStorage.getItem('token');
      console.log(this.tokenUrl)
      axios.get('http://192.168.1.58:8080/drillingList', {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
      })
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
    }
  }
}
// http://192.168.1.58:8080/rightMap
</script>