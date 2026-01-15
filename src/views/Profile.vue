<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button></ion-menu-button>
        </ion-buttons>
        <ion-title>My Account</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
        <div v-if="user">
            <ion-card>
                <ion-card-header>
                    <ion-card-title>User Profile</ion-card-title>
                    <ion-card-subtitle>{{ user.email }}</ion-card-subtitle>
                </ion-card-header>
                <ion-card-content>
                    <p>User ID: {{ user.id }}</p>
                    <p>Last Sign In: {{ new Date(user.last_sign_in_at).toLocaleString() }}</p>
                </ion-card-content>
            </ion-card>
            
            <ion-button expand="block" color="danger" @click="handleLogout">Logout</ion-button>
        </div>
        <div v-else class="ion-text-center">
            <p>You are not logged in.</p>
            <ion-button router-link="/login">Go to Login</ion-button>
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
  IonButtons,
  IonMenuButton,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonButton
} from '@ionic/vue';
import { ref, onMounted } from 'vue';
import { supabase } from '../supabase';
import { useRouter } from 'vue-router';

const user = ref<any>(null);
const router = useRouter();

onMounted(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
        user.value = session?.user;
    });
});

const handleLogout = async () => {
    await supabase.auth.signOut();
    router.push('/login');
};
</script>
