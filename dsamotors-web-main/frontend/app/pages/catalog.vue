<script setup lang="ts">
type Product = {
  id: string;
  title: string;
  category: string;
  condition?: string;
  make?: string;
  price: number;
  oem?: string;
  images?: string[];
};

const route = useRoute();
const router = useRouter();
const { get } = useApi();

const q = ref(String(route.query.q || ''));
const category = ref(String(route.query.category || ''));
const condition = ref('');
const brand = ref('');
const minPrice = ref('');
const maxPrice = ref('');
const sort = ref('default');
const shown = ref(200);

const { data: products, pending } = await useAsyncData(
  'products-catalog',
  () => get<Product[]>('/products', q.value ? { q: q.value } : undefined),
  { watch: [q] },
);

const allProducts = computed(() => products.value || []);

const uniqueValues = (field: keyof Product) => computed(() => {
  return Array.from(new Set(
    allProducts.value
      .map((product) => String(product[field] || '').trim())
      .filter(Boolean),
  )).sort((a, b) => a.localeCompare(b, 'ru'));
});

const categories = uniqueValues('category');
const conditions = uniqueValues('condition');
const brands = uniqueValues('make');

const normalized = (value: string) =>
  value.toLowerCase().replace(/ё/g, 'е').replace(/[^a-zа-я0-9]+/gi, ' ').trim();

const compact = (value: string) => normalized(value).replace(/\s+/g, '');

const matchesQuery = (product: Product, query: string) => {
  const tokens = normalized(query).split(/\s+/).filter(Boolean);
  if (!tokens.length) return true;
  const haystack = [
    product.oem || '',
    product.title || '',
    product.category || '',
    product.condition || '',
    product.make || '',
  ].join(' ');
  const normalizedHaystack = normalized(haystack);
  const compactHaystack = compact(haystack);
  const compactQuery = compact(query);

  return tokens.every((token) => normalizedHaystack.includes(token))
    || (compactQuery.length > 1 && compactHaystack.includes(compactQuery));
};

const filteredProducts = computed(() => {
  const min = minPrice.value === '' ? null : Number(minPrice.value);
  const max = maxPrice.value === '' ? null : Number(maxPrice.value);
  let result = allProducts.value.filter((product) => {
    if (q.value && !matchesQuery(product, q.value)) return false;
    if (category.value && String(product.category || '').trim() !== category.value) return false;
    if (condition.value && String(product.condition || '').trim() !== condition.value) return false;
    if (brand.value && String(product.make || '').trim() !== brand.value) return false;
    if (min !== null && Number.isFinite(min) && Number(product.price || 0) < min) return false;
    if (max !== null && Number.isFinite(max) && Number(product.price || 0) > max) return false;
    return true;
  });

  result = result.slice();
  if (sort.value === 'priceAsc') result.sort((a, b) => (a.price || 0) - (b.price || 0));
  if (sort.value === 'priceDesc') result.sort((a, b) => (b.price || 0) - (a.price || 0));
  if (sort.value === 'titleAsc') result.sort((a, b) => String(a.title || '').localeCompare(String(b.title || ''), 'ru'));
  if (sort.value === 'titleDesc') result.sort((a, b) => String(b.title || '').localeCompare(String(a.title || ''), 'ru'));
  return result;
});

const visibleProducts = computed(() => filteredProducts.value.slice(0, shown.value));

const applyFilters = async () => {
  shown.value = 200;
  await router.replace({
    query: {
      ...(q.value.trim() ? { q: q.value.trim() } : {}),
      ...(category.value ? { category: category.value } : {}),
    },
  });
};

const resetFilters = async () => {
  q.value = '';
  category.value = '';
  condition.value = '';
  brand.value = '';
  minPrice.value = '';
  maxPrice.value = '';
  sort.value = 'default';
  shown.value = 200;
  await router.replace({ query: {} });
};

watch(() => route.query, (query) => {
  q.value = String(query.q || '');
  category.value = String(query.category || '');
});
</script>

<template>
  <main class="page">
    <div class="container">
      <div class="section-header" style="align-items:flex-end; margin:10px 0 16px;">
        <div>
          <h1 class="section-title">Каталог товаров</h1>
        </div>
        <NuxtLink class="btn" to="/">На главную</NuxtLink>
      </div>

      <div class="toolbar" aria-label="Поиск и фильтры">
        <form class="search" role="search" aria-label="Поиск запчастей" @submit.prevent="applyFilters">
          <input v-model="q" type="text" placeholder="VIN Mercedes / номер детали" aria-label="VIN или номер детали">
          <button type="submit">Найти</button>
        </form>

        <div class="filter-panel" aria-label="Фильтры каталога">
          <div class="filters">
            <div class="field">
              <label for="categorySelect">Категория</label>
              <select id="categorySelect" v-model="category">
                <option value="">Все</option>
                <option v-for="item in categories" :key="item" :value="item">{{ item }}</option>
              </select>
            </div>
            <div class="field">
              <label for="conditionSelect">Состояние</label>
              <select id="conditionSelect" v-model="condition">
                <option value="">Все</option>
                <option v-for="item in conditions" :key="item" :value="item">{{ item }}</option>
              </select>
            </div>
            <div class="field">
              <label for="brandSelect">Бренд автомобиля</label>
              <select id="brandSelect" v-model="brand">
                <option value="">Все</option>
                <option v-for="item in brands" :key="item" :value="item">{{ item }}</option>
              </select>
            </div>
            <div class="field">
              <label for="minPrice">Цена от, ₽</label>
              <input id="minPrice" v-model="minPrice" type="number" min="0" step="1" placeholder="0">
            </div>
            <div class="field">
              <label for="maxPrice">Цена до, ₽</label>
              <input id="maxPrice" v-model="maxPrice" type="number" min="0" step="1" placeholder="∞">
            </div>
            <div class="field" style="grid-column: 1 / -1;">
              <label for="sortSelect">Сортировка</label>
              <select id="sortSelect" v-model="sort">
                <option value="default">По умолчанию</option>
                <option value="priceAsc">Цена: по возрастанию</option>
                <option value="priceDesc">Цена: по убыванию</option>
                <option value="titleAsc">Название: А → Я</option>
                <option value="titleDesc">Название: Я → А</option>
              </select>
            </div>
          </div>

          <div class="filters-actions">
            <button class="btn secondary" type="button" @click="resetFilters">Сбросить</button>
            <button class="btn primary" type="button" @click="applyFilters">Применить</button>
          </div>
        </div>
      </div>

      <div class="summary" aria-live="polite">
        Найдено: <b>{{ filteredProducts.length }}</b> из <b>{{ allProducts.length }}</b>.
        Показано: <b>{{ visibleProducts.length }}</b>.
      </div>

      <div v-if="pending" class="panel">Загружаю каталог…</div>
      <div v-else-if="!filteredProducts.length" class="panel">
        <div class="note">Ничего не найдено по выбранным фильтрам. Попробуйте изменить запрос или нажмите «Сбросить».</div>
      </div>
      <div v-else class="grid" aria-live="polite">
        <ProductCard v-for="product in visibleProducts" :key="product.id" :product="product" />
      </div>

      <div v-if="shown < filteredProducts.length" class="controls">
        <button class="btn primary" type="button" @click="shown += 200">
          Показать еще ({{ Math.min(200, filteredProducts.length - shown) }})
        </button>
      </div>
    </div>
  </main>
</template>
