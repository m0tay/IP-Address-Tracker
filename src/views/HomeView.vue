<template>
  <div class="bg-slate-200 flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32 shadow-2xl">
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input v-model="queryIP" class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" type="text"
            placeholder="Search for any IP address or leave empty to get your IP info">
          <i @click="getIpInfo"
            class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fa-solid fa-chevron-right"></i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>
    <!-- Map -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from "../components/IPInfo.vue";
import leaflet from 'leaflet';
import { onMounted, ref } from "vue";
import axios from 'axios';

export default {
  name: 'HomeView',
  components: { IPInfo },
  setup() {
    let appMap;
    const queryIP = ref("");
    const ipInfo = ref(null);

    const minLatitude = -40;
    const maxLatitude = 50;
    const minLongitude = -80;
    const maxLongitude = 100;

    const randomLatitude = Math.random() * (maxLatitude - minLatitude) + minLatitude;
    const randomLongitude = Math.random() * (maxLongitude - minLongitude) + minLongitude;


    onMounted(() => {
      appMap = leaflet.map('map').setView([randomLatitude, randomLongitude], 4);
      leaflet.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }).addTo(appMap);
    })

    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country,city?apiKey=at_gDMQO1gDCPrGmTBnuS3TTbtzHaaHt&ipAddress=${queryIP.value}`
        )
        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(appMap);
        appMap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      }
      catch (err) {
        alert(err.message);
      }
    }
    return { queryIP, ipInfo, getIpInfo };
  },
}
</script>
