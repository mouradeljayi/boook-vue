<template lang="html">
  <div class="bg-gray-900 shadow-md fixed inset-x-0 top-0 z-20">
    <div class="container mx-auto px-12 py-4">
      <nav class="md:flex flex-row md:justify-between">
        <div class="flex flex-row justify-between items-center">
          <router-link to="/"> <span class="text-white text-4xl font-bold"><i class="fas fa-book-reader"></i> <span class="text-yellow-500">Vue</span> </span> </router-link>
          <button class="md:hidden focus:outline-none text-white" @click="showMobileMenu = !showMobileMenu"><i class="fas fa-bars fa-2x"></i></button>
        </div>
        <ul :class="{'hidden' : showMobileMenu}" class="flex flex-col md:flex-row md:items-center">
          <li class="mt-8 pr-8 ml-0 md:mt-0 md:ml-4 text-xl">
            <router-link to="/" class="hover:text-yellow-600 text-white">Books</router-link>
          </li>
          <li class="mt-8 md:mt-0 ml-0 pr-8 md:ml-4 text-xl" v-if="!$store.state.isAuthenticated">
            <router-link to="/signIn" class="hover:text-yellow-600 text-white">Sign In</router-link>
          </li>
          <li class="mt-8 md:mt-0 ml-0 pr-8 md:ml-4 text-xl" v-if="!$store.state.isAuthenticated">
            <router-link to="/signUp" class="hover:text-yellow-600 text-white">Sign Up</router-link>
          </li>
          <li class="mt-8 md:mt-0 ml-0 pr-8 md:ml-4 text-xl" v-if="$store.state.isAuthenticated">
            <router-link to="/myAccount" class="hover:text-yellow-600 text-white">My Account</router-link>
          </li>
          <li class="mt-8 md:mt-0 ml-0 pr-8 md:ml-4 text-xl" v-if="$store.state.isAuthenticated">
            <span @click="logout" class="hover:text-yellow-600 text-white" >Logout</span>
          </li>
          <li class="mt-8 md:mt-0 ml-0 pr-8 md:ml-4 text-xl uppercase font-semibold">
            <router-link to="/cart" class="text-white hover:text-yellow-700 relative">
               <i class="fas fa-shopping-cart fa-lg"></i> <span class="absolute bottom-4 left-5 rounded-full bg-yellow-600 text-white text-sm px-2"> {{ cartTotalLength }} </span>
             </router-link>
          </li>
          <li class="mt-8 pr-8 md:mt-0 md:ml-4 ml-0 text-lg uppercase font-semibold">
            <div class="flex items-center border-b border-yellow-500 py-2">
              <input v-model="search" class="bg-transparent border-none w-full text-white mr-3 py-1 px-2 leading-tight focus:outline-none" type="text" placeholder="Search for a book">
              <i class="fas fa-search text-yellow-500"></i>
            </div>
          </li>
        </ul>
      </nav>
    </div>
  </div>
  <div v-if="search" class="bg-yellow-400 text-white p-2 z-20 fixed mt-16 top-0 right-0" style="width:15.7rem;margin-right:5.9rem">
    <div v-for="product in searchProducts.slice(0,5)" class="p-2 hover:bg-white hover:text-gray-700">
      <router-link :to="{ path: '/books/' + product.slug }" class="flex items-center">
        <img :src="product.image" class="w-14" alt="book image">
        <div class="ml-2 flex flex-col">
          <span class="text-sm">
            {{ product.title }}
          </span>
          <span class="text-xs text-gray-700">
            by {{ product.author }}
          </span>
        </div>
      </router-link>
    </div>
    <div class="px-8" v-if="searchProducts.length === 0">
      No results for "{{ search }}"
    </div>
  </div>
  <router-view />
</template>

<script>
import User from '../apis/User'
import axios from 'axios'

export default {
  name: 'NavBar',
  data() {
    return {
      showMobileMenu: false,
      search: '',
      cart: {
        items: [],
      }
    }
  },
  beforeCreate() {
    this.$store.commit('initializeStore')

    const token = this.$store.state.token

    if(token) {
      axios.defaults.headers.common['Authorization'] =  "Token" + token
    } else {
      axios.defaults.headers.common['Authorization'] = ""
    }
  },
  mounted() {
    this.cart = this.$store.state.cart
  },
  computed: {
    logout() {
      User.logout().then(() => {
        this.$store.commit('REMOVE_TOKEN')
        this.$router.push({ name: 'home' });
      })
    },
    cartTotalLength() {
      let totalLength = 0

      for(let i = 0; i < this.cart.items.length; i++) {
        totalLength += this.cart.items[i].qty
      }
      return totalLength
    },
    searchProducts() {
      return this.$store.state.products.filter(product => {
        return product.title.toLowerCase().includes(this.search.toLowerCase())
      })
    },
  },
  watch: {
    $route(to, from) {
      if(to.name === 'Book') {
        this.$store.dispatch('fetchProduct', { id: this.$route.params.id })
        this.search = ""
      }
    }
  },
}
</script>

<style lang="css" scoped>
</style>
