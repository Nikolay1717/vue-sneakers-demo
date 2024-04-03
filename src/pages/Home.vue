<script setup>
import axios from 'axios';
import debounce from 'lodash.debounce';

import CardList from '../components/CardList.vue';
import { inject, reactive, watch, ref, onMounted } from 'vue';

const items = ref([]);
const { cartItems, addToCart, removeFromCart } = inject('cart');
const filters = reactive({
  sortBy: 'title',
  searchQuery: '',
});

const onChangeSelect = (event) => {
  filters.sortBy = event.target.value;
};

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value;
}, 300);

const addToFavorites = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id,
      };
      const { data } = await axios.post('https://662e98819836f729.mokky.dev/favorites', obj);
      item.isFavorite = true;
      item.favoriteId = data.id;
    } else {
      await axios.delete(`https://662e98819836f729.mokky.dev/favorites/${item.favoriteId}`);
      item.isFavorite = false;
      item.favoriteId = null;
    }
  } catch (error) {
    console.log(error);
  }
};

const onClickAddToCart = (item) => {
  if (!item.isAdded) {
    addToCart(item);
  } else {
    removeFromCart(item);
  }
};

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get('https://662e98819836f729.mokky.dev/favorites');
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id);
      if (!favorite) {
        return item;
      }
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id,
      };
    });
  } catch (error) {
    console.log(error);
  }
};

const fetchItems = async () => {
  const params = {
    sortBy: filters.sortBy,
  };
  if (filters.searchQuery) {
    params.title = `*${filters.searchQuery}*`;
  }
  try {
    const { data } = await axios.get('https://662e98819836f729.mokky.dev/items', {
      params,
    });
    items.value = data.map((item) => ({
      ...item,
      isFavorite: false,
      favoriteId: null,
      isAdded: false,
    }));
  } catch (error) {
    console.log(error);
  }
};

onMounted(async () => {
  cartItems.value = JSON.parse(localStorage.getItem('cartItems')) || [];

  await fetchItems();
  await fetchFavorites();

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cartItems.value.some((cartItem) => cartItem.id === item.id),
  }));
});

watch(filters, fetchItems);

watch(cartItems, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false,
  }));
});
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>
    <div class="flex gap-4">
      <select
        @change="onChangeSelect"
        class="py-2 px-3 border rounded-md outline-none"
        name="filter"
        id="filter"
      >
        <option value="name">По названию</option>
        <option value="price">По цене (дешёвые)</option>
        <option value="-price">По цене (дорогие)</option>
      </select>

      <div class="relative">
        <img class="absolute left-3 top-3" src="/search.svg" alt="" />
        <input
          @input="onChangeSearchInput"
          class="border rounded-md py-1.5 pl-10 pr-4 outline-none focus:border-gray-400"
          placeholder="Поиск..."
          type="text"
        />
      </div>
    </div>
  </div>
  <div class="mt-6">
    <CardList :items="items" @add-to-favorites="addToFavorites" @add-to-cart="onClickAddToCart" />
  </div>
</template>
