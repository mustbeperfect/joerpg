<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import ollama from 'ollama'

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

      skills: [],
      inventory: []
  };

  let difficulty = 0;
  let activeDifficultyButton = null;
  const raceTypes = ["Human", "Dralisite", "Vrusk", "Yazarian", "Sathar"];
  const sexTypes = ["Male", "Female"];

  function statGenerator() {
      const stat = Math.floor(Math.random() * 100) + 1;
      return stat;
  }

  function setDifficulty(num: number) {
      difficulty = num;
  }

  async function generate() {

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

      const namePrompt = "Generate a sci-fi name for a " + npc.sex + "individual. Include nothing more. DO NOT SAY YOU ARE GENERATING A NAME. THE ONLY TEXT SHOULD BE A NAME.";

      try {
          const name = await ollama.chat({
              model: 'llama3.2:1b',
              messages: [{ role: 'user', content: namePrompt }],
          });

          npc.name = name.message.content;
      } catch (error) {
          console.error("Error fetching response:", error);
      }

      const descriptionPrompt = "Generate a very short background for a character within the Star Frontiers sci-fi RPG called " + npc.name + " who is a " + npc.race + ". Generate and include nothing more than description and keep it under 100 words. DO NOT SAY YOU ARE MAKING A DESCRIPTION. Generate a background and past specifically and choose jobs and other lore relative to her stats. Strength: " + npc.strength + ", Stamina: " + npc.stamina + ", Dexterity: " + npc.dexterity + ", Response time: " + npc.response + ", Intuition: " + npc.intuition + ", Logic: " + npc.logic + ", Personality: " + npc.personality + ", Leadership: " + npc.leadership;

      try {
          const description = await ollama.chat({
              model: 'llama3.2:1b',
              messages: [{ role: 'user', content: descriptionPrompt }],
          });

          npc.description = description.message.content;
      } catch (error) {
          console.error("Error fetching response:", error);
      }
  }

</script>

<main class="bg-black h-screen box-border">
    <nav class="flex flex-row justify-between items-center px-8 h-16">
        <div class="flex gap-8 items-center h-full">
            <div class="text-lg font-semibold text-gray-300">Joerpg</div>
            <div class="w-px h-full bg-gray-800"></div>
            <div class="flex gap-4">
                <div class="flex gap-4">
                    <button class="buttonhybrid" aria-label="joe">Character 1</button>
                </div>
                <a class="iconhybrid" href="" aria-label="Youtube">
                    <i class="bi bi-plus-lg"></i>
                </a>
            </div>

        </div>

        <div class="text-sm font-semibold text-gray-500">v0.1.0-alpha</div>
    </nav>
    <div class="h-px w-full bg-gray-800"></div>
    <div class="flex flex-row h-[calc(100vh-65px)]">
        <div class="w-full p-6 flex gap-6 font-mono">
            <div class="w-full flex flex-col gap-4">
                <div class="regularlabel">{npc.name}</div>
                <div class="flex gap-4">
                    <div class="regularlabel w-full">{npc.race}</div>
                    <div class="regularlabel w-full">{npc.sex}</div>
                </div>
                <div class="flex gap-4">
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.strength}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Strength</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.stamina}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Stamina</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.dexterity}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Dexterity</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.response}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Response</div>
                    </div>
                </div>
                <div class="flex gap-4">
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.intuition}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Intuition</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.logic}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Logic</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.personality}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Personality</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.leadership}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Leadership</div>
                    </div>
                </div>
                <div class="h-px w-full bg-gray-800"></div>
                <div class="flex gap-4">
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.height}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Height</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.weight}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Weight</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.hitPoints}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Hit Points</div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.age}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Age</div>
                    </div>
                </div>
                <div class="flex gap-4">
                    <div class="flex gap-4 w-full">
                        <div class="flex flex-col statlabel">
                            <div class="text-lg py-4 ">{npc.walk}</div>
                            <div class="h-px bg-gray-700 w-full"></div>
                            <div class="text-[10px] py-2 text-gray-300">Walk</div>
                        </div>
                        <div class="flex flex-col statlabel">
                            <div class="text-lg py-4 ">{npc.run}</div>
                            <div class="h-px bg-gray-700 w-full"></div>
                            <div class="text-[10px] py-2 text-gray-300">Run</div>
                        </div>
                    </div>
                    <div class="flex flex-col statlabel">
                        <div class="text-lg py-4 ">{npc.credits}</div>
                        <div class="h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Credits</div>
                    </div>
                </div>
            </div>
            <div class="w-full flex flex-col gap-4">
                <div class="regularlabel" id="description">{npc.description}</div>
                <div class="regularlabel p-0 flex flex-col gap-2">
                    <div class="">Equipment (Loot)</div>
                </div>
                <div class="regularlabel p-0 flex flex-col gap-2">
                    <div class="">Skills</div>
                </div>
            </div>
        </div>
        <div class="w-px bg-gray-800"></div>
        <div class="flex flex-col justify-between w-lg">
            <div class="flex justify-evenly">
                <a class="icon" href="" aria-label="Youtube">
                    <i class="bi bi-arrow-clockwise"></i>
                </a>
                <div class="w-px bg-gray-800"></div>
                <a class="icon" href="" aria-label="Youtube">
                    <i class="bi bi-shuffle"></i>
                </a>
                <div class="w-px bg-gray-800"></div>
                <a class="icon" href="" aria-label="Youtube">
                    <i class="bi bi-clipboard"></i>
                </a>
                <div class="w-px bg-gray-800"></div>
                <a class="icon" href="" aria-label="Youtube">
                    <i class="bi bi-sliders2-vertical"></i>
                </a>
                <div class="w-px bg-gray-800"></div>
                <a class="icon" href="" aria-label="Youtube">
                    <i class="bi bi-gear"></i>
                </a>
            </div>
            <div class="h-px w-full bg-gray-800"></div>
            <div class="flex m-4 gap-4">
                <a class="iconhybriddualedge w-full rounded-none" href="" aria-label="Youtube">
                    <div class="text-center text-xs font-semibold w-full font-mono">llama3.2:1b</div>
                </a>
                <a class="iconhybridsecond rounded-none" href="" aria-label="Youtube">
                    <i class="bi bi-eject"></i>
                </a>
            </div>
            <div class="h-px w-full bg-gray-800"></div>
            <div class="h-full flex flex-col">
                <div class="flex gap-3 flex-wrap m-4">
                    <button on:click={setDifficulty(2)} class="tag">Easy</button>
                    <button on:click={setDifficulty(4)} class="tag">Medium</button>
                    <button on:click={setDifficulty(6)} class="tag">Hard</button>
                    <button on:click={setDifficulty(8)} class="tag">Nightmare</button>
                    <button on:click={setDifficulty(10)} class="tag">Chad</button>
                    <button on:click={setDifficulty(2)} class="tag">Random</button>
                </div>
                <div class="flex gap-3 flex-wrap m-4">
                    <button class="tag">Human</button>
                    <button class="tag">Yazarian</button>
                    <button class="tag">Vrusk</button>
                    <button class="tag">Dralasite</button>
                    <button class="tag">Random</button>
                </div>
            </div>
            <div class="h-px w-full bg-gray-800"></div>
            <div class="p-4 flex">
                <div class="iconhybriddual w-full" >
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
