<script setup lang="ts">
type Product = {
  id: string;
  title: string;
  category: string;
  make?: string;
  price: number;
  oem?: string;
  images?: string[];
};

const router = useRouter();
const { get } = useApi();
const q = ref('');
const shownCount = ref(8);

const { data: products, pending } = await useAsyncData(
  'products-home',
  () => get<Product[]>('/products'),
);

const list = computed(() => products.value || []);

const categories = computed(() => {
  const counts = new Map<string, number>();
  for (const product of list.value) {
    const category = String(product.category || '').trim();
    if (category) counts.set(category, (counts.get(category) || 0) + 1);
  }
  return Array.from(counts.entries())
    .sort((a, b) => b[1] - a[1] || a[0].localeCompare(b[0], 'ru'))
    .slice(0, 8);
});

const popularProducts = computed(() => list.value.slice(0, shownCount.value));

const submitSearch = async () => {
  const query = q.value.trim();
  if (!query) {
    window.alert('Введите VIN или номер детали для поиска');
    return;
  }
  await router.push({ path: '/catalog', query: { q: query } });
};
</script>

<template>
  <main>
    <section class="hero">
      <div class="container hero-inner">
        <div>
          <h1>Запчасти для Mercedes-Benz и BMW</h1>
          <p>Оригинальные и аналоговые запчасти из Японии. Поиск по VIN, OEM и названию детали.</p>
          <form class="search" role="search" @submit.prevent="submitSearch">
            <input
              v-model="q"
              type="text"
              placeholder="VIN Mercedes / номер детали"
              aria-label="VIN или номер детали"
            >
            <button type="submit">Найти</button>
          </form>
        </div>

        <div class="hero-card">
          <div class="pill">DSA Motors</div>
          <h2 class="section-title" style="margin-top:12px;">Подбор и продажа автозапчастей</h2>
          <p class="note">Каталог подключен к бэкенду, поэтому позже оплату можно привязать к текущему оформлению заказа.</p>
          <div class="hero-stats">
            <div class="stat">
              <strong>{{ list.length || '100+' }}</strong>
              <span>товаров</span>
            </div>
            <div class="stat">
              <strong>OEM</strong>
              <span>поиск</span>
            </div>
            <div class="stat">
              <strong>1 день</strong>
              <span>быстрый ответ</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="categories" class="section">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">Категории</h2>
          <NuxtLink class="link" to="/catalog">Открыть каталог</NuxtLink>
        </div>
        <div v-if="pending" class="panel">Загружаю товары…</div>
        <div v-else class="grid">
          <NuxtLink
            v-for="[category, count] in categories"
            :key="category"
            class="card"
            :to="{ path: '/catalog', query: { category } }"
            :aria-label="`Открыть категорию: ${category}`"
          >
            <div class="pill">{{ category }}</div>
            <div class="product-title">{{ count.toLocaleString('ru-RU') }} товаров</div>
          </NuxtLink>
        </div>
      </div>
    </section>

    <section id="popular" class="section">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">Популярные товары для Mercedes-Benz</h2>
          <NuxtLink class="link" to="/catalog">В магазин</NuxtLink>
        </div>
        <div v-if="pending" class="panel">Загружаю товары…</div>
        <div v-else class="grid">
          <ProductCard v-for="product in popularProducts" :key="product.id" :product="product" />
        </div>
        <div v-if="shownCount < list.length" class="popular-more">
          <button class="btn primary" type="button" @click="shownCount += 8">Показать еще</button>
        </div>
      </div>
    </section>

    <section id="benefits" class="section">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">Почему DSA Motors</h2>
        </div>
        <div class="benefits">
          <div class="benefit">
            <div class="icon">VIN</div>
            <div>
              <h4>Подбор по VIN</h4>
              <p>Помогаем найти совместимую деталь.</p>
            </div>
          </div>
          <div class="benefit">
            <div class="icon">OEM</div>
            <div>
              <h4>Поиск по OEM</h4>
              <p>Быстро сверяем номера и аналоги.</p>
            </div>
          </div>
          <div class="benefit">
            <div class="icon">₽</div>
            <div>
              <h4>Готовность к оплате</h4>
              <p>Страница товара уже создает заказ через бэкенд.</p>
            </div>
          </div>
          <div class="benefit">
            <div class="icon">24</div>
            <div>
              <h4>Связь с менеджером</h4>
              <p>Можно заказать по телефону.</p>
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
