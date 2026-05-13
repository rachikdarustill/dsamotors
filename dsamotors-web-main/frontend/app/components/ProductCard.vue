<script setup lang="ts">
defineProps<{
  product: {
    id: string;
    title: string;
    category: string;
    make?: string;
    price: number;
    oem?: string;
    images?: string[];
  };
}>();

const formatRub = (value: number) =>
  `${Math.round(value || 0).toLocaleString('ru-RU')} ₽`;
</script>

<template>
  <NuxtLink :to="`/product/${product.id}`" class="card" :aria-label="`Открыть товар: ${product.title}`">
    <img
      class="product-image"
      :src="product.images?.[0] || '/no-image.svg'"
      :alt="product.title"
      loading="lazy"
      @error="(event) => ((event.target as HTMLImageElement).src = '/no-image.svg')"
    >
    <div class="pill">{{ product.category || 'Запчасть' }}</div>
    <div class="product-title">{{ product.title }}</div>
    <div><span class="price">{{ formatRub(product.price) }}</span></div>
    <div v-if="product.make" class="note" style="font-size: 11px; margin-top: 6px;">
      Бренд авто: {{ product.make }}
    </div>
    <div v-if="product.oem" class="note" style="font-size: 11px; margin-top: 6px;">
      OEM: {{ product.oem }}
    </div>
  </NuxtLink>
</template>
