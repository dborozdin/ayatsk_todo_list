<template>
	<div id="app">
		<ConnectForm @db-connected="onConnect" v-show="isConnectFormVisible"></ConnectForm>
		<div v-show="!isConnectFormVisible">
      <h1>Перечень важных дел</h1>
			<ToDoForm @add-item="onAdd" @delete-all="onClear"
          :host="host"
          :session_key="session_key"
          :ToDoItems="ToDoItems">>
      </ToDoForm>
			<ul aria-labelledby="list-summary" class="stack-large">
				<li v-for="itemRef in ToDoItems" :key="itemRef.id">
					<ToDoItem
              :itemData="itemRef"
              :host="host"
              :session_key="session_key"></ToDoItem>
				</li>
			</ul>
		</div>
	</div>
</template>


<script>
import ToDoItem from "./components/ToDoItem.vue";
import ConnectForm from "./components/ConnectForm";
import ToDoForm from "./components/ToDoForm";
import axios from 'axios'
import {computed} from 'vue';
import { useStore } from 'vuex'

export default {
	name: "app",
  components: {
		ToDoItem,
		ConnectForm,
		ToDoForm,
	},
  setup(){
      const store = useStore();

      let products = computed(function () {
        return store.state.products
      });

      let cart = computed(function () {
        return store.state.cart
      });

      let ToDoItems = computed(function () {
        return store.state.toDoItems
      });

      return {
        products,
        cart,
        ToDoItems
      }
  },
  data() {
		return {
      session_key: "",
      host: "",
			isConnectFormVisible: true,
      //ToDoItems: [],
			};
  },
	methods: {
    async updateTodoItemsFromDb() {
      try {
        //const res = await axios.get(baseURL);
        const config = {
          headers:{
            "X-APL-SessionKey": this.session_key
          }
        };
        let query= "http://"+this.host+ "/rest/load/t=e&ent=apl_alert&all_attrs=true";
        let self= this;
        axios.get(query, config)
            .then(function (response) {
              // handle success
              console.log("response.data.instances: ",response.data.instances);
              if(response.data.instances)
              {
                for(let i=0; i< response.data.instances.length; i++)
                {
                  let todoData= response.data.instances[i];
                  if(todoData!==undefined)
                  {
                    const instId= todoData.id;
                    const alertText= todoData.attributes.alert_text;
                    let stateDone= false;
                    if(todoData.attributes.state==="выполнено")
                    {
                      stateDone= true;
                    }
                    self.ToDoItems.push({ id:instId, label: alertText, done: stateDone});
                  }
                }
              }
            })
            .catch(function (error) {
              // handle error
              console.log(error);
            });
      } catch (e) {
        console.error(e);
      }

		},
		onConnect(session_key, host) {

      console.log("products:", this.products)

      console.log("App onConnect()!");
      console.log("app got session_key:"+session_key);
      console.log("app got host:"+host);
      this.session_key= session_key;
      this.host= host;
			this.updateTodoItemsFromDb();
			this.isConnectFormVisible=false;
		},
    onAdd(newItem) {
      console.log("App onAdd()!");
      this.ToDoItems.push(newItem);

    },
    onClear() {
      console.log("App onClear()!");
      this.ToDoItems= [];
    },
	},
};
</script>

<style>
  /* Global styles */
  .btn {
    padding: 0.8rem 1rem 0.7rem;
    border: 0.2rem solid #4d4d4d;
    cursor: pointer;
    text-transform: capitalize;
  }
  .btn__danger {
    color: #fff;
    background-color: #ca3c3c;
    border-color: #bd2130;
  }
  .btn__filter {
    border-color: lightgrey;
  }
  .btn__danger:focus {
    outline-color: #c82333;
  }
  .btn__primary {
    color: #fff;
    background-color: #000;
  }
  .btn-group {
    display: flex;
    justify-content: space-between;
  }
  .btn-group > * {
    flex: 1 1 auto;
  }
  .btn-group > * + * {
    margin-left: 0.8rem;
  }
  .label-wrapper {
    margin: 0;
    flex: 0 0 100%;
    text-align: center;
  }
  [class*="__lg"] {
    display: inline-block;
    width: 100%;
    font-size: 1.9rem;
  }
  [class*="__lg"]:not(:last-child) {
    margin-bottom: 1rem;
  }
  @media screen and (min-width: 620px) {
    [class*="__lg"] {
      font-size: 2.4rem;
    }
  }
  .visually-hidden {
    position: absolute;
    height: 1px;
    width: 1px;
    overflow: hidden;
    clip: rect(1px 1px 1px 1px);
    clip: rect(1px, 1px, 1px, 1px);
    white-space: nowrap;
  }

  .visually-hidden {
    clip-path: inset(1px, 1px, 1px, 1px);
  }

  [class*="stack"] > * {
    margin-top: 0;
    margin-bottom: 0;
  }
  .stack-small > * + * {
    margin-top: 1.25rem;
  }
  .stack-large > * + * {
    margin-top: 2.5rem;
  }
  @media screen and (min-width: 550px) {
    .stack-small > * + * {
      margin-top: 1.4rem;
    }
    .stack-large > * + * {
      margin-top: 2.8rem;
    }
  }
  /* End global styles */
  #app {
    background: #fff;
    margin: 2rem 0 4rem 0;
    padding: 1rem;
    padding-top: 0;
    position: relative;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2), 0 2.5rem 5rem 0 rgba(0, 0, 0, 0.1);
  }
  @media screen and (min-width: 550px) {
    #app {
      padding: 4rem;
    }
  }
  #app > * {
    max-width: 50rem;
    margin-left: auto;
    margin-right: auto;
  }
  #app > form {
    max-width: 100%;
  }
  #app h1 {
    display: block;
    min-width: 100%;
    width: 100%;
    text-align: center;
    margin: 0;
    margin-bottom: 1rem;
  }
</style>
