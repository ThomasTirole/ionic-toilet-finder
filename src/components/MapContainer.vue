<template>
  <div id="map" class="map-container"></div>
</template>

<script setup lang="ts">
import { onMounted, watch, ref } from 'vue';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

// Fix for default marker icons in Leaflet with Vite/Webpack
import markerIcon2x from 'leaflet/dist/images/marker-icon-2x.png';
import markerIcon from 'leaflet/dist/images/marker-icon.png';
import markerShadow from 'leaflet/dist/images/marker-shadow.png';

delete (L.Icon.Default.prototype as any)._getIconUrl;
L.Icon.Default.mergeOptions({
  iconRetinaUrl: markerIcon2x,
  iconUrl: markerIcon,
  shadowUrl: markerShadow,
});

const props = defineProps<{
  toilets: any[];
  userLocation: { lat: number; lng: number } | null;
  selectedToilet: any | null;
  itineraryTarget: any | null;
}>();

const emit = defineEmits(['markerClick']);

let map: L.Map | null = null;
let userMarker: L.Marker | null = null;
let toiletMarkers: L.Marker[] = [];
let routePolyline: L.Polyline | null = null;

// Custom Icons
const toiletIcon = L.icon({
    iconUrl: 'https://cdn-icons-png.flaticon.com/512/1257/1257334.png', // Placeholder toilet icon
    iconSize: [32, 32],
    iconAnchor: [16, 32],
    popupAnchor: [0, -32]
});

const selectedIcon = L.icon({
    iconUrl: 'https://cdn-icons-png.flaticon.com/512/1257/1257334.png',
    iconSize: [40, 40], // Slightly larger
    className: 'selected-marker',
    iconAnchor: [20, 40],
});

const userIcon = L.divIcon({
    className: 'user-marker',
    html: '<div style="background-color: #4285F4; width: 15px; height: 15px; border-radius: 50%; border: 2px solid white; box-shadow: 0 0 5px rgba(0,0,0,0.5);"></div>',
    iconSize: [20, 20],
    iconAnchor: [10, 10]
});


onMounted(() => {
  // Default to a central location if no user location yet (e.g., Delémont, CH based on prompt Screenshot addresses)
  const initialLat = 47.3686;
  const initialLng = 7.3456;
  
  map = L.map('map', {
      zoomControl: false
  }).setView([initialLat, initialLng], 15);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);
  
  // Reposition zoom control
  L.control.zoom({
     position: 'topright'
  }).addTo(map);

  updateMarkers();
});

watch(() => props.toilets, () => {
    updateMarkers();
}, { deep: true });

watch(() => props.userLocation, (newLoc) => {
    if (map && newLoc) {
        if (userMarker) {
            userMarker.setLatLng([newLoc.lat, newLoc.lng]);
        } else {
            userMarker = L.marker([newLoc.lat, newLoc.lng], { icon: userIcon }).addTo(map);
        }
    }
}, { deep: true });

watch(() => props.selectedToilet, (toilet) => {
    if (map && toilet) {
        map.setView([toilet.lat, toilet.lng], 16);
        // We could also highlight the marker here
        updateMarkers(); // Re-render to update icons
    }
});

watch(() => props.itineraryTarget, async (target) => {
    if (map && props.userLocation && target) {
        if (routePolyline) {
            map.removeLayer(routePolyline);
        }
        
        try {
            // Fetch route from OSRM
            const response = await fetch(`http://router.project-osrm.org/route/v1/walking/${props.userLocation.lng},${props.userLocation.lat};${target.lng},${target.lat}?overview=full&geometries=geojson`);
            const data = await response.json();

            if (data.routes && data.routes.length > 0) {
                const coordinates = data.routes[0].geometry.coordinates.map((coord: number[]) => [coord[1], coord[0]]);
                
                routePolyline = L.polyline(coordinates, {color: 'blue', weight: 4}).addTo(map);
                map.fitBounds(routePolyline.getBounds(), { padding: [50, 50] });
            } else {
                 throw new Error('No route found');
            }
        } catch (e) {
            console.error('Routing failed, falling back to straight line', e);
             // Fallback to straight line
            const latlngs = [
                [props.userLocation.lat, props.userLocation.lng],
                [target.lat, target.lng]
            ];
            
            routePolyline = L.polyline(latlngs as L.LatLngExpression[], {color: 'blue', dashArray: '10, 10', weight: 4}).addTo(map);
            map.fitBounds(routePolyline.getBounds(), { padding: [50, 50] });
        }

    } else if (routePolyline && !target) {
        map?.removeLayer(routePolyline);
    }
});

function updateMarkers() {
    if (!map) return;
    
    // Clear existing markers
    toiletMarkers.forEach(m => map!.removeLayer(m));
    toiletMarkers = [];
    
    props.toilets.forEach(t => {
        const isSelected = props.selectedToilet && props.selectedToilet.id === t.id;
        const icon = isSelected ? selectedIcon : toiletIcon;
        
        const marker = L.marker([t.lat, t.lng], { icon })
            .addTo(map!)
            .on('click', () => {
                emit('markerClick', t);
            });
            
        toiletMarkers.push(marker);
    });
}
</script>

<style>
.map-container {
  width: 100%;
  height: 100%;
  z-index: 1;
}
.selected-marker {
    filter: hue-rotate(120deg); /* Change color for selected */
}
</style>
