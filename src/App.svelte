<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import ollama from 'ollama'
  import { onMount } from 'svelte';

  let npc = {
      name: "Name",
      race: "Race",
      sex: "Sex",
      description: "Description",

      strength: 50,
      stamina: 50,
      dexterity: 50,
      response: 50,
      intuition: 50,
      logic: 50,
      personality: 50,
      leadership: 50,

      height: 5,
      weight: 5,
      hitPoints: 5,
      age: 5,
      walk: 5,
      run: 5,
      credits: 5,

      skills: [] as string[],
      inventory: [] as string[],
      weapons: [] as string[],
      skillLevel: [] as string[],
  };

  let difficulty = 4;
  let activeDifficultyButton = null;
  let numInventory = 3;
  let numSkills = 3;
  let numWeapons = 3;
  let customDescriptionActive = false;
  let customDescription = "";
  let descriptionLength = 100;

  const raceTypes = ["Human", "Dralisite", "Vrusk", "Yazarian"];
  const sexTypes = ["Male", "Female"];

  function statGenerator() {
      let randomFactor = Math.pow(Math.random(), 1 / (difficulty / 10));
      let stat: number;
      stat = Math.floor(Math.random() * 100) + 1;
      return stat;
  }

  function setDifficulty(num: number) {
      difficulty = num;
  }

  async function generate() {

      npc.inventory = [];
      npc.skills = [];
      npc.weapons = [];
      npc.description = "";
      npc.name = "";

      const race = Math.floor(Math.random() * (raceTypes.length-1)) + 1;
      npc.race = raceTypes[race];

      const sex = Math.floor(Math.random()) + 1;
      npc.sex = sexTypes[sex];

      npc.strength = statGenerator();
      npc.stamina = statGenerator();
      npc.dexterity = statGenerator();
      npc.response = statGenerator();
      npc.intuition = statGenerator();
      npc.logic = statGenerator();
      npc.personality = statGenerator();
      npc.leadership = statGenerator();

      npc.height = Math.floor(Math.random() * 3) + 1;
      npc.weight = Math.floor(Math.random() * (60 + 1)) + 40;
      npc.hitPoints = Math.floor(Math.random() * (60 + 1)) + 40;
      npc.age = Math.floor(Math.random() * (180 + 1)) + 20;
      npc.walk = Math.floor(Math.random() * (70 + 1)) + 10;
      npc.run = Math.floor(Math.random() * (70 + 1)) + 10;

      npc.credits = Math.floor(Math.random() * (100000 + 1)) + 100;

      const namePrompt = "Generate a sci-fi first and last name for a " + npc.sex + "individual. Include nothing more. DO NOT SAY YOU ARE GENERATING A NAME. THE ONLY TEXT SHOULD BE A NAME.";
      const lootPrompt = "Act as a scifi equipment generator. Generate the name of a piece equipment that could be in someone's inventory such as a welder. Include nothing more. KEEP YOUR RESPONSE UNDER FOUR WORDS.";
      const skillPrompt = "Act as a skill generator for an NPC. Generate a skill and include nothing more. KEEP YOUR RESPONSE UNDER TWO WORDS.";
      const weaponPrompt = "Act as a scifi weapon generator. Generate the name of a weapon that could be in someone's inventory such as a Laser Rifle. Include nothing more. KEEP YOUR RESPONSE UNDER FOUR WORDS.";

      try {
          const name = await ollama.chat({
              model: 'llama3.2:1b',
              messages: [{ role: 'user', content: namePrompt }],
              stream: true
          });
          for await (const part of name) {
              npc.name += part.message.content;
          }
      } catch (error) {
          console.error("Error fetching response:", error);
      }

      let descriptionPrompt = "";

      if (customDescription != "")
      {
          descriptionPrompt = "Generate a background for a character within the Star Frontiers sci-fi RPG called " + npc.name + " who is a " + npc.race + ". Here are your specific instructions: " + customDescription + ". Generate and include nothing more than description and keep it under" + descriptionLength + "words. DO NOT SAY YOU ARE MAKING A DESCRIPTION. Only generate a background. Do not include stats.";
      }
      else {
          descriptionPrompt = "Generate a background for a character within the Star Frontiers sci-fi RPG called " + npc.name + " who is a " + npc.race + ". Generate and include nothing more than description and keep it under" + descriptionLength + "words. DO NOT SAY YOU ARE MAKING A DESCRIPTION. Only generate a background. Do not include stats.";
      }

      try {
          const description = await ollama.chat({
              model: 'llama3.2:1b',
              messages: [{ role: 'user', content: descriptionPrompt }],
              stream: true
          });
          for await (const part of description) {
              npc.description += part.message.content;
          }
      } catch (error) {
          console.error("Error fetching response:", error);
      }

      for (let i = 0; i < numInventory; i++) {
          try {
              const inventory = await ollama.chat({
                  model: 'llama3.2:1b',
                  messages: [{ role: 'user', content: lootPrompt }],
              });

              npc.inventory = [...npc.inventory, inventory.message.content];
          } catch (error) {
              console.error("Error fetching response:", error);
          }
      }

      for (let i = 0; i < numSkills; i++) {
          try {
              const skills = await ollama.chat({
                  model: 'llama3.2:1b',
                  messages: [{ role: 'user', content: skillPrompt }],
              });

              npc.skills = [...npc.skills, skills.message.content];
          } catch (error) {
              console.error("Error fetching response:", error);
          }
      }

      for (let i = 0; i < numWeapons; i++) {
          try {
              const weapons = await ollama.chat({
                  model: 'llama3.2:1b',
                  messages: [{ role: 'user', content: weaponPrompt }],
              });

              npc.weapons = [...npc.weapons, weapons.message.content];
          } catch (error) {
              console.error("Error fetching response:", error);
          }
      }
  }


  //Experimental code

  let currentModel = 'No Model Selected';
  let showModelDropdown = false;
  let modelWarn = false;
  let isGenerating = false;
  let modelNames: string[] = [];

  //handle ollama models
  async function ollamaModels() {
      const ollamaModel = await ollama.list();
      modelNames = ollamaModel.models.map((model: any) => model.name);
  }

  function selectModel(button: any) {
      console.log(button)
      currentModel = button
      showModelDropdown = false;
  }

  function modelDropdown() {
      showModelDropdown = !showModelDropdown;
  }

  onMount(() => {
      ollamaModels();
  })

