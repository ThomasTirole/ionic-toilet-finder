<template>
  <div class="drawer-container">
      <!-- Handle for dragging (visual only for now as we use standard overflow) -->
      <div class="drawer-handle-bar">
          <div class="drawer-handle"></div>
      </div>

    <ion-content>
      <ion-list>
          <ion-item v-for="toilet in toilets" :key="toilet.id" button @click="$emit('select', toilet)" :color="selectedId === toilet.id ? 'light' : ''">
              <div class="toilet-item">
                  <div class="toilet-info">
                      <h3>
                          <ion-icon :icon="waterOutline" class="toilet-icon"></ion-icon> 
                          {{ toilet.name }}
                      </h3>
                      <p>{{ toilet.address }}</p>
                      
                      <div class="amenities">
                           <span v-if="toilet.is_accessible" title="Accessible"><ion-icon :icon="bodyOutline"></ion-icon></span>
                           <span v-if="toilet.price === 0 || toilet.price === null" title="Free">Free</span>
                           <span v-else>{{ toilet.price }} CHF</span>
                      </div>
                  </div>
                  <div class="toilet-action">
                      <div class="distance" v-if="userLocation">
                          {{ calculateDistance(userLocation.lat, userLocation.lng, toilet.lat, toilet.lng) }}m
                      </div>
                      <ion-button v-if="selectedId === toilet.id" size="small" @click.stop="$emit('startItinerary', toilet)">
                          GO !
                      </ion-button>
                      <ion-icon v-else :icon="navigateOutline" class="nav-icon"></ion-icon>
                  </div>
              </div>
          </ion-item>
      </ion-list>
    </ion-content>
  </div>
</template>

<script setup lang="ts">
import { IonList, IonItem, IonIcon, IonContent, IonButton } from '@ionic/vue';
import { waterOutline, bodyOutline, navigateOutline } from 'ionicons/icons';

const props = defineProps<{
    toilets: any[];
    userLocation: { lat: number; lng: number } | null;
    selectedId: number | null;
}>();

const emit = defineEmits(['select', 'startItinerary']);

// Haversine formula to calculate distance
function calculateDistance(lat1: number, lon1: number, lat2: number, lon2: number) {
  const R = 6371e3; // metres
  const φ1 = lat1 * Math.PI/180; // φ, λ in radians
  const φ2 = lat2 * Math.PI/180;
  const Δφ = (lat2-lat1) * Math.PI/180;
  const Δλ = (lon2-lon1) * Math.PI/180;

  const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
            Math.cos(φ1) * Math.cos(φ2) *
            Math.sin(Δλ/2) * Math.sin(Δλ/2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

  return Math.round(R * c);
}
</script>

<style scoped>
.drawer-container {
    height: 100%;
    background: white;
    display: flex;
    flex-direction: column;
}
.drawer-handle-bar {
    width: 100%;
    height: 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    background: white;
    border-top-left-radius: 16px;
    border-top-right-radius: 16px;
}
.drawer-handle {
    width: 40px;
    height: 4px;
    background: #e0e0e0;
    border-radius: 2px;
}
.toilet-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    padding: 10px 0;
}
.toilet-info h3 {
    margin: 0;
    font-size: 16px;
    font-weight: bold;
    display: flex;
    align-items: center;
    gap: 8px;
}
.toilet-info p {
    margin: 4px 0;
    color: #666;
    font-size: 14px;
}
.amenities {
    display: flex;
    gap: 10px;
    font-size: 12px;
    color: #888;
    margin-top: 5px;
}
.toilet-action {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 5px;
    min-width: 80px;
}
.distance {
    font-weight: bold;
    font-size: 14px;
}
.nav-icon {
    font-size: 24px;
    color: var(--ion-color-primary);
}
</style>
