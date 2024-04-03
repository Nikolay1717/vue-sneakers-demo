<script setup>
import Card from './Card.vue';

defineProps({
  items: Array,
  isFavorites: Boolean,
});

const emit = defineEmits(['addToFavorites', 'addToCart']);
</script>

<template>
  <div class="grid grid-cols-4 gap-5" v-auto-animate>
    <Card
      v-for="item in items"
      :key="item.id"
      :id="item.id"
      :title="item.title"
      :image-url="item.imageUrl"
      :price="item.price"
      :is-favorite="item.isFavorite"
      :favorite-id="item.favoriteId"
      :on-click-add="isFavorites ? null : () => emit('addToCart', item)"
      :on-click-favorite="isFavorites ? null : () => emit('addToFavorites', item)"
      :is-added="item.isAdded"
    />
  </div>
</template>
