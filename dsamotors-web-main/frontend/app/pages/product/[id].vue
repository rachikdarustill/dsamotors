<script setup lang="ts">
type Product = {
  id: string;
  category: string;
  condition?: string;
  address?: string;
  title: string;
  description?: string;
  brand?: string;
  make?: string;
  price: number;
  oem?: string;
  images?: string[];
};

const route = useRoute();
const { get, post } = useApi();

const email = ref('');
const phone = ref('');
const error = ref('');
const loading = ref(false);
const activeImage = ref('');

const { data: product, pending } = await useAsyncData(
  `product-${route.params.id}`,
  () => get<Product>(`/products/${route.params.id}`),
);

const { data: products } = await useAsyncData(
  'products-similar',
  () => get<Product[]>('/products'),
);

watchEffect(() => {
  if (product.value && !activeImage.value) {
    activeImage.value = product.value.images?.[0] || '/no-image.svg';
  }
});

const formatRub = (value: number) =>
  `${Math.round(value || 0).toLocaleString('ru-RU')} ₽`;

const images = computed(() => {
  const values = product.value?.images?.filter(Boolean) || [];
  return values.length ? values : ['/no-image.svg'];
});

const similarProducts = computed(() => {
  const current = product.value;
  if (!current) return [];
  const currentCategory = String(current.category || '').trim().toLowerCase();
  const currentMake = String(current.make || '').trim().toLowerCase();
  if (!currentCategory || !currentMake) return [];

  return (products.value || [])
    .filter((item) => (
      item.id !== current.id
      && String(item.category || '').trim().toLowerCase() === currentCategory
      && String(item.make || '').trim().toLowerCase() === currentMake
    ))
    .slice(0, 4);
});

const setFallbackImage = (event: Event) => {
  const image = event.target as HTMLImageElement;
  image.src = '/no-image.svg';
};

const pay = async () => {
  error.value = '';
  loading.value = true;

  try {
    const order = await post<{ paymentUrl?: string }>('/orders', {
      productId: route.params.id,
      email: email.value || undefined,
      phone: phone.value || undefined,
    });

    if (order.paymentUrl) {
      await navigateTo(order.paymentUrl, { external: true });
      return;
    }

    error.value = 'Не удалось получить ссылку на оплату.';
  } catch (err: any) {
    error.value = err?.data?.message || 'Ошибка создания заказа.';
  } finally {
    loading.value = false;
  }
};
</script>

<template>
  <main class="page">
    <div class="container">
      <nav class="breadcrumbs" aria-label="Хлебные крошки">
        <NuxtLink to="/">Главная</NuxtLink>
        <span>/</span>
        <NuxtLink to="/catalog">Каталог</NuxtLink>
        <span>/</span>
        <span>Карточка товара</span>
      </nav>

      <div v-if="pending" class="panel">Загружаю товар…</div>

      <section v-else-if="product" class="product">
        <div class="product-side">
          <div class="panel gallery-panel">
            <img
              class="main-image"
              :src="activeImage || images[0]"
              :alt="product.title"
              @error="setFallbackImage"
            >
            <div v-if="images.length > 1" class="thumbs" role="list">
              <button
                v-for="image in images.slice(0, 12)"
                :key="image"
                class="thumb"
                type="button"
                :aria-current="image === activeImage ? 'true' : 'false'"
                @click="activeImage = image"
              >
                <img :src="image" alt="" loading="lazy" @error="setFallbackImage">
              </button>
            </div>
          </div>

          <div v-if="similarProducts.length" class="panel">
            <h2 class="similar-heading">Похожие</h2>
            <div class="similar-list">
              <NuxtLink
                v-for="item in similarProducts"
                :key="item.id"
                class="similar-card"
                :to="`/product/${item.id}`"
              >
                <img
                  :src="item.images?.[0] || '/no-image.svg'"
                  :alt="item.title"
                  loading="lazy"
                  @error="setFallbackImage"
                >
                <div>
                  <div class="similar-title">{{ item.title }}</div>
                  <div class="similar-meta">
                    {{ formatRub(item.price) }}<span v-if="item.oem"> · OEM: {{ item.oem }}</span>
                  </div>
                </div>
              </NuxtLink>
            </div>
          </div>
        </div>

        <div class="panel">
          <div class="pill">{{ product.category || 'Запчасть' }}</div>
          <h1 class="title">{{ product.title || 'Товар' }}</h1>
          <div class="price product-price">{{ formatRub(product.price) }}</div>

          <div class="meta">
            <div class="meta-item">
              <div class="k">OEM</div>
              <div class="v">{{ product.oem || '—' }}</div>
            </div>
            <div class="meta-item">
              <div class="k">Состояние</div>
              <div class="v">{{ product.condition || '—' }}</div>
            </div>
            <div class="meta-item">
              <div class="k">Адрес</div>
              <div class="v">{{ product.address || '—' }}</div>
            </div>
            <div class="meta-item">
              <div class="k">Бренд</div>
              <div class="v">{{ product.brand || '—' }}</div>
            </div>
            <div class="meta-item">
              <div class="k">Бренд авто</div>
              <div class="v">{{ product.make || '—' }}</div>
            </div>
          </div>

          <div class="actions-stack">
            <a class="btn primary" href="tel:89000013023">Позвонить и заказать</a>
            <NuxtLink class="btn" to="/catalog">Вернуться в каталог</NuxtLink>
          </div>

          <div class="payment-box">
            <h2 class="payment-heading">Оплата онлайн</h2>
            <p class="note">
              Блок оставлен подключенным к текущему бэкенду: он создает заказ через `/orders`.
              Когда платежный провайдер будет настроен, кнопка отправит покупателя на оплату.
            </p>
            <div class="field">
              <label for="email">Email</label>
              <input id="email" v-model="email" type="email" placeholder="buyer@example.com">
            </div>
            <div class="field">
              <label for="phone">Телефон</label>
              <input id="phone" v-model="phone" type="tel" placeholder="+7 900 000-00-00">
            </div>
            <div v-if="error" class="error">{{ error }}</div>
            <div class="payment-actions">
              <button class="btn primary" type="button" :disabled="loading" @click="pay">
                {{ loading ? 'Создаю ссылку…' : 'Оплатить через Точку' }}
              </button>
            </div>
          </div>

          <div class="desc">
            <h3>Описание</h3>
            <div class="text">{{ product.description || 'Описание не указано.' }}</div>
          </div>
        </div>
      </section>

      <section v-else class="panel">
        <div class="empty">
          <strong>Товар не найден.</strong>
          <div style="margin-top:14px;">
            <NuxtLink class="btn primary" to="/catalog">Вернуться в каталог</NuxtLink>
          </div>
        </div>
      </section>
    </div>
  </main>
</template>
