<template>
  <div
    :class="[
      'title-container',
      !hasPlayedStartup && 'animate-startup-container',
      isMenuOpen && !isClosing && 'animate-menu-open',
      isClosing && 'animate-menu-close'
    ]"
    @click="handleClick"
    style="cursor: pointer; pointer-events: auto;"
  >
    <h1
      class="font-dalmation text-white whitespace-nowrap cursor-pointer select-none pointer-events-auto"
      @click.stop="handleClick"
      @keydown.enter="handleClick"
      @keydown.space.prevent="handleClick"
      tabindex="0"
      role="button"
      aria-label="Toggle navigation menu"
    >
      <slot />
    </h1>
  </div>
</template>

<script setup>
const props = defineProps({
  /** Whether the menu is currently open */
  isMenuOpen: {
    type: Boolean,
    default: false
  },
  /** Whether the menu is currently animating closed */
  isClosing: {
    type: Boolean,
    default: false
  },
  /** Whether the startup animation has played */
  hasPlayedStartup: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['click'])

const handleClick = () => {
  emit('click')
}
</script>

<style>
.title-container {
  position: fixed;
  top: 50%;
  left: 50%;
  z-index: 10 !important;
  pointer-events: auto !important;
  cursor: pointer;
}

.title-container h1 {
  transform: translate(-50%, -50%) scale(3);
  transform-origin: center center;
  transition: transform 0.2s ease;
  cursor: pointer !important;
  z-index: 100 !important;
  pointer-events: auto !important;
  white-space: nowrap;
  font-size: 3.75rem;
}

@media (min-width: 769px) and (max-width: 1024px) {
  .title-container h1 {
    font-size: 2rem;
  }
}

@media (max-width: 768px) {
  .title-container h1 {
    font-size: 1.25rem;
  }
}

.title-container.animate-startup-container h1 {
  animation: startup-text 2s ease-out forwards;
}

@keyframes startup-text {
  0% {
    transform: translate(-50%, -50%) scale(0.1);
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) scale(3);
    opacity: 1;
  }
}

.title-container:hover h1 {
  filter: brightness(1.2);
}

.title-container.animate-menu-open {
  animation: menu-open 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

.title-container.animate-menu-open h1 {
  animation: menu-open-text 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

.title-container.animate-menu-close {
  animation: menu-close 0.4s ease-in-out forwards;
}

.title-container.animate-menu-close h1 {
  animation: menu-close-text 0.4s ease-in-out forwards;
}

@keyframes menu-open {
  0% {
    top: 50%;
    left: 50%;
  }
  100% {
    top: calc(50% - 35vh);
    left: 50%;
  }
}

@keyframes menu-open-text {
  0% {
    transform: translate(-50%, -50%) scale(3);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) scale(1.5);
    opacity: 1;
  }
}

@media (max-width: 1024px) {
  @keyframes menu-open-text {
    0% {
      transform: translate(-50%, -50%) scale(3);
      opacity: 1;
    }
    100% {
      transform: translate(-50%, -50%) scale(3);
      opacity: 1;
    }
  }

  @keyframes menu-close-text {
    0% {
      transform: translate(-50%, -50%) scale(3);
      opacity: 1;
    }
    100% {
      transform: translate(-50%, -50%) scale(3);
      opacity: 1;
    }
  }
}

@keyframes menu-close {
  0% {
    top: calc(50% - 35vh);
    left: 50%;
  }
  100% {
    top: 50%;
    left: 50%;
  }
}

@keyframes menu-close-text {
  0% {
    transform: translate(-50%, -50%) scale(1.5);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) scale(3);
    opacity: 1;
  }
}
</style>
