<template>
  <form @submit.prevent="">
    <h1>Соединение с БД</h1>
    <h2 class="label-wrapper">
      <label for="connect-input-host" class="label__lg">
        Адрес сервера
      </label>
    </h2>
    <input
        type="text"
        id="connect-input-host"
        name="connect-db-host"
        autocomplete="off"
        v-model.lazy.trim="host"
        class="input__lg" />
    <select id="connect-dbname"
            v-model="selected_db_name"
            class="input__lg">
      <option v-for="dbname in dbnames" :value="dbname.value" v-bind:key="dbname">
        {{ dbname.value }}
      </option>
    </select>
    <button @click="updateDbList" class="btn__primary btn__lg">Обновить список БД</button>
    <h2 class="label-wrapper">
      <label for="connect-input-username" class="label__lg">
        Пользователь
      </label>
    </h2>
    <input
      type="text"
      id="connect-input-username"
      name="connect-db-username"
      autocomplete="off"
      v-model.lazy.trim="username"
      class="input__lg" />
    <h2 class="label-wrapper">
      <label for="connect-input-password" class="label__lg">
        Пароль
      </label>
    </h2>
    <input
        type="text"
        id="connect-input-password"
        name="connect-db-password"
        autocomplete="off"
        v-model.lazy.trim="password"
        class="input__lg" />
    <button type="submit" class="btn btn__primary btn__lg" @click="submitForm">Соединение</button>
  </form>
</template>

<script>
  import { ref } from 'vue'
  import axios from 'axios'
	export default {
    async mounted() {
      console.log('connect form created');
      this.updateDbList();
    },
    methods: {
      updateDbList(e) {
        console.log("updateDbList called");
        this.dbnames=[];
        let query= "http://" + this.host + "/rest/dblist";
        let self= this;
        axios.get(query)
            .then(function (response) {
              // handle success
              console.log("dblist response.data: ",response.data);
              for(const dbname of response.data.dblist)
              {
                self.dblistCnt= self.dbnames.push({value: dbname});
              }
              console.log("self.dbnames:");
              console.log(self.dbnames);
              console.log("self.dbnames cnt:",self.dblistCnt);
              if(self.dblistCnt>0)
              {
                self.selected_db_name=self.dbnames[0].value;
              }
            })
            .catch(function (error) {
              // handle error
              console.log(error);
            });
        if(e) {
          e.preventDefault();
        }
      },
      async doConnect() {
        let session_key="";
        let self= this;
        let query= "http://"+this.host+ "/rest/connect/user="+this.username;
        if(this.password!="" && this.password!=undefined)
        {
          query+="&pass="+this.password;
        }
        query+="&db="+this.selected_db_name;
        console.log("connect query:"+query);
        axios.get(query)
            .then(function (response) {
              // handle success
              console.log("response.data.session_key: "+response.data.session_key);
              session_key=response.data.session_key;
              self.$emit('db-connected', session_key, self.host)
            })
            .catch(function (error) {
              // handle error
              console.log(error);
            });
      },
      async submitForm() {
				if (this.username === "") {
					return;
				}
        let queryDisconnect="http://"+this.host+"/rest/disconnect";
        console.log("queryDisconnect:"+queryDisconnect);
        let self= this;
        axios.get(queryDisconnect)
            .then(function () {
              // handle success
              console.log("disconnected from "+this.host);
              self.doConnect();

            })
            .catch(function (errorDisconnect) {
              console.log(errorDisconnect);
              self.doConnect();
            });


			},
		},
		data() {
			return {
        host: "localhost:7239",
        username: "Administrator",
        password: "",
        selected_db_name: ref('< Соединение с БД не выбрано >'),
        dbnames: ref([
          { value: '< Соединение с БД не выбрано >' }
        ]),
			};
		},
	};
</script>