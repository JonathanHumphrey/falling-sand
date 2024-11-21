<script setup>
    import { RouterLink } from 'vue-router'
</script>

<template>
    <header>
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
  </header>
</template>

<style>
    
header {
  line-height: 1.5;
  max-height: 100vh;
  min-width: 10rem;
  max-width: 20rem;
}

nav {
  width: 30%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
  border-radius: .25rem;
}

nav a.router-link-exact-active {
  color: rgb(255, 252, 252);
  border-left: 1px solid var(--color-border);
  background: rgb(137, 137, 137);
  border-radius: .25rem;
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  margin: 2rem;
  padding: 0 1rem;
}

nav a:first-of-type {
  border: 0;
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

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>