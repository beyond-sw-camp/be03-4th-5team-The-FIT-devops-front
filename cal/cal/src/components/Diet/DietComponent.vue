<template>
  <title>Diet</title>
  <div class="w-[1440px] h-[1024px] relative overflow-hidden animated-background">
      <BackgroundComponent></BackgroundComponent>
      <h2 class="dietTitle">
          오늘의 식단
      </h2>
      <div class="content-wrapper">
          <div class="record">
              <table class="ml-20 justify-center text-l text-left rtl:text-right text-gray-500 ">
                  <thead class="text-l text-gray-700 uppercase dark:bg-gray-700 ">
                      <tr>
                          <th scope="col" class="px-6 py-3">
                              분류
                          </th>
                          <th scope="col" class="px-6 py-3">
                              사진
                          </th>
                          <th scope="col" class="px-6 py-3">
                              상세설명
                          </th>
                          <th scope="col" class="px-6 py-3">
                              업로드 시간
                          </th>

                      </tr>
                  </thead>
                  <tbody>
                      <tr v-for="diet in addDietList" :key="diet.id">
                      <td>{{ diet.type }}</td>
                      <td><img :src="diet.imagePath" style="height: 100px; width: auto;"></td>
                      <td>{{ diet.comment }}</td>
                      <td>{{ diet.dietDate }}</td>
                  </tr>
                  </tbody>
              </table>
          </div>

          <div class="flex justify-center -ml-44 relative top-1/3">
              <div class="relative grid grid-cols-1 gap-4 p-4 mb-8 border rounded-lg bg-white shadow-lg">
                  <div class="relative flex gap-4">
                      <img :src="trainerInfo.profileImage" class="relative rounded-lg -top-8 -mb-4 bg-white border h-20 w-20" alt="" loading="lazy">
                      <div class="flex flex-col w-full">
                          <div class="flex flex-row justify-between">
                              <p class="relative text-xl whitespace-nowrap truncate overflow-hidden">{{ trainerInfo.name}}</p>
                              <a class="text-gray-500 text-xl" href="#"><i class="fa-solid fa-trash"></i></a>
                          </div>
                          <p class="text-gray-400 text-sm">{{ feedback.createdTime }}</p>
                      </div>
                  </div>
                  <p class="-mt-4 text-gray-500">
                      {{ feedback.feedBack ? feedback.feedBack : '아직 피드백을 작성하지 않았네요!' }}
                  </p>
                  <br>
                  <p class="-mt-4 text-gray-500">
                      {{ feedback.rating }}
                  </p>
              </div>
            </div>
          </div>

          <div class="flex justify-center -ml-44 relative top-1/3" v-if="role=='MEMBER' || role == 'ADMIN'">
              <button @click="openDietModal" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
                식단 등록
              </button>
              <ModalComponent ref="modal"/>
          </div>
          <div class="flex justify-center -ml-44 relative top-1/3" v-if="role == 'TRAINER' || role == 'ADMIN'">
              <button @click="openDietModa2" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
                피드백 등록
              </button>
              <FeedbackComponent ref="moda2"/>
          </div> 
      </div>
</template>

<script>
import ModalComponent from './DietModalComponent.vue';
import FeedbackComponent from './DietFeedbackModalComponent.vue';
import axios from 'axios';
export default {
  data() {
      return {
          addDietList: [],
          feedback:[],
          trainerInfo:[],
          role:"",
          doneFeedbak:false,
      };
  },
  name: 'app',
  components: {
      ModalComponent,
      FeedbackComponent
  },
  created(){
      this.fetchDiets();
      this.fetchFeedback();
      this.fetchTrainer();
        this.role = localStorage.getItem("role");
  },  
  methods: {
    openDietModal() {
      this.$refs.modal.openModal();
    },
    openDietModa2() {
        if(this.doneFeedbak){
            confirm("피드백을 수정하시겠습니까?")
        }
        this.$refs.moda2.openModal();
    },
      async fetchDiets(){
          try{
            const urlParams = new URLSearchParams(window.location.search);
            const memberEmail = urlParams.get('memberEmail');
            const date = urlParams.get('date');
            const token = localStorage.getItem('token');
            const refreshToken = localStorage.getItem('refreshToken');
            const headers = token ? {Authorization: `Bearer ${token}`,refreshToken:`${refreshToken}`}:{};
            const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/diet/list/member?memberEmail=${memberEmail}&date=${date}`,{headers});
            // console.log(response);
            this.addDietList = response.data.result;
          }catch(error){
              console.log(error);
          }
      },
      async fetchFeedback(){
          try{
              const token = localStorage.getItem('token');
              const refreshToken = localStorage.getItem('refreshToken');
              const urlParams = new URLSearchParams(window.location.search);
              const date = urlParams.get('date');
              const headers = token ? {Authorization: `Bearer ${token}`,refreshToken:`${refreshToken}`}:{};
              if(localStorage.getItem('role')==="MEMBER"){
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/diet/feedback/member/find?date=${date}`,{headers});
                console.log(response);
                this.feedback = response.data.result;
                this.doneFeedbak = true;
              }else{
                const memberEmail = localStorage.getItem('accessEmail')
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/diet/feedback/trainer/find?date=${date}&memberEmail=${memberEmail}`,{headers});
                console.log(response);
                this.feedback = response.data.result;
                this.doneFeedbak = true;
              }
          }catch(error){
              console.log(error);
          }
      },
      async fetchTrainer(){
          try{
              const token = localStorage.getItem('token');
              const refreshToken = localStorage.getItem('refreshToken');
              const headers = token ? {Authorization: `Bearer ${token}`,refreshToken:`${refreshToken}`}:{};
              const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/trainer/find`,{headers});
              console.log(response);
              this.trainerInfo = response.data.result;
              console.log(this.trainerInfo.profileImage)
          }catch(error){
              console.log(error);
          }
      },
  }
  
}
</script>
<style>
* {
font-family: "Jua", sans-serif;
font-weight: 400;
font-style: normal;
}

.dietTitle {
margin-top: 30px;
text-align: center;
font-size: 24px;
margin-bottom: -50px;
letter-spacing: 15px;
font-size: 80px;
color: teal;
}

.content-wrapper {
display: grid;
grid-template-columns: 1fr 1fr;
gap: 20px;
margin: 30px 0;
}

.table, .trainer-info {
margin-top: 100px;
width: auto;
min-width: 0;
border-collapse: collapse;
}

th, td {
padding: 12px 15px;
text-align: left;
}

thead {
background-color: #f4f4f4;
color: #333;
}

tbody tr:nth-child(odd) {
background-color: #fff;
}

tbody tr:nth-child(even) {
background-color: #f9f9f9;
}

tbody tr:hover {
background-color: #f1f1f1;
}

.circle-gradient {
animation: rotate 10s linear infinite;
transform-origin: center;
}

@keyframes rotate {
100% {
  transform: rotate(360deg);
}
}
</style>