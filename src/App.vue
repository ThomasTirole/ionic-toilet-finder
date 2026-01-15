<template>
  <ion-app>
    <ion-split-pane content-id="main-content">
      <ion-menu content-id="main-content" type="overlay">
        <ion-content>
          <ion-list id="inbox-list">
            <ion-list-header>Menu</ion-list-header>
            <ion-note>Toilet Finder App</ion-note>

            <ion-menu-toggle :auto-hide="false" v-for="(p, i) in appPages" :key="i">
              <ion-item @click="selectedIndex = i" router-direction="root" :router-link="p.url" lines="none" :detail="false" class="hydrated" :class="{ selected: selectedIndex === i }">
                <ion-icon aria-hidden="true" slot="start" :ios="p.iosIcon" :md="p.mdIcon"></ion-icon>
                <ion-label>{{ p.title }}</ion-label>
              </ion-item>
            </ion-menu-toggle>
            
            <ion-menu-toggle :auto-hide="false" v-if="user">
               <ion-item button @click="handleLogout" lines="none" :detail="false">
                  <ion-icon aria-hidden="true" slot="start" :ios="logOutOutline" :md="logOutSharp"></ion-icon>
                  <ion-label>Logout</ion-label>
               </ion-item>
            </ion-menu-toggle>
          </ion-list>
        </ion-content>
      </ion-menu>
      <ion-router-outlet id="main-content"></ion-router-outlet>
    </ion-split-pane>
  </ion-app>
</template>

<script setup lang="ts">
import {
  IonApp,
  IonContent,
  IonIcon,
  IonItem,
  IonLabel,
  IonList,
  IonListHeader,
  IonMenu,
  IonMenuToggle,
  IonNote,
  IonRouterOutlet,
  IonSplitPane,
} from '@ionic/vue';
import { ref, onMounted } from 'vue';
import {
  personOutline,
  personSharp,
  mapOutline,
  mapSharp,
  logOutOutline,
  logOutSharp
} from 'ionicons/icons';
import { supabase } from './supabase';
import { useRouter } from 'vue-router';

const router = useRouter();
const selectedIndex = ref(0);
const user = ref<any>(null);

const appPages = [
  {
    title: 'Map',
    url: '/home',
    iosIcon: mapOutline,
    mdIcon: mapSharp,
  },
  {
    title: 'My Account',
    url: '/profile',
    iosIcon: personOutline,
    mdIcon: personSharp,
  },
];

onMounted(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      user.value = session?.user ?? null;
    });

    supabase.auth.onAuthStateChange((_, session) => {
      user.value = session?.user ?? null;
    });
});

const handleLogout = async () => {
  await supabase.auth.signOut();
  router.push('/login');
};

const path = window.location.pathname;
if (path !== undefined) {
  selectedIndex.value = appPages.findIndex((page) => page.url === path);
}
</script>

<style scoped>
ion-menu ion-content {
  --background: var(--ion-item-background, var(--ion-background-color, #fff));
}

ion-item {
  --padding-start: 16px;
  --padding-end: 16px;
  --min-height: 50px;
}

ion-item.selected {
  --color: var(--ion-color-primary);
}

ion-item.selected ion-icon {
  color: var(--ion-color-primary);
}
</style>
