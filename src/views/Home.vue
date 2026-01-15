<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
             <ion-menu-button></ion-menu-button>
        </ion-buttons>
        <ion-title>Toilet Finder</ion-title>
      </ion-toolbar>

        <!-- Info Banner during Itinerary -->
        <div v-if="itineraryTarget" class="itinerary-banner">
            <div class="itinerary-info">
                <span>Navigating to: <strong>{{ itineraryTarget.name }}</strong></span>
                <ion-button size="small" fill="clear" color="light" @click="itineraryTarget = null">
                    <ion-icon name="close-circle"></ion-icon> Cancel
                </ion-button>
            </div>
        </div>
    </ion-header>
    
    <ion-content :fullscreen="true">
        <div class="content-container">
            <MapContainer 
                :toilets="toilets" 
                :userLocation="userLocation"
                :selectedToilet="selectedToilet"
                :itineraryTarget="itineraryTarget"
                @markerClick="handleMarkerClick"
            />
        </div>
        
        <!-- Bottom Drawer -->
        <ion-modal 
            :is-open="true" 
            :initial-breakpoint="0.25" 
            :breakpoints="[0.1, 0.25, 0.5, 0.9]"
            :backdrop-dismiss="false"
            :show-backdrop="false"
            class="bottom-drawer"
        >
            <ToiletDrawer 
                :toilets="toilets"
                :userLocation="userLocation"
                :selectedId="selectedToilet?.id"
                @select="handleListSelect"
                @startItinerary="handleStartItinerary"
            />
        </ion-modal>
        


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
    IonModal,
    IonButton,
    IonIcon
} from '@ionic/vue';
import { ref, onMounted } from 'vue';
import { Geolocation } from '@capacitor/geolocation';
import { supabase } from '../supabase';
import MapContainer from '../components/MapContainer.vue';
import ToiletDrawer from '../components/ToiletDrawer.vue';

const toilets = ref<any[]>([]);
const userLocation = ref<{ lat: number; lng: number } | null>(null);
const selectedToilet = ref<any>(null);
const itineraryTarget = ref<any>(null);

onMounted(async () => {
    await fetchToilets();
    await getCurrentPosition();
});

const fetchToilets = async () => {
    const { data, error } = await supabase.from('toilets').select('*');
    if (error) {
        console.error('Error fetching toilets:', error);
    } else {
        toilets.value = data || [];
    }
};

const getCurrentPosition = async () => {
    try {
        const coordinates = await Geolocation.getCurrentPosition();
        userLocation.value = {
            lat: coordinates.coords.latitude,
            lng: coordinates.coords.longitude
        };
    } catch (e) {
        console.error('Error getting location', e);
        // Fallback or alert user
    }
};

const handleMarkerClick = (toilet: any) => {
    selectedToilet.value = toilet;
    // We might want to expand the modal here, but for now let's just select it
};

const handleListSelect = (toilet: any) => {
    selectedToilet.value = toilet;
};

const handleStartItinerary = (toilet: any) => {
    itineraryTarget.value = toilet;
    // Logic to maybe zoom out to fit both user and target is handled in MapContainer watcher
};
</script>

<style scoped>
.content-container {
    height: 100%;
    width: 100%;
    position: relative;
}

.itinerary-banner {
    position: relative; /* Normal flow in header */
    width: 100%;
    background: var(--ion-color-primary);
    color: white;
    padding: 10px;
    z-index: 1000;
}
/* To make the banner appear above the map but below the drawer when drawer is high,
   or effectively, we just want it to be visible. 
   Actually, when itinerary is active, we probably want the drawer minimized or hidden?
   The prompt screenshot 4 shows "35m restant" at the bottom. 
   The drawer might just be the banner content in that state.
   For simplicity, I will stick to the drawer logic, and maybe the user collapses it.
   
   The provided screenshot 4 shows the drawer is minimized to just the bottom bar essentially.
*/

.itinerary-info {
    display: flex;
    justify-content: space-between;
    align-items: center;
}
</style>
