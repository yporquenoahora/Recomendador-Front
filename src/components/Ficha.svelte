<!-- Ficha.svelte -->
<script>
    import {  selectedSortOption, recomendadas, categoria, keywords, 
      recomendadasKeywords, titulo, selectedTags, recomendadasTags, 
    searchTerm, filteredMovies} from "../stores/movieStore"
    import {getUrls, ENV} from "./../stores/diccionario.js"

    export let data = [];
    let isVisible = false;
    let divResaltado = null;
    let showSinopsis = false;
    const customSort = (a,b)=> {        
                
                for (const feature of $selectedSortOption) {
                    
                    if (a[feature] !== b[feature]) {
                        if(feature =="title")
                            return b[feature] < a[feature] ? 1 : -1;
                        else
                            return b[feature] > a[feature] ? 1 : -1;
                    }
                }
                return 0; // Si todas las características son iguales
            }; 

    function onMouseOver(div) {
        isVisible = true;
        divResaltado = div;
    }

    function onMouseOut() {
        isVisible = false;
        divResaltado = null;
    }

    function toggleVissible() {
        showSinopsis = !showSinopsis
    }
    
    $: callAPI = ( key, title) => {
        async function obtenerKeyWords(  ) {
                let url, urlTags;
                //console.log("%cobtenerKeyWords",'color: pink; font-size: 18px;', $titulo)
                urlTags = getUrls().urlsTags;
                url = (ENV == "local") ? 
                    ($categoria.peliculas) ? urlTags.peliculas.dev : urlTags.series.dev
                    : 
                    ($categoria.peliculas) ?  urlTags.peliculas.build : urlTags.series.build
                
                const respuesta = await fetch(url, 
                    {
                    method: 'GET',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                 }).then(response => {                    
                        if (!response.ok) {
                            throw new Error('Network response was not ok');
                        }
                        return response.json();
                    })
                    .then(data => {

                        $keywords = data.keywords;
                        $recomendadas = data.data;
                        $recomendadasKeywords = data.keywords;
                        //console.log("%ckeywords",'color: green; font-size: 18px;',data, $recomendadas, $keywords)                        
                    })
               
            .catch(error => {
                console.error('Error al cargar la plantilla:', error);
            });
        }
        async function obtenerDatosDesdeAPI(  ) {
                title = (title == "" || !title) ? "Con faldas y a lo loco" : title
                $titulo = title;
                let url, urls;
                urls = getUrls().urlItems;
                url = (ENV == "local") ? 
                    ($categoria.peliculas) ? urls.peliculas.dev : urls.series.dev
                    : 
                    ($categoria.peliculas) ?  urls.peliculas.build : urls.series.build                
              
                
                const respuesta = await fetch(url, 
                    {
                    method: 'GET',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                 }).then(response => response.json()) 
                    .then(data => {                        
                        $recomendadas = data || $recomendadas;
                        $recomendadasTags = (key == "tags") ? data : $recomendadasTags;
                        $selectedTags= {tags:[],audio:[],generos:[],pais:[], keywords:[]};                                                                        
                        $searchTerm = "";
                        $filteredMovies = $recomendadas;
                        //refreshSelectedTags()
                       // console.log("%crecomendadas","color:orange, font-size:16px", $recomendadas);
                        obtenerKeyWords($titulo, 20); 
                    })
               
            .catch(error => {
                console.error('Error al cargar la plantilla:', error);
            });
                
        }

        // Llama a la función para obtener datos desde la API
        obtenerDatosDesdeAPI();
        
    }
  </script>
  {#if data.length>0}
  {#each data.sort(customSort) as movie}
      <div class="contenedor {movie.title}"
  
     >
      
      <div
      >
      <div class="column">
       
          <h2><a href="{movie.link}">{movie.title}</a></h2>
          <img class="pic" src="{movie.image}" alt="{movie.title}" />
     
          <p> <b style="font-weight: bold;">{movie.generos}</b>,
          {#if movie.availability}     
              Disponible hasta {movie.availability},
          {/if}
          {movie.duracion},
          {#if movie.estreno}     
              {movie.estreno},
          {/if}
          {movie.audio},
          puntuado {movie.score},
          <b style="font-weight: bold;">{movie.votos}</b> votos<br>
          <div class="recomendar"
          on:mouseover={() => onMouseOver(movie.title)}
          on:click={callAPI("sin",movie.title)}
          on:keydown={callAPI("sin",movie.title)}
          on:mouseout={onMouseOut}
          on:focus={onMouseOut}
          on:blur={onMouseOut}>
              <button>Recomendar parecidas</button>
          </div>
          <div class="sinopsis">
          {#if showSinopsis}
         
            <span>{movie.sinopsis}</span>
            <button on:click={toggleVissible}>
                Ocultar sinopsis
            </button>
          
          {:else}
          <button on:click={toggleVissible}>
            Mostrar sinopsis
            </button>
          {/if}
        </div>
          
      </div>
     
      <div class="column">
          
              <div class="keywords">
                  {#if $recomendadas[movie.title]}
                      {#each $recomendadas[movie.title] as ix}
                      <span class="key_catalogo">{ix}</span>
                      {/each}
                  {/if}
              </div>
          </div>

      </div>
      <div class="tags">
          {#each movie.tags.split(",") as tag}
              <span class="tag">{tag}</span>
          {/each}
      </div>
      
      
      </div>
  {/each}
{/if}

<style lang="scss">
    @import './Ficha.scss'; // Importa tu archivo SCSS aquí
    /* Otras reglas de estilo SCSS aquí */
  </style>