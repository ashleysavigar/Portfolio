<template>
  <div class="w-screen h-screen bg-black flex items-center justify-center overflow-hidden">
    <CrtScreen>
      <div class="absolute inset-0 flex items-center justify-center content-wrapper">
        <Title
          :is-menu-open="isMenuOpen"
          :is-closing="isClosing"
          :has-played-startup="hasPlayedStartup"
          @click="toggleMenu"
        >
          Ashley Savigar
        </Title>

        <Menu
          :is-open="isMenuOpen"
          :is-closing="isClosing"
          @close="closeMenu"
        />

        <div
          v-if="isMenuOpen"
          class="absolute inset-0 z-[5] cursor-pointer bg-transparent"
          @click="closeMenu"
          aria-hidden="true"
        ></div>
      </div>
    </CrtScreen>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import CrtScreen from '@/components/screen/CrtScreen.vue'
import Menu from '@/components/navigation/Menu.vue'
import Title from '@/components/title/Title.vue'

const isMenuOpen = ref(false)
const hasPlayedStartup = ref(false)
const isClosing = ref(false)

const toggleMenu = () => {
  if (isMenuOpen.value) {
    isClosing.value = true
    setTimeout(() => {
      isMenuOpen.value = false
      isClosing.value = false
    }, 400)
  } else {
    isMenuOpen.value = true
  }
}

onMounted(() => {
  setTimeout(() => {
    hasPlayedStartup.value = true
  }, 2000)
})

const closeMenu = () => {
  if (isMenuOpen.value) {
    isClosing.value = true
    setTimeout(() => {
      isMenuOpen.value = false
      isClosing.value = false
    }, 400)
  }
}
</script>
