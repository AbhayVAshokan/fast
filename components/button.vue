<script setup>
const emits = defineEmits(["calculateDownloadSpeed"]);
const { isCalculating, isFinished } = defineProps({
  isCalculating: Boolean,
  isFinished: Boolean,
});

const handleCalculate = () => {
  if (!isCalculating || isFinished) {
    emits("calculateDownloadSpeed");
  }
};
</script>

<template>
  <div class="pulse"></div>
  <div class="ripple"></div>
  <Icon v-if="isFinished" name="ic:baseline-loop" class="retry" @click="handleCalculate" />
  <button @click="handleCalculate" :class="{ fade: isCalculating }">
    <slot></slot>
  </button>
  <div class="overlay" :class="{ expand: isCalculating }"></div>

  <Icon name="ic:baseline-loop" class="invisible" />
</template>

<style scoped>
.pulse {
  position: absolute;
  inset: 0;
  margin: auto;
  z-index: 10;
  background-color: var(--primary-button-foreground);
  border-radius: 50%;
  height: 15rem;
  width: 15rem;
  animation: pulse infinite 1s alternate;
}

.ripple {
  position: absolute;
  inset: 0;
  margin: auto;
  z-index: 8;
  height: 15rem;
  width: 15rem;
  border: 3px solid var(--primary-button-foreground);
  border-radius: 50%;
  animation: ripple infinite 6s;
}

.retry {
  background: #fff;
  opacity: 0.1;
  height: calc(min(90vh, 90vw));
  width: calc(min(90vh, 90vw));
  margin: auto;
  z-index: 15;
  cursor: pointer;
}

button {
  position: absolute;
  inset: 0;
  margin: auto;
  z-index: 15;
  width: 15rem;
  height: 15rem;
  border-radius: 50%;
  letter-spacing: 0.125rem;
  text-transform: uppercase;
  color: #fff;

  &:hover {
    background-color: rgba(red(var(--primary-button-foreground)),
        green(var(--primary-button-foreground)),
        blue(var(--primary-button-foreground)),
        0);

    &~.overlay {
      transform: translate(-50%, -50%) scale3d(1.5, 1.5, 1);
    }
  }
}

.fade {
  background-color: rgba(red(var(--primary-button-foreground)),
      green(var(--primary-button-foreground)),
      blue(var(--primary-button-foreground)),
      0);
}

.overlay {
  position: absolute;
  z-index: 12;
  top: 50%;
  left: 50%;
  margin: auto;
  width: calc(max(100vw, 100vh));
  height: calc(max(100vw, 100vh));
  background: var(--primary-button-foreground);
  border-radius: 50%;
  transform: translate(-50%, -50%) scale3d(0, 0, 1);
  transition: transform 0.3s ease-in-out;
}

.expand {
  transform: translate(-50%, -50%) scale3d(1.5, 1.5, 1);
}

@keyframes pulse {
  from {
    transform: scale3d(1, 1, 1);
  }

  to {
    transform: scale3d(1.05, 1.05, 1.05);
  }
}

@keyframes ripple {
  0% {
    transform: scale3d(1, 1, 1);
    opacity: 0.75;
  }

  30% {
    transform: scale3d(1.25, 1.25, 1.25);
    opacity: 0;
  }

  100% {
    transform: scale3d(1.25, 1.25, 1.25);
    opacity: 0;
  }
}
</style>