</script>

<main class="bg-black h-screen box-border">
    <nav class="flex-x justify-between items-center px-8 h-16">
        <div class="flex-x gap-8 items-center h-full">
            <div class="text-lg font-semibold text-gray-300">Joerpg</div>
            <div class="w-px h-full bg-gray-800"></div>
            <div class="flex gap-4">
                <div class="flex-x gap-4">
                    <button class="button-hybrid-rounded" aria-label="joe">Character 1</button>
                </div>
                <button class="icon-hybrid-rounded" aria-label="Youtube">
                    <i class="bi bi-plus-lg"></i>
                </button>
            </div>

        </div>

        <div class="text-sm font-semibold text-gray-500">v0.1.0-alpha</div>
    </nav>
    <div class="h-px w-full bg-gray-800"></div>
    <div class="flex-x h-[calc(100vh-65px)]">
        <div class="w-full p-6 flex gap-6 font-mono">
            <div class="w-full flex-y gap-4">
                <div class="label-fixed justify-between">
                    <div class="">{npc.name}</div>
                    <div class="">{npc.age} yrs</div>
                </div>

                <div class="flex gap-4">
                    <div class="label-fixed w-full">{npc.race}</div>
                    <div class="label-fixed w-full">{npc.sex}</div>
                </div>
                <div class="flex gap-4">
                    <div class="flex-y stat-label">
                        <div class="text-lg py-4 ">{npc.strength}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Strength</div>
                    </div>
                    <div class="flex-yl stat-label">
                        <div class="text-lg py-4 ">{npc.stamina}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Stamina</div>
                    </div>
                    <div class="flex-y stat-label">
                        <div class="text-lg py-4 ">{npc.dexterity}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Dexterity</div>
                    </div>
                    <div class="flex-y stat-label">
                        <div class="text-lg py-4 ">{npc.response}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Response</div>
                    </div>
                </div>
                <div class="flex gap-4">
                    <div class="flex-y stat-label">
                        <div class="text-lg py-4 ">{npc.intuition}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Intuition</div>
                    </div>
                    <div class="flex-y stat-label">
                        <div class="text-lg py-4 ">{npc.logic}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Logic</div>
                    </div>
                    <div class="flex-y stat-label">
                        <div class="text-lg py-4 ">{npc.personality}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Personality</div>
                    </div>
                    <div class="flex-y stat-label">
                        <div class="text-lg py-4 ">{npc.leadership}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Leadership</div>
                    </div>
                </div>
                <div class="h-px w-full bg-gray-800"></div>
                <div class="flex gap-4 w-full">
                    <div class="outline flex w-full">
                        <div class="text-[10px] text-gray-300 px-4 w-full p-2">Height</div>
                        <div class="min-w-px bg-gray-800"></div>
                        <div class="text-[10px] text-gray-300 px-4 w-full text-center p-2">{npc.height}</div>
                    </div>
                    <div class="outline flex w-full">
                        <div class="text-[10px] text-gray-300 px-4 w-full p-2">Weight</div>
                        <div class="min-w-px bg-gray-800"></div>
                        <div class="text-[10px] text-gray-300 px-4 w-full text-center p-2">{npc.weight}</div>
                    </div>
                </div>
                <div class="flex gap-4 w-full">
                    <div class="outline flex w-full">
                        <div class="text-[10px] text-gray-300 px-4 w-full p-2">Walk</div>
                        <div class="min-w-px bg-gray-800"></div>
                        <div class="text-[10px] text-gray-300 px-4 w-full text-center p-2">{npc.walk}</div>
                    </div>
                    <div class="outline flex w-full">
                        <div class="text-[10px] text-gray-300 px-4 w-full p-2">Run</div>
                        <div class="min-w-px bg-gray-800"></div>
                        <div class="text-[10px] text-gray-300 px-4 w-full text-center p-2">{npc.run}</div>
                    </div>
                </div>
                <div class="flex gap-4 w-full">
                    <div class="outline flex w-full">
                        <div class="text-[10px] text-gray-300 px-4 w-full p-2">Health</div>
                        <div class="min-w-px bg-gray-800"></div>
                        <div class="text-[10px] text-gray-300 px-4 w-full text-center p-2">{npc.hitPoints}</div>
                    </div>
                    <div class="outline flex w-full">
                        <div class="text-[10px] text-gray-300 px-4 w-full p-2">Credits</div>
                        <div class="min-w-px bg-gray-800"></div>
                        <div class="text-[10px] text-gray-300 px-4 w-full text-center p-2">{npc.credits}</div>
                    </div>
                </div>
                <div class="label flex-y gap-2 h-full">
                    <div class="flex-x justify-between">
                        <div class="">Skills</div>
                        <div class="">Lvl</div>
                    </div>
                    <div class="h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex-y gap-2 h-full">
                        {#each npc.skills as item}
                            <div class="w-full flex-x justify-between">
                                <div>{item}</div>
                                <div>2</div>
                            </div>
                        {/each}
                    </div>
                </div>
            </div>
            <div class="w-full flex-y gap-4">
                <div class="label overflow-y-auto h-full" id="description">{npc.description}</div>
                <div class="label flex-y gap-2">
                    <div class="">Inventory (Equipment)</div>
                    <div class="h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex-y gap-2 h-32">
                        {#each npc.inventory as item}
                            <div>{item}</div>
                        {/each}
                    </div>
                </div>
                <div class="label flex-y gap-2">
                    <div class="">Weapons</div>
                    <div class="h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex-y gap-2 h-32">
                        {#each npc.weapons as item}
                            <div>{item}</div>
                        {/each}
                    </div>
                </div>
            </div>
        </div>
        <div class="w-px bg-gray-800"></div>
        <div class="flex-y justify-between w-lg">
            <div class="flex justify-evenly">
                <button class="icon" aria-label="Youtube">
                    <i class="bi bi-arrow-clockwise"></i>
                </button>
                <div class="w-px bg-gray-800"></div>
                <button class="icon" aria-label="Youtube">
                    <i class="bi bi-shuffle"></i>
                </button>
                <div class="w-px bg-gray-800"></div>
                <button class="icon" aria-label="Youtube">
                    <i class="bi bi-clipboard"></i>
                </button>
                <div class="w-px bg-gray-800"></div>
                <button class="icon" aria-label="Youtube">
                    <i class="bi bi-sliders2-vertical"></i>
                </button>
                <div class="w-px bg-gray-800"></div>
                <button class="icon" aria-label="Youtube">
                    <i class="bi bi-gear"></i>
                </button>
            </div>
            <div class="h-px w-full bg-gray-800"></div>
            <div class="flex-y gap-4 m-4">
                <div class="flex gap-4">
                    <button on:click={modelDropdown()}  class="button-hybrid w-full rounded-none text-center font-semibold font-mono flex justify-center">{currentModel}</button>

                    <a class="icon-hybrid rounded-none" href="" aria-label="Youtube">
                        <i class="bi bi-eject"></i>
                    </a>
                </div>

                {#if showModelDropdown}
                    <div class="hybrid flex-y">
                        {#each modelNames as names}
                            <button on:click={() => selectModel(names)} class="tag">{names}</button>
                        {/each}
                    </div>
                {/if}
            </div>

            <div class="h-px w-full bg-gray-800"></div>
            <div class="h-full flex flex-y m-4 gap-5 overflow-y-auto">
                <div class="flex gap-3 flex-wrap">
                    <button class="hybrid tag">Easy</button>
                    <button class="hybrid tag">Medium</button>
                    <button class="hybrid tag">Hard</button>
                    <button class="hybrid tag">Nightmare</button>
                    <button class="hybrid tag">God</button>
                    <button class="hybrid tag">Random</button>
                </div>
                <div class="flex gap-3 flex-wrap">
                    <button class="hybrid tag">Human</button>
                    <button class="hybrid tag">Yazarian</button>
                    <button class="hybrid tag">Vrusk</button>
                    <button class="hybrid tag">Dralasite</button>
                    <button class="hybrid tag">Random</button>
                </div>
                <input type="text" id="first_name" class="label font-mono placeholder-gray-500 focus:outline-none" placeholder="Custom Description Prompt" bind:value={customDescription}>

                <div class="flex-y gap-4">
                    <div class="flex-x gap-3 font-mono">
                        <div class="hybrid text-gray-300 h-10 text-xs w-[65%] flex items-center px-4">Equipment Amount</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={numInventory}>
                    </div>
                    <div class="flex-x gap-3 font-mono">
                        <div class="hybrid text-gray-300 h-10 text-xs w-[65%] flex items-center px-4">Number of Skills</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={numSkills}>
                    </div>
                    <div class="flex-x gap-3 font-mono">
                        <div class="hybrid text-gray-300 h-10 text-xs w-[65%] flex items-center px-4">Weapons Amount</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={numWeapons}>
                    </div>
                    <div class="flex-x gap-3 font-mono">
                        <div class="hybrid text-gray-300 h-10 text-xs w-[65%] flex items-center px-4">Description Word Count</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={descriptionLength}>
                    </div>
                </div>
            </div>
            <div class="h-px w-full bg-gray-800"></div>
            <div class="p-4 flex">
                <div class="button-hybrid-dual w-full" >
                    <i class="bi bi-hammer"></i>
                    <div class="w-px bg-gray-700 h-full"></div>
                    <button on:click={generate} class="text-center text-sm font-semibold w-full font-mono cursor-pointer">Generate</button>
                </div>
            </div>
        </div>
    </div>
</main>

<style>

</style>
