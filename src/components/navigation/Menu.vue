<template>
  <nav
    v-if="isVisible"
    :class="[
      'navigation-menu',
      !isClosing && 'animate-menu-appear',
      isClosing && 'animate-menu-disappear'
    ]"
    role="navigation"
    aria-label="Main navigation"
  >
    <ul class="menu-list">
      <MenuItem
        v-for="(item, index) in menuItems"
        :key="item.href"
        :href="item.href"
        :index="index + 1"
        @click="handleItemClick"
      >
        {{ item.label }}
      </MenuItem>
    </ul>
  </nav>
</template>

<script setup>
import { computed } from 'vue'
import MenuItem from './MenuItem.vue'

const props = defineProps({
  /** Whether the menu is currently open */
  isOpen: {
    type: Boolean,
    default: false
  },
  /** Whether the menu is currently animating closed */
  isClosing: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['close'])

const menuItems = [
  { label: 'Projects', href: '#projects' },
  { label: 'About', href: '#about' },
  { label: 'Skills', href: '#skills' },
  { label: 'Contact', href: '#contact' },
  { label: 'Resume', href: '#resume' }
]

const isVisible = computed(() => props.isOpen || props.isClosing)

const handleItemClick = () => {
  emit('close')
}
</script>

<style>
.navigation-menu {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 20;
  pointer-events: auto;
}

.animate-menu-appear {
  animation: var(--animate-menu-appear);
}

.animate-menu-disappear {
  animation: menu-disappear 0.3s ease-in forwards;
}

.menu-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: center;
}
</style>
