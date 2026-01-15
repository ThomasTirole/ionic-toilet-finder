<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button></ion-menu-button>
        </ion-buttons>
        <ion-title>Login</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div v-if="loading" class="spinner-container">
        <ion-spinner></ion-spinner>
      </div>

      <div v-else class="login-container">
        <h1>Welcome Back</h1>
        <ion-item>
          <ion-input label="Email" label-placement="floating" v-model="email" type="email" placeholder="email@domain.com"></ion-input>
        </ion-item>
        <ion-item>
          <ion-input label="Password" label-placement="floating" v-model="password" type="password"></ion-input>
        </ion-item>

        <div class="ion-padding-top">
            <ion-button expand="block" @click="handleLogin">Login</ion-button>
            <ion-button expand="block" fill="outline" @click="handleSignUp">Sign Up</ion-button>
        </div>

        <ion-toast
          :is-open="isOpen"
          :message="toastMessage"
          :duration="2000"
          @didDismiss="isOpen = false"
        ></ion-toast>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonItem,
  IonInput,
  IonButton,
  IonSpinner,
  IonButtons,
  IonMenuButton,
  IonToast
} from '@ionic/vue';
import { ref } from 'vue';
import { supabase } from '../supabase';
import { useRouter } from 'vue-router';

const email = ref('');
const password = ref('');
const loading = ref(false);
const router = useRouter();
const isOpen = ref(false);
const toastMessage = ref('');

const handleLogin = async () => {
  loading.value = true;
  const { error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  });

  if (error) {
    toastMessage.value = error.message;
    isOpen.value = true;
  } else {
    router.push('/home');
  }
  loading.value = false;
};

const handleSignUp = async () => {
  loading.value = true;
  const { error } = await supabase.auth.signUp({
    email: email.value,
    password: password.value,
  });

  if (error) {
    toastMessage.value = error.message;
    isOpen.value = true;
  } else {
    toastMessage.value = 'Check your email for the login link!';
    isOpen.value = true;
  }
  loading.value = false;
};
</script>

<style scoped>
.login-container {
    max-width: 400px;
    margin: 0 auto;
    padding-top: 50px;
}
.spinner-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
}
</style>
