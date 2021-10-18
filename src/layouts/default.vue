<template>
  <section>
    <nav class="bg-gray-600 py-4 grid grid-cols-12">
      <!-- Hamburger menu icon -->
      <div class="flex justify-center mx-auto">
          <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="24" height="24" viewBox="0 0 24 24" style=" fill:#000000;" class="my-auto ml-2">
        <path d="M 2 5 L 2 7 L 22 7 L 22 5 L 2 5 z M 2 11 L 2 13 L 22 13 L 22 11 L 2 11 z M 2 17 L 2 19 L 22 19 L 22 17 L 2 17 z"></path>
      </svg>
      </div>
      <!-- Search -->
      <input type="text" class="px-2 py-2 sm:py-4 col-start-3 sm:col-start-2 col-span-8 sm:col-span-10 bg-opacity-0 rounded-md shadow-lg" placeholder="r/">
    </nav>
  </section>

  <!-- Body -->
  <div class="bg-gray-400 overflow-y-scroll" style="height: 91vh">
    <slot/>
  </div>
</template>

<script>
export default {

}
</script>

<style>
  ::-webkit-scrollbar {
    width: 3px;
  }

  /* Track */
  ::-webkit-scrollbar-track {
    background: var(--darkestgray); 
    
    border-radius: 1px;
  }
  
  /* Handle */
  ::-webkit-scrollbar-thumb {
    background: #333;
    
    border-radius: 1px;
  }

  /* Handle on hover */
  ::-webkit-scrollbar-thumb:hover {
    background: none; 
  }
</style>