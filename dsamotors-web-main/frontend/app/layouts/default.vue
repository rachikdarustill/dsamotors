<script setup lang="ts">
const year = new Date().getFullYear();
</script>

<template>
  <div>
    <header class="topbar">
      <div class="container topbar-inner">
        <div class="header-left">
          <NuxtLink class="header-logo" to="/" aria-label="DSA Motors">
            <img :src="'/logo.png'" alt="DSA Motors">
          </NuxtLink>
          <NuxtLink class="brand-name" to="/">DSA Motors</NuxtLink>
        </div>

        <nav class="nav" aria-label="Основная навигация">
          <NuxtLink to="/">Главная</NuxtLink>
          <NuxtLink to="/catalog">Каталог</NuxtLink>
          <NuxtLink to="/#benefits">Преимущества</NuxtLink>
        </nav>

        <a href="tel:89000013023" class="header-phone">8 (900) 001-30-23</a>
      </div>
    </header>

    <slot />

    <footer>
      <div class="container footer-inner">
        <div class="footer-col">
          <div class="footer-brand">
            <img class="footer-logo" :src="'/logo.png'" alt="DSA Motors">
            <div class="brand-name">DSA Motors</div>
          </div>
          <p class="note" style="margin:0;">Интернет-магазин запчастей</p>
        </div>
        <div class="footer-col">
          <h5>Категории</h5>
          <NuxtLink to="/catalog">Все товары</NuxtLink>
          <NuxtLink to="/catalog?q=Mercedes">Mercedes-Benz</NuxtLink>
          <NuxtLink to="/catalog?q=BMW">BMW</NuxtLink>
        </div>
        <div class="footer-col">
          <h5>Покупателям</h5>
          <span class="non-clickable">Доставка и оплата</span>
          <span class="non-clickable">Гарантия и возврат</span>
        </div>
        <div class="footer-col">
          <h5>Контакты</h5>
          <a href="mailto:support@dsamotors.com">support@dsamotors.com</a>
          <a href="tel:89000013023">+7 (900) 001-30-23</a>
        </div>
      </div>
      <div class="copyright">
        © {{ year }} DSA. Все права защищены.
      </div>
    </footer>

    <NuxtLink to="/" class="corner-logo" aria-label="DSA">
      <img :src="'/logo.png'" alt="DSA">
    </NuxtLink>
  </div>
</template>
