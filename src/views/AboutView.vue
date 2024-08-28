<template>
  <main>
    <div class="container">
      <h1>Falling Sand</h1>
      <p>{{ text.value }}</p>
    </div>
  </main>
</template>
<script setup>
  const text = {
    value: 
    "The Falling Sand game was a browser game that I played as a kid. I always marveled at it, but never knew how to navigate to it so I would beg my brother to get to it for me. After some time I stopped being able to find it for years. There are other versions and recreations of it now online but I haven't seen the original. \n Anyways, this is a little passion project I put together to experiment with particle collision and physics rendering just using the web browser and Vue3."
    
  }
</script>

<style scoped>
main{
  
  height: 100vh;
}
.container{
  background: rgb(103, 97, 97);
  color: white;
  padding: 1rem;
  width: 40rem;
  border-radius: .5rem;
  margin-top: 5rem;
}
p{
  margin: 1rem;
}
</style>
