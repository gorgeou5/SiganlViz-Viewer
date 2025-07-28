<script>
export default {
  data: function() {
    return {
      name: '뷰',
      type: 'javascript',
    };
  },
}
</script>

<template>
  <header>
    <div class="wrapper">
      {{ name }}
    </div>
  </header>
  <main>
    {{ type }}
</main>
</template>

<style scoped>
header {
  line-height: 1.5;
  color: red;
}


@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }


  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
