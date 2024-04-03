<script setup>
import CartItem from './CartItem.vue';
import { inject } from 'vue';

const { cartItems, removeFromCart } = inject('cart');
</script>

<template>
  <div class="flex flex-col gap-4 flex-1 justify-between" v-auto-animate>
    <CartItem
      v-for="item in cartItems"
      :key="item.id"
      :title="item.title"
      :image-url="item.imageUrl"
      :price="item.price"
      @on-click-remove="() => removeFromCart(item)"
    />
  </div>
</template>
