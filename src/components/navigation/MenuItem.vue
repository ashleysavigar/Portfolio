<template>
  <li :class="itemClass">
    <a :href="href" @click="handleClick">
      <slot />
    </a>
  </li>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  /** The href/anchor link for the menu item */
  href: {
    type: String,
    required: true
  },
  /** The index of the menu item for animation delay */
  index: {
    type: Number,
    required: true
  }
})

const emit = defineEmits(['click'])

const itemClass = computed(() => `menu-item-${props.index}`)

const handleClick = () => {
  emit('click')
}
</script>

<style>
.menu-list li {
  opacity: 0;
  animation: fade-in-up 0.4s ease-out forwards;
}

.menu-item-1 { animation-delay: 0.3s; }
.menu-item-2 { animation-delay: 0.4s; }
.menu-item-3 { animation-delay: 0.5s; }
.menu-item-4 { animation-delay: 0.6s; }
.menu-item-5 { animation-delay: 0.7s; }

.menu-list a {
  font-family: var(--font-dalmation);
  font-size: 3rem;
  color: #fff;
  text-decoration: none;
  display: block;
}

@media (max-width: 768px) {
  .menu-list a {
    font-size: 2rem;
  }
}
</style>
