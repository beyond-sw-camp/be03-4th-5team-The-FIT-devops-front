<template>
  <header v-if="isLogin" class="header">
    <div class="header-content">
      <div class="header-buttons-left">
        <button class="header-button" @click="goToCalendar">캘린더</button>
        <button @click="viewTrainerInfo" class="header-button" v-if="userRole === 'MEMBER'">나의 트레이너 보기</button>
        <button @click="viewMemberInfo" class="header-button" v-if="userRole === 'TRAINER'">나의 트레이니 보기</button>
        <!-- <button @click="userInfo" class="header-button" v-if="userRole === 'ADMIN'">회원 조회</button> -->
      </div>
      
      <h1 class="header-title">THE FIT</h1>
      
      <!-- 로그아웃 및 내정보 버튼을 감싸는 div 추가 가능 -->
      <div class="header-buttons-right">
        <button class="header-button" @click="goToMyInfo">내정보</button>
        <button class="header-button" @click="logout">로그아웃</button>
      </div>
    </div>
  </header>
  <TrainerModal :isVisible="isModalVisible" :trainer="trainer" @close="closeModal"/>
  <MemberListModal :isVisible="isModalVisible" :members="members" @close="isModalVisible = false"/>
</template>
  
  <script>
  import { ref } from 'vue'; 
  import { useRouter } from 'vue-router'; 
  import TrainerModal from './TrainerModal.vue';
  import MemberListModal from './MemberListModal.vue';
  import axios from 'axios'; // Axios import 추가
  
  export default {
    name: 'HeaderComponents',
    components: {
      TrainerModal,
      MemberListModal
    },
      setup() {
      const router = useRouter();
      const isLogin = ref(!!localStorage.getItem("token"));
      const userRole = ref(localStorage.getItem("role"));
      const isModalVisible = ref(false);
      const trainers = ref([]);
      const members = ref([]); // 회원 목록을 위한 상태도 ref를 사용하여 선언
  
      function goToCalendar() {
        router.push({ name: 'CalendarComponent' });
      }
    
      function goToMyInfo() {
        if (userRole.value === 'TRAINER') {
          router.push({ name: 'MyPageTrainerComponent' });
        } else if (userRole.value === 'MEMBER') {
          router.push({ name: 'MyPageComponent' });
        }
      }
    
      function logout() {
        localStorage.removeItem("token");
        localStorage.removeItem("role");
        localStorage.removeItem("email");
        localStorage.removeItem("refreshToken");
        isLogin.value = false;
        userRole.value = null;
        router.push({ name: 'HOME' });
      }

      async function userInfo() {
      try {
        const token = localStorage.getItem('token');
        const headers = token ? { Authorization: `Bearer ${token}` } : {};
        const memberResponse = await axios.get(`http://localhost:8080/member/list`, { headers });
        const trainerResponse = await axios.get(`http://localhost:8080/trainer/list`, { headers });
        members.value = memberResponse.data; // 회원 목록 데이터 설정
        trainers.value = trainerResponse.data; // 트레이너 목록 데이터 설정
        isModalVisible.value = true; // 모달 표시
      } catch (error) {
        console.error('회원 및 트레이너 정보 불러오기 중 에러가 발생했습니다.', error);
        alert('회원 및 트레이너 정보 불러오기에 실패했습니다.');
      }
    }

        async function viewMemberInfo() {
      try {
        const token = localStorage.getItem('token');
        const headers = token ? { Authorization: `Bearer ${token}` } : {};
        const response = await axios.get(`http://localhost:8080/trainer/my/members`, { headers });
        members.value = response.data.result; // 'members' 상태를 직접 업데이트
        isModalVisible.value = true; // 모달 표시
      } catch (error) {
        console.error('회원 정보 불러오기 중 에러가 발생했습니다.', error);
        alert('회원 정보 불러오기에 실패했습니다.');
      }
    }

    async function viewTrainerInfo() {
      try {
        const token = localStorage.getItem('token');
        const headers = token ? { Authorization: `Bearer ${token}` } : {};
        const response = await axios.get(`http://localhost:8080/member/my/trainer`, { headers });
        trainers.value = response.data.result; // 'trainers' 상태를 직접 업데이트
        isModalVisible.value = true; // 모달 표시
      } catch (error) {
        console.error('트레이너 정보 불러오기 중 에러가 발생했습니다.', error);
        alert('트레이너 정보 불러오기에 실패했습니다.');
      }
    }

      function closeModal() {
        isModalVisible.value = false; // 모달 닫기
      }
    
      return {
        goToCalendar,
        goToMyInfo,
        logout,
        viewTrainerInfo,
        viewMemberInfo,
        userInfo,
        closeModal,
        isLogin,
        userRole,
        isModalVisible,
        trainers,
        members,
      
      };
    },
  }
  </script>
  
  <style scoped>
  .header {
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(255, 255, 255, 0.8); 
    padding: 10px;
    position: relative; 
  }
  
  .header-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    max-width: 1200px; 
  }
  
  .header-title {
    color: #ABECD6; 
    font-size: 24px;
  }
  
  .header-button {
    background-color: #BC96FB; 
    color: white; 
    border: none;
    padding: 10px 20px;
    margin: 0 10px;
    cursor: pointer;
    border-radius: 5px; 
    transition: background-color 0.3s ease; 
  }
  
  .header-button:hover {
    background-color: #ABECD6; 
  }
  </style>