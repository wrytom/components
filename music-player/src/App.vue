<script setup>
import Player from './components/Player.vue';
import ThumbnailImage from './assets/images/mono.jpg';

const songs = [
  { title: "Shokolad", artist: "Ostava", duration: "3:31", thumbnail: ThumbnailImage },
  { title: "Ponedelnik", artist: "Ostava", duration: "3:32", thumbnail: ThumbnailImage },
  { title: "President", artist: "Ostava", duration: "3:30", thumbnail: ThumbnailImage },
];
</script>

<template>
  <Player :songs="songs"/>
</template>
