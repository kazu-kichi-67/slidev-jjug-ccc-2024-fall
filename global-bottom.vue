<template>
    <footer
      v-if="$slidev.nav.currentLayout !== 'cover' && $slidev.nav.currentPage !== $slidev.nav.total"
      class="absolute bottom-0 right-5"
    >
        {{ $slidev.nav.currentPage - 1 }} / {{ $slidev.nav.total - 2 }}
    </footer>
</template>