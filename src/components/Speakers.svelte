<script>
  import { onMount } from 'svelte';
  let full_view = false;

  let speaker_list = [];
  let speakerColors = new Map();

  const colors = [
    { border: '#155dfc', bg: '#ffffff' }, // Red
    { border: '#155dfc', bg: '#ffffff' }, // Red

  ];

  function shuffleColorsWithoutRepeat(prevColor) {
    const available = colors.filter(c => c !== prevColor);
    for (let i = available.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [available[i], available[j]] = [available[j], available[i]];
    }
    return available;
  }

  function assignColors(flatList) {
    let queue = [...colors];
    let lastColor = null;
    let i = 0;

    for (const sp of flatList) {
      if (i >= queue.length) {
        queue = shuffleColorsWithoutRepeat(lastColor);
        i = 0;
      }
      speakerColors.set(sp.photo, queue[i]); // assuming photo is unique, else use name
      lastColor = queue[i];
      i++;
    }
  }

  function findPos(obj) {
    var curtop = 0;
    if (obj.offsetParent) {
        do {
            curtop += obj.offsetTop;
        } while (obj = obj.offsetParent);
    return curtop;
    }
  }


  onMount(() => {
    fetch(`https://events.startupmission.in/api/event/sep/speakers?category=speakers`)
      .then(response => response.json())
      .then((json) => {
        speaker_list = json;

        // Flatten all speakers for color assignment
        const all = Object.values(json).flat();
        assignColors(all);
      });
  });

  function getImage(photo) {
    return photo || '/default-avatar.png';
  }
</script>

{#each Object.entries(speaker_list) as [category, speakers]}
  <section class="overflow-hidden  openTrans relative">

    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4 ">
      {#each speakers as { name, designation, organisation, photo, linkedin }}
        <div class="relative group rounded-lg overflow-hidden flex flex-col border-2 h-full shadow-sm hover:shadow-lg transition-all duration-300"
             style="border-color: {speakerColors.get(photo)?.border}">
          <img src={getImage(photo)} alt={name} class="  w-full object-cover" loading="lazy" />

          <div class="text-black text-center py-2 px-2 font-semibold h-full flex-1" style="background-color: {speakerColors.get(photo)?.bg}">
            <h3 class="text-base">{name}</h3>
            <p class="text-xs font-normal">{designation}<br />{organisation}</p>
          </div>

          {#if linkedin}
            <a href={linkedin} target="_blank"
               class="absolute bottom-0 left-8 transform -translate-x-1/2 translate-y-full 
                      opacity-0 group-hover:-translate-y-20 group-hover:opacity-100 
                      transition-all duration-300 ease-in-out bg-white shadow-md 
                      rounded-full p-2 z-10"
               style="border: 2px solid {speakerColors.get(photo)?.bg}; color: {speakerColors.get(photo)?.bg}">
              <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" class="w-5 h-5" viewBox="0 0 24 24">
                <path d="M19 0h-14c-2.761 0-5 2.238-5 5v14c0 
                  2.762 2.239 5 5 5h14c2.762 0 5-2.238 
                  5-5v-14c0-2.762-2.238-5-5-5zm-11 
                  19h-3v-10h3v10zm-1.5-11.268c-.966 
                  0-1.75-.784-1.75-1.75s.784-1.75 
                  1.75-1.75 1.75.784 
                  1.75 1.75-.784 1.75-1.75 
                  1.75zm13.5 11.268h-3v-5.604c0-1.337-.026-3.059-1.865-3.059-1.867 
                  0-2.153 1.459-2.153 2.967v5.696h-3v-10h2.881v1.367h.041c.401-.761 
                  1.379-1.561 2.837-1.561 3.033 
                  0 3.593 1.996 3.593 4.59v5.604z"/>
              </svg>
            </a>
          {/if}
        </div>
      {/each}


     

    </div>
  </section>
   
{/each}



