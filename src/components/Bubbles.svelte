<script>    
	import { filteredMovies } from '@stores/movieStore.js';
	import { onMount, onDestroy } from 'svelte';
	import { generos, selectedTags, recomendadas, keywords } from '../stores/movieStore.js';
    
    

    export let data =  [];
    export let key = "";
    //export let labels = ['Label A', 'Label B', 'Label C', 'Label D', 'Label E', 'Label F'];
   
    export let height = 300;
    let itemsToShow = 5;
    let elementWidth = 0; // Initialize with an initial value

    let targetElement; // Reference to the target element
    let show = []

    // Update elementWidth when the component is mounted
    // and when the target element's width changes due to resizing
    const updateElementWidth = () => {
        elementWidth = targetElement.offsetWidth;
    };

    onMount(updateElementWidth); // Initialize elementWidth when the component is mounted

    // Update elementWidth when the target element is resized
    const handleResize = () => {
        updateElementWidth();
    };

    window.addEventListener('resize', handleResize);

    // Clean up the event listener when the component is destroyed
    onDestroy(() => {
        window.removeEventListener('resize', handleResize);
    });
    
    
    const toggleSubCategory = (key)=> {
      if(show[key]) show[key] = false
      else show[key] = true
    }

    //$:data = data ? data : []
    $: sortedData = Object.entries(data).sort(customSort);
    $: sortedGenres = Object.entries(genre_counts).sort(customSort);
    
    const customSort = (a,b)=> {                        
               return a[1] < b[1];
            };
    const genresMap = {};
    
    /* const individualGenres = Object.entries(sortedData).map(([genre, count]) => ({
        genre,
        count
    }));    */      
    let genre_counts = {};

    $: {
        genre_counts = {};
        sortedData.forEach(entry => {
        
        let genres = entry[0].split(',');
        let count = entry[1]
        genres.forEach(genre => {
            if (genre in genre_counts)
                genre_counts[genre] += count
            else
                genre_counts[genre] = count            
        });
    });}
   

//$: console.log(sortedData,genre_counts, $generos, data)

    

    let activeIndex = -1; // -1 indica que no hay ningún panel abierto

  function togglePanel(index) {
    activeIndex = (activeIndex === index) ? -1 : index;
    
    
   
      if ($selectedTags[key].includes(index)) {
            $selectedTags[key] = $selectedTags[key].filter(opt => opt !== index);
          } else {
            $selectedTags[key] = [...$selectedTags[key], index];
          } 
      
    
  }

  function showMore() {
    // Incrementa la cantidad de elementos para mostrar    
    itemsToShow = (sortedGenres.length >= itemsToShow + 10) ? itemsToShow + 10 : sortedGenres.length; 
  }
    
  function showLess() {
        // Incrementa la cantidad de elementos para mostrar    
        itemsToShow = (Object.entries(sortedGenres).length >= 10 ) ? itemsToShow - 10 : Object.entries(sortedDataTags).length - 1; 
    }

  $: reset = ()=>{
    $selectedTags[key] = [];
  }
    
  </script>
    
  <!-- <button class="reset" on:click={()=> reset()}>RESET</button> -->

  <div class="chart" bind:this={targetElement}>
    {#each sortedGenres.slice(0, itemsToShow) as [keyItem, value]}    
    <div class="bar-container" 
      height=" {height / data.length - 10}px"
      class:active={$selectedTags[key].includes(keyItem)}
      on:click={() => togglePanel(keyItem)}
      on:keydown={togglePanel(keyItem)}>
        <div class="label">{keyItem}</div>
        <div
          class="bar"
          
        />
         <span class="valor"> &nbsp;{value}</span>
      </div>
     
    {/each}
    </div>
    {#if itemsToShow < sortedGenres.length}
      <button on:click={showMore}>Mostrar más ...</button>
    {/if}

    {#if itemsToShow > 9}
      <button on:click={showLess}>Mostrar menos ...</button>
    {/if}
  <style>
     
    .chart {
    display: flex;
    align-items: initial;
    height: auto;
    
    flex-wrap: wrap;
  }

  .valor{
    color: var(--int-font-color-default)
  }
  button{
    border: 2px solid rgb(104, 133, 110) ;
    border-radius: 15px;
    padding: .5rem;
    background-color: var(--filmin-color-base) ;
    font-weight: bold;
    font-size: 14px;
    cursor: pointer;
  }

  .bar-container {
    display: flex;
    align-items: center;
    margin: 10px;
    padding: 10px;
    border-radius: 1rem;
    background-color: var(--filmin-color-base) ;
    cursor: pointer;
  }
  .bar-container.active{
      background-color: rgb(255, 127, 221);
    }

 

  .bar {
    background-color: #d5f2e8;
    height: 20px;
    
  }

  .label {
    font-size: 12px;
    margin-right: 10px;
    color: var(--int-font-color-default)
    
  }
 
  </style>