<template>
  <div class="container max-w-xl mx-auto mb-4 px-4">
    <div class="flex flex-wrap">
      <div class="w-full mb-6 md:w-1/4 md:mb-0"><Categories /></div>
      <div class="w-full md:w-3/4">
        <div class="cards">
          <div
            class="flex shadow-lg rounded-lg overflow-hidden bg-white"
            v-for="product in products"
            :key="product._id"
          >
            <Product :product="product" />
          </div>
        </div>
        <div class="text-center mt-6">
          <a
            href="#"
            class="rounder inline-block px-4 py-3 bg-blue-dark text-white no-underline rounded"
            @click.prevent="loadMoreProduct"
            v-if="isLoadMoreBtn"
            >Load more</a
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ProductService from '@/services/product';
import Product from '@/components/Product.vue';
import Pagination from '@/components/Pagination.vue';
import Categories from '@/components/Categories.vue';

export default {
  components: { Product, Pagination, Categories },
  data() {
    return {
      bottom: false,
      products: [],
      currentPage: 0,
      pages: 0,
      isLoadMoreBtn: true,
      isLoadMoreFinish: false
    };
  },
  watch: {
    bottom(bottom) {
      if (bottom && !this.isLoadMoreFinish) {
        this.loadMoreProduct();
      }
    }
  },

  created() {
    ProductService.getProducts().then(res => {
      this.products = res.data.products;
      this.currentPage = res.data.currentPage;
      this.pages = res.data.pages;
    });
  },
  mounted() {
    window.addEventListener('scroll', () => {
      this.bottom = this.bottomVisible();
    });
  },
  methods: {
    bottomVisible() {
      const scrollY = window.scrollY;
      const visible = document.documentElement.clientHeight;
      const pageHeight = document.documentElement.scrollHeight;
      const bottomOfPage = visible + scrollY >= pageHeight;
      return bottomOfPage || pageHeight < visible;
    },
    clickHandlerPage(page) {
      ProductService.getProductsByPage(page).then(res => {
        this.products = res.data.products;
        this.currentPage = res.data.currentPage;
        this.pages = res.data.pages;
      });
    },
    loadMoreProduct() {
      const page = this.currentPage + 1;
      ProductService.getProductsByPage(page).then(res => {
        const { data } = res;

        if (!data.products.length) {
          this.isLoadMoreBtn = false;
          this.isLoadMoreFinish = true;
          return;
        }

        data.products.forEach(product => {
          this.products.push(product);
        });

        this.currentPage = page;
      });
    }
  }
};
</script>

<style scoped></style>
