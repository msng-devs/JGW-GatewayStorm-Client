<template>

</template>

<script setup>
import {storeToRefs} from "pinia";
import {inject, onBeforeMount, onMounted, ref, watch} from 'vue';
import { onAuthStateChanged } from "firebase/auth";
import {auth} from '../firebase.config'
import router from "../router.js";
import {signOut} from "firebase/auth";

import {useAuthStore} from "../store/auth.store";
const authStore = useAuthStore();
authStore.loadInfoFromLocalStorage();
const {isLogin} = storeToRefs(authStore)


const handleLoginChange = (newLoginStatus) => {
  if(!newLoginStatus) {
      router.push({name: 'Login'});
  }
};

watch(isLogin, handleLoginChange,{ immediate: true });

const emits = defineEmits(['onError'])

const axios = inject('axios')

const checkServer = async (token) => {
  try {
    await axios.get('/api/v1/ping/auth', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });


    return true;
  } catch (error) {
    if (error.response) {
      switch (error.response.status) {
        case 403:

          emits('onError', '해당 시스템에 접근할 권한이 없습니다. 관리자에게 문의하세요 ');
          break;
        case 404:
        case 500:

          emits('onError', '서버 연결중 에러가 발생했습니다. 다시 시도해 주세요.')
          break;

        default:
          emits('onError', '알 수 없는 에러가 발생했습니다. 다시 시도해 주세요.')
          break;
      }
    } else if (error.request) {
      emits('onError', '서버에 연결할 수 없습니다. 네트워크 상태를 확인해주세요.')
    }

    return false;
  }
}
auth.onIdTokenChanged((user) => {
  if (user) {
    user.getIdToken().then((accessToken) => {

      token.value = accessToken
      authStore.setAccessToken(accessToken)
    });
  }
});

const token = ref(null)
onMounted(async () => {
  await onAuthStateChanged(auth,async (user) => {
    if(user){
      token.value = await user.getIdToken()
      const status = await checkServer(token.value)
      if(!status) {
        authStore.setLogin(false)
        await signOut(auth);


      } else{
        authStore.setAccessToken(token.value)

      }
    }
    else {
      authStore.setLogin(false)
    }
  })

})



</script>

<style scoped>

</style>