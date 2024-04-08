<template>
  <form @submit.prevent="onSubmit">
    <h2 class="label-wrapper">
      <label for="new-todo-input" class="label__lg">
        Что требуется сделать?
      </label>
    </h2>
    <input
      type="text"
      id="new-todo-input"
      name="new-todo"
      autocomplete="off"
      v-model.lazy.trim="label"
      class="input__lg" />
    <button type="submit" class="btn btn__primary btn__lg" @click="onAdd">Добавить</button>
    <button type="submit" class="btn btn__danger btn__lg" @click="onDeleteAll">Очистить список</button>
  </form>
</template>

<script>
  import axios from 'axios'
	export default {
    props: {
      host:String,
      session_key:String,
      ToDoItems: Array,
    },
		methods: {
			async onAdd(e) {
				if (this.label === "") {
					return;
				}
        try {
          const config = {
            headers:{
              "X-APL-SessionKey": this.session_key
            }
          };
          let query= "http://"+this.host+ "/rest/save";
          let data= {
            "format":"apl_json_1",
            "dictionary": "apl_pss_a",
            "instances" : [
              {
                "index": 1,
                "type": "apl_alert",
                "attributes":
                    {
                      "alert_text": this.label
                    }
              }
            ]
          };
          let self= this;
          axios.post(query, data, config)
              .then(function (response) {
                // handle success
                console.log("response.data.instances: ", response.data.instances);
                for(let i=0; i!= response.data.instances.length; i++)
                {
                  let todoData= response.data.instances[i];
                  console.log("todoData:");
                  console.log(todoData);
                  if(todoData!=undefined)
                  {
                    const instId= todoData.id;
                    self.$emit('add-item', { id:instId, label: self.label, done: false});
                    self.label = "";
                  }
                }
              })
              .catch(function (error) {
                // handle error
                console.log(error);
              });
        } catch (e) {
          console.error(e);
          self.label = "";
        }
        e.preventDefault();
			},
      async onDeleteAll(e) {
        if(!confirm('Очистить список?'))
        {
          return;
        }
        console.log('onDeleteAll!');
        let self= this;
        try {
          const config = {
            headers:{
              "X-APL-SessionKey": self.session_key
            }
          };
          let baseQueryURL= "http://"+self.host+ "/rest/";
          let idS="";
          self.ToDoItems.forEach((value) => {
            if(idS!=="")
            {
              idS+=",";
            }
            idS+=String(value.id);
          });
          if(idS==="")
          {
            return;
          }
          baseQueryURL+=idS;
          console.log("delete query:"+baseQueryURL);
          axios.delete(baseQueryURL, config)
              .then(function () {
                // handle success
                console.log("axios.delete success! ");
                self.$emit('delete-all');
              })
              .catch(function (error) {
                // handle error
                console.log(error);
              });
        } catch (e) {
          console.error(e);
        }
        e.preventDefault();
      },
		},
		data() {
			return {
				label: "",
			};
		},
	};
</script>