<template>
  <div class="flex justify-center items-center h-screen w-screen" v-if="landing">
    <div class="flex flex-row">
      <div class="text-2xl bg-orange-400 text-white p-4 rounded-md mx-5 cursor-pointer" @click="toggleEntry">Entry</div>
      <div class="text-2xl bg-orange-400 text-white p-4 rounded-md mx-5 cursor-pointer" @click="toggleDisplay">View</div>
    </div>
  </div>
  <div class="flex justify-center items-center h-screen w-screen" v-if="entry">
    <form class="min-w-[1000px] max-h-[800px] border-2 border-dotted border-gray-400 px-[100px] py-[100px] rounded-2xl" method="post" @submit.prevent="submit" @reset="onReset">
      <div class="flex flex-col">
        <label for="email" class="px-6 py-2 uppercase text-base">Email:</label>
        <input type="text" name="email" id="email" v-model="subscriber.email" class=" border-gray-600 border mx-6 p-2 focus:outline-none focus:border-red-600 focus:border-[2px]">
        <p class="px-6 text-red-600 text-base" v-for="error of v$.subscriber.email.$errors" :key="error.$uid">
          <strong>{{ error.$property }} {{ error.$validator }}</strong>
        </p>
        <label for="email" class="px-6 py-2 uppercase text-base">Name:</label>
        <input type="text" name="email" id="email" v-model="subscriber.name" class=" border-gray-600 border mx-6 p-2 focus:outline-none focus:border-red-600 focus:border-[2px]">
        <p class="px-6 text-red-600 text-base" v-for="error of v$.subscriber.name.$errors">
          <strong>{{ error.$property }} {{ error.$validator }}</strong>
        </p>
        <label for="email" class="px-6 py-2 uppercase text-base">Last Name:</label>
        <input type="text" name="email" id="email" v-model="subscriber.last_name" class=" border-gray-600 border mx-6 p-2 focus:outline-none focus:border-red-600 focus:border-[2px]">
        <p class="px-6 text-red-600 text-base" v-for="error of v$.subscriber.last_name.$errors">
          <strong>{{ error.$property }} {{ error.$validator }}</strong>
        </p>
        <label for="email" class="px-6 py-2 uppercase text-base">Status:</label>
        <select name="status" id="status" v-model="subscriber.status" class="border-gray-600 border mx-6 p-2 focus:outline-none focus:border-red-600 focus:border-[2px]">
            <option value="1">Enable</option>
            <option value="0">Disable</option>
        </select>
        <p class="px-6 text-red-600 text-base" v-for="error of v$.subscriber.status.$errors">
          <strong>{{ error.$property }} {{ error.$validator }}</strong>
        </p>
      </div>
      <div class="flex justify-center items-center py-[20px]">
       <div class="flex flex-col text-center">
          <button type="submit" class="rounded-lg bg-orange-500 text-white p-4 hover:bg-orange-900">Create Subscriber</button>
          <div class="underline text-base uppercase text-blue-600 mt-6 cursor-pointer" @click="toggleLanding">Back</div>
       </div>
      </div>
    </form>
  </div>
  <div class="mx-[150px] my-[100px]" v-if="display">
   <div class="flex justify-center items-center">
    <div class="flex flex-col">
      <p class="uppercase text-2xl font-bold">List of all Subscribers</p>
      <div class="text-center uppercase underline text-blue-600 font-bold cursor-pointer" @click="toggleLanding">back</div>
    </div>
   </div>
   <div class="flex flex-col">
    <table class="min-w-full text-left text-sm font-light">
      <thead class="border-b font-medium">
        <th scope="col" class="px-6 py-4">Id</th>
        <th scope="col" class="px-6 py-4">Email</th>
        <th scope="col" class="px-6 py-4">Name</th>
        <th scope="col" class="px-6 py-4">Last Name</th>
        <th scope="col" class="px-6 py-4">Status</th>
      </thead>
      <tbody>
        <tr v-for="(p, index) in paginated" :key="index" class="border-b">
          <td class="whitespace-nowrap px-6 py-4">{{ p.id }}</td>
          <td class="whitespace-nowrap px-6 py-4">{{ p.email }}</td>
          <td class="whitespace-nowrap px-6 py-4">{{ p.name }}</td>
          <td class="whitespace-nowrap px-6 py-4">{{ p.last_name }}</td>
          <td class="whitespace-nowrap px-6 py-4">{{ p.status }}</td>
        </tr>
      </tbody>
   </table>
   <div class="flex flex-row">
      <div class="rounded bg-orange-400 p-[5px] my-4 mx-4 text-white cursor-pointer" @click="prev">Prev</div>
      <div class="flex justify-center items-center">
        <span class="text-1xl mx-4">{{ current }} of {{ total }}</span>
      </div>
      <div class="rounded bg-orange-400 p-[5px] my-4 mx-4 text-white cursor-pointer" @click="next">Next</div>
   </div>
   </div>
  </div>
</template>

<script>
  import { useVuelidate } from '@vuelidate/core'
  import { required, email, minLength, maxLength, integer } from '@vuelidate/validators'
  export default{
    data() {
      return {
        v$ : useVuelidate(),
        landing: true,
        entry : false,
        display : false,
        subscriber: {
          email: null,
          name: null,
          last_name: null,
          status: null,
        },
        totalCount : 0,
        current : 1,
        pageSize: 10,
        subscribers: [],
      }
    },
    validations(){
      return {
        subscriber: {
          email: {required, email},
          name: {required},
          last_name: {required},
          status: {
            required,
            minLength: 1,
            maxLength: 2,
            integer,
          }
        }
      }
    },  
    mounted(){
      this.getdata();
    },
    computed: {
      indexStart() {
        return (this.current - 1) * this.pageSize;
      },
      indexEnd(){
        return this.indexStart + this.pageSize;
      },
      paginated(){
        return this.subscribers.slice(this.indexStart, this.indexEnd);
      },
      total(){
        let x = this.subscribers.length / 10;
        if(this.subscribers.length % 10 === 0){

        } else {
          x++;
        }

        return Math.trunc(x);
      },
    },
   
    methods: {
      toggleEntry(){
        this.landing = false;
        this.entry = true;
        this.display = false;
      },
      toggleLanding(){
        this.landing = true;
        this.entry = false;
        this.display = false;
      },
      toggleDisplay(){
        this.landing = false;
        this.entry = false;
        this.display = true;
      },
      prev(){
        if(this.current > 1){
          this.current--; 
        }
      }, 
      next(){
        let x = this.subscribers.length / 10;

        if(this.subscribers.length % 10 === 0){
          
        } else {
          x++;
        }

        if(this.current < Math.trunc(x)){
          this.current++;
        }
      },
      async submit() {
        const result = await this.v$.$validate();
        if(!result){
          alert("Invalid Form Data");

          return;
        } 

        const requestOptions = {
          method: "POST",
          headers: {"Content-Type": "application/json charset='UTF-8'"},
          body: JSON.stringify({
            email: this.subscriber.email,
            name: this.subscriber.name,
            last_name: this.subscriber.last_name,
            status: parseInt(this.subscriber.status),
          }),
        };

        fetch("http://localhost:8000/subscriber", requestOptions)
          .then(async response => {
            const data = await response.json();

            if(!response.ok){
              const error = (data && data.message) || response.status;
              return Promise.reject(error);
            }

           
            if(data.message){
              alert(data.message);
            } else if(data.data){
              alert("Subscriber Added.");

              this.subscriber.email = null;
              this.subscriber.name = null;
              this.subscriber.last_name = null;
              this.subscriber.status = null;

              this.v$.$reset();

              this.getdata();
            }
          }).catch(error => {
            alert(error);
          });
      },
      onReset(){
        this.v$.$reset();
      },
      async getdata(){
        const requestOptions = {
          method: "GET",
          headers: {"Content-Type": "application/json charset='UTF-8'"},
        };

        fetch("http://localhost:8000/subscriber", requestOptions)
          .then(async response => {
            const data = await response.json();

            if(!response.ok){
              const error = (data && data.message) || response.status;
              return Promise.reject(error);
            }

            this.subscribers = data.data;

          }).catch(error => {
            console.log(error);
          });
      }
    }
  }
</script>

<style>

</style>