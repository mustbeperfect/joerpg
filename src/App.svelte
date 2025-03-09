<script lang="ts">
    import ollama from 'ollama'
    import {onMount} from 'svelte';

    //AI system
    let currentModel = 'No Model Selected';
    let modelNames: string[] = [];
    let showModelDropdown = false;
    let noModel = false;
    let isGenerating = false;
    let ollamaRunning = false;

    async function ollamaModels() {
        const ollamaModel = await ollama.list();
        modelNames = ollamaModel.models.map((model: any) => model.name);
    }

    function selectModel(button: any) {
        console.log(button)
        currentModel = button;
        showModelDropdown = false;
    }

    function modelDropdown() {
        showModelDropdown = !showModelDropdown;
    }

    function ejectModel() {
        currentModel = 'No Model Selected';
    }

    async function ollamaStatus() {
        try {
            await ollama.list();
            ollamaRunning = true;
        } catch (error) {
            ollamaRunning = false;
        }
    }

    //NPC Stats

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
        skillLevel: [] as number[],
        inventory: [] as string[],
        weapons: [] as string[],
    };

    let characters: string[] = [];
    characters = [npc.name];

    //World stats
    const raceTypes = ["Human", "Dralisite", "Vrusk", "Yazarian"];
    const sexTypes = ["Male", "Female"];

    //Generation parameters
    let difficulty = 10;
    let numInventory = 3;
    let numSkills = 3;
    let numWeapons = 3;
    let descriptionLength = 100;
    let customDescription = "";
    let descriptionPrompt = "";
    let usedSkills = "";
    let usedInventory = "";
    let usedWeapons = "";
    let randomRace = true;
    let customRace = "";

    //Numerical generators
    function randomNum(min: number, max: number) {
        if (min > max) {
            [min, max] = [max, min];
        }
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }

    function randomStat() {
        let stat: number;
        stat = Math.floor(Math.random() * 100) + 1;
        return stat;
    }

    function weightedStat(difficulty: number) {
        const exponent = 1 + (difficulty - 1) * 0.5;
        const randomValue = Math.pow(Math.random(), exponent);
        return Math.floor(randomValue * 100) + 1;
    }

    //Setters
    function setDifficulty(num: number) {
        difficulty = num;
    }

    function setRace(race: string) {
        customRace = race;
        randomRace = false;
    }

    function setRaceRandom() {
        randomRace = true;
    }

    function resetParameters() {
        difficulty = 10;
        numInventory = 3;
        numSkills = 3;
        numWeapons = 3;
        descriptionLength = 100;
        customDescription = "";
        randomRace = true;
        customRace = "";
    }

    function randomParameters() {
        let difficultyQueue = randomNum(1,6)
        if (difficultyQueue == 6) {
            difficulty = 10;
        }
        else {
            difficulty = difficultyQueue;
        }
        numInventory = randomNum(1,20);
        numSkills = randomNum(1,20);
        numWeapons = randomNum(1,20);
        descriptionLength = randomNum(1,1000);
        let raceQueue = randomNum(0,raceTypes.length);
        if (raceQueue == raceTypes.length) {
            randomRace = true;
            customRace = "";
        }
        else {
            customRace = raceTypes[raceQueue];
            randomRace = false;
        }
    }

    function addCharacter() {
        characters = [...characters, npc.name];
    }

    //Generator
    async function generate() {

        npc.inventory = [];
        npc.skills = [];
        npc.weapons = [];
        npc.description = "";
        npc.name = "";
        npc.skillLevel = [];

        if (customRace == "Human") {
            npc.race = "Human";
        }
        else if (customRace == "Dralisite") {
            npc.race = "Dralisite";
        }
        else if (customRace == "Vrusk") {
            npc.race = "Vrusk";
        }
        else if (customRace == "Yazarian") {
            npc.race = "Yazarian";
        }
        else {
            randomRace = true;
        }

        if (randomRace == true) {
            npc.race = raceTypes[randomNum(0,raceTypes.length-1)];
        }

        npc.sex = sexTypes[Math.floor(Math.random()) + 1];

        if (difficulty == 10)
        {
            npc.strength = randomStat();
            npc.stamina = randomStat();
            npc.dexterity = randomStat();
            npc.response = randomStat();
            npc.intuition = randomStat();
            npc.logic = randomStat();
            npc.personality = randomStat();
            npc.leadership = randomStat();

            npc.height = randomNum(1,3);
            npc.weight = randomNum(20,100);
            npc.hitPoints = randomNum(20,100);
            npc.age = randomNum(15,200);
            npc.walk = randomNum(10,70);
            npc.run = randomNum(10,70);
        }
        else
        {
            npc.strength = weightedStat(difficulty);
            npc.stamina = weightedStat(difficulty);
            npc.dexterity = weightedStat(difficulty);
            npc.response = weightedStat(difficulty);
            npc.intuition = weightedStat(difficulty);
            npc.logic = weightedStat(difficulty);
            npc.personality = weightedStat(difficulty);
            npc.leadership = weightedStat(difficulty);

            npc.height = randomNum(1,3);
            npc.weight = randomNum(20,100);
            npc.hitPoints = randomNum(20,100);
            npc.age = randomNum(15,200);
            npc.walk = randomNum(10,70);
            npc.run = randomNum(10,70);
        }

        npc.credits =randomNum(100,100000);

        let namePrompt = "Generate a sci-fi first and last name for a " + npc.sex + "individual. Include nothing more. DO NOT SAY YOU ARE GENERATING A NAME. THE ONLY TEXT SHOULD BE A NAME.";
        let lootPrompt = "Act as a scifi equipment generator. Generate the name of a piece equipment that could be in someone's inventory such as a welder. Include nothing more. DO NOT PUT A PERIOD AT THE END. KEEP YOUR RESPONSE UNDER FOUR WORDS.";
        let skillPrompt = "Act as a skill generator for an NPC. Generate a skill and include nothing more. DO NOT PUT A PERIOD AT THE END. KEEP YOUR RESPONSE UNDER TWO WORDS.";
        let weaponPrompt = "Act as a scifi weapon generator. Generate the name of a weapon that could be in someone's inventory such as a Laser Rifle. Include nothing more. DO NOT PUT A PERIOD AT THE END. KEEP YOUR RESPONSE UNDER FOUR WORDS.";

        //Ollama
        try {
          const name = await ollama.chat({
              model: currentModel,
              messages: [{ role: 'user', content: namePrompt }],
              stream: true
          });
          for await (const part of name) {
              npc.name += part.message.content;
          }
        } catch (error) {
          console.error("Error fetching response:", error);
        }
        characters = [npc.name];

        if (customDescription != "")
        {
          descriptionPrompt = "Generate a background for a character within the Star Frontiers sci-fi RPG called " + npc.name + " who is a " + npc.race + ". Here are your specific instructions: " + customDescription + ". Generate and include nothing more than description and keep it under" + descriptionLength + "words. DO NOT SAY YOU ARE MAKING A DESCRIPTION. Only generate a background. Do not include stats.";
        }
        else {
          descriptionPrompt = "Generate a background for a character within the Star Frontiers sci-fi RPG called " + npc.name + " who is a " + npc.race + ". Generate and include nothing more than description and keep it under" + descriptionLength + "words. DO NOT SAY YOU ARE MAKING A DESCRIPTION. Only generate a background. Do not include stats.";
        }

        try {
          const description = await ollama.chat({
              model: currentModel,
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
            let usedInventory = "";
          try {
              const inventory = await ollama.chat({
                  model: currentModel,
                  messages: [{ role: 'user', content: lootPrompt }],
              });

              npc.inventory = [...npc.inventory, inventory.message.content];
          } catch (error) {
              console.error("Error fetching response:", error);
          }
            usedInventory += npc.inventory[i] + "";
            lootPrompt = "Act as a scifi equipment generator. Generate the name of a piece equipment that could be in someone's inventory such as a welder. Include nothing more. You cannot use the following equipment: " + usedInventory + ".DO NOT PUT A PERIOD AT THE END. KEEP YOUR RESPONSE UNDER FOUR WORDS.";
            /*
              for (let j = 0; j < npc.inventory.length; j++) {
                  usedInventory += npc.inventory[j];
              }

             */
        }

        for (let i = 0; i < numSkills; i++) {
          try {
              const skills = await ollama.chat({
                  model: currentModel,
                  messages: [{ role: 'user', content: skillPrompt }],
              });

              npc.skills = [...npc.skills, skills.message.content];
          } catch (error) {
              console.error("Error fetching response:", error);
          }
            usedSkills += npc.skills[i] + "";
            npc.skillLevel = [...npc.skillLevel,randomNum(1,6)];
            skillPrompt = "Act as a skill generator for an NPC. Generate a skill and include nothing more. You cannot use the following skills: " + usedSkills + ".DO NOT PUT A PERIOD AT THE END. KEEP YOUR RESPONSE UNDER TWO WORDS.";
        }

        for (let i = 0; i < numWeapons; i++) {
            try {
              const weapons = await ollama.chat({
                  model: currentModel,
                  messages: [{ role: 'user', content: weaponPrompt }],
              });

              npc.weapons = [...npc.weapons, weapons.message.content];
            } catch (error) {
              console.error("Error fetching response:", error);
            }
            usedWeapons += npc.weapons[i] + "";
            weaponPrompt = "Act as a scifi weapon generator. Generate the name of a weapon that could be in someone's inventory such as a Laser Rifle. Include nothing more. You cannot use the following weapons: " + usedWeapons + ".DO NOT PUT A PERIOD AT THE END. KEEP YOUR RESPONSE UNDER FOUR WORDS.";
        }
    }

    onMount(() => {
      ollamaModels();

        ollamaStatus();
        const interval = setInterval(ollamaStatus, 5000);

        return () => clearInterval(interval);
    })

</script>

<main class="bg-black h-screen box-border">
    <nav class="flex flex-row justify-between items-center px-8 h-16">
        <div class="flex flex-row gap-8 items-center h-full">
            <div class="text-lg font-semibold text-gray-300">Joerpg</div>
            <div class="min-w-px h-full bg-gray-800"></div>
            <div class="flex gap-4 overflow-x-auto w-full">
                <div class="flex flex-row gap-4">
                    {#each characters as character}
                        <button class="button-hybrid-rounded" aria-label="joe">{character}</button>
                    {/each}
                </div>

                <button class="icon-hybrid-rounded" aria-label="Youtube" on:click={addCharacter}>
                    <i class="bi bi-plus-lg"></i>
                </button>
            </div>

        </div>
        <div class="flex gap-8 items-center justify-center">
            {#if ollamaRunning}
                <div class="status-green">
                    <div class="min-h-[10px] min-w-[10px] bg-green-400 rounded-full"></div>
                    <div>Ollama Running</div>
                </div>
            {:else}
                <div class="status-red">
                    <div class="min-h-[10px] min-w-[10px] bg-red-400 rounded-full"></div>
                    <div>Ollama Not Running</div>
                </div>
            {/if}

            <div class="text-sm font-semibold text-gray-500">v0.1.0-alpha</div>
        </div>

    </nav>
    <div class="min-h-px w-full bg-gray-800"></div>
    <div class="flex flex-row h-[calc(100vh-65px)]">
        <div class="w-full p-6 flex gap-6 font-mono">
            <div class="min-w-sm flex flex-col gap-4">
                <div class="label-fixed justify-between">
                    <div class="">{npc.name}</div>
                    <div class="">{npc.age} yrs</div>
                </div>

                <div class="flex gap-4">
                    <div class="label-fixed w-full">{npc.race}</div>
                    <div class="label-fixed w-full">{npc.sex}</div>
                </div>
                <div class="flex gap-4">
                    <div class="flex flex-col stat-label">
                        <div class="text-lg py-4 ">{npc.strength}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Strength</div>
                    </div>
                    <div class="flex flex-coll stat-label">
                        <div class="text-lg py-4 ">{npc.stamina}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Stamina</div>
                    </div>
                    <div class="flex flex-col stat-label">
                        <div class="text-lg py-4 ">{npc.dexterity}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Dexterity</div>
                    </div>
                    <div class="flex flex-col stat-label">
                        <div class="text-lg py-4 ">{npc.response}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Response</div>
                    </div>
                </div>
                <div class="flex gap-4">
                    <div class="flex flex-col stat-label">
                        <div class="text-lg py-4 ">{npc.intuition}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Intuition</div>
                    </div>
                    <div class="flex flex-col stat-label">
                        <div class="text-lg py-4 ">{npc.logic}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Logic</div>
                    </div>
                    <div class="flex flex-col stat-label">
                        <div class="text-lg py-4 ">{npc.personality}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Personality</div>
                    </div>
                    <div class="flex flex-col stat-label">
                        <div class="text-lg py-4 ">{npc.leadership}</div>
                        <div class="min-h-px bg-gray-700 w-full"></div>
                        <div class="text-[10px] py-2 text-gray-300">Leadership</div>
                    </div>
                </div>
                <div class="min-h-px w-full bg-gray-800"></div>
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
                <div class="label flex flex-col gap-2 h-[calc(100%-528px)]">
                    <div class="flex flex-row justify-between">
                        <div class="">Skills</div>
                        <div class="">Lvl</div>
                    </div>
                    <div class="min-h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex flex-col gap-2 h-full">
                        {#each npc.skills as skill, i}
                            <div class="w-full flex flex-row justify-between">
                                <div>{skill}</div>
                                <div>{npc.skillLevel[i]}</div>
                            </div>
                        {/each}
                    </div>
                </div>
            </div>
            <div class="w-full flex flex-col gap-4">
                <div class="label overflow-y-auto h-[40%] xl:h-full" id="description">{npc.description}</div>
                <div class="label xl:hidden flex flex-col gap-2 h-[calc(30%-1rem)]">
                    <div class="">Inventory (Equipment)</div>
                    <div class="min-h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex flex-col gap-2">
                        {#each npc.inventory as item}
                            <div>{item}</div>
                        {/each}
                    </div>
                </div>
                <div class="label xl:hidden flex flex-col gap-2 h-[calc(30%-1rem)]">
                    <div class="">Weapons</div>
                    <div class="min-h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex flex-col gap-2">
                        {#each npc.weapons as item}
                            <div>{item}</div>
                        {/each}
                    </div>
                </div>
            </div>

            <div class="xl:flex xl:flex-col gap-4 hidden w-full">
                <div class="label flex flex-col gap-2 h-[calc(50%-0.5rem)]">
                    <div class="">Inventory (Equipment)</div>
                    <div class="min-h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex flex-col gap-2">
                        {#each npc.inventory as item}
                            <div>{item}</div>
                        {/each}
                    </div>
                </div>
                <div class="label flex flex-col gap-2 h-[calc(50%-0.5rem)]">
                    <div class="">Weapons</div>
                    <div class="min-h-px bg-gray-800"></div>
                    <div class="overflow-y-auto flex flex-col gap-2">
                        {#each npc.weapons as item}
                            <div>{item}</div>
                        {/each}
                    </div>
                </div>
            </div>

        </div>
        <div class="min-w-px bg-gray-800"></div>
        <div class="flex flex-col justify-between min-w-[350px] max-w-[350px]">
            <div class="flex justify-evenly">
                <button class="icon" aria-label="Reset" on:click={resetParameters}>
                    <i class="bi bi-arrow-clockwise"></i>
                </button>
                <div class="min-w-px bg-gray-800"></div>
                <button class="icon" aria-label="Shuffle" on:click={randomParameters}>
                    <i class="bi bi-shuffle"></i>
                </button>
                <div class="min-w-px bg-gray-800"></div>
                <button class="icon" aria-label="Copy">
                    <i class="bi bi-clipboard"></i>
                </button>
                <div class="min-w-px bg-gray-800"></div>
                <button class="icon" aria-label="Parameters">
                    <i class="bi bi-sliders2-vertical"></i>
                </button>
                <div class="min-w-px bg-gray-800"></div>
                <button class="icon" aria-label="Settings">
                    <i class="bi bi-gear"></i>
                </button>
            </div>
            <div class="min-h-px w-full bg-gray-800"></div>
            <div class="flex flex-col gap-4 m-4">
                <div class="flex gap-4">
                    <button on:click={modelDropdown}  class="button-hybrid w-full font-mono flex justify-center">{currentModel}</button>

                    <button on:click={ejectModel} class="icon-hybrid" aria-label="Youtube">
                        <i class="bi bi-eject"></i>
                    </button>
                </div>

                {#if showModelDropdown}
                    <div class="hybrid flex flex-col px-5 py-3 gap-3">
                        {#each modelNames as names}
                            <button on:click={() => selectModel(names)} class="dropdown-element">{names}</button>
                            <div class="min-h-px bg-gray-800"></div>
                        {/each}
                        <button on:click={ejectModel} class="dropdown-element">None</button>
                    </div>
                {/if}
            </div>

            <div class="min-h-px w-full bg-gray-800"></div>
            <div class="h-full flex flex-col m-4 gap-5 overflow-y-auto">
                <div class="flex gap-3 flex-wrap">
                    <button class={difficulty === 1 ? 'tag-toggled' : 'tag'}
                            on:click={() => setDifficulty(1)}>Easy
                    </button>
                    <button class={difficulty === 2 ? 'tag-toggled' : 'tag'}
                            on:click={() => setDifficulty(2)}>Medium
                    </button>
                    <button class={difficulty === 3 ? 'tag-toggled' : 'tag'}
                            on:click={() => setDifficulty(3)}>Hard
                    </button>
                    <button class={difficulty === 4 ? 'tag-toggled' : 'tag'}
                            on:click={() => setDifficulty(4)}>Nightmare
                    </button>
                    <button class={difficulty === 5 ? 'tag-toggled' : 'tag'}
                            on:click={() => setDifficulty(5)}>God
                    </button>
                    <button class={difficulty === 10 ? 'tag-toggled' : 'tag'}
                            on:click={() => setDifficulty(10)}>Random
                    </button>
                </div>
                <div class="flex gap-3 flex-wrap">
                    {#each raceTypes as race}
                        <button
                                class={customRace === race ? 'tag-toggled' : 'tag'}
                                on:click={() => setRace(race)}>
                            {race}
                        </button>
                    {/each}
                    <button
                            class={randomRace === true ? 'tag-toggled' : 'tag'}
                            on:click={setRaceRandom}>
                        Random
                    </button>
                </div>
                <input type="text" id="first_name" class="label font-mono placeholder-gray-500 focus:outline-none" placeholder="Custom Description Prompt" bind:value={customDescription}>

                <div class="flex flex-col gap-4">
                    <div class="flex flex-row gap-3 font-mono">
                        <div class="setting-label">Equipment Amount</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={numInventory}>
                    </div>
                    <div class="flex flex-row gap-3 font-mono">
                        <div class="setting-label">Number of Skills</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={numSkills}>
                    </div>
                    <div class="flex flex-row gap-3 font-mono">
                        <div class="setting-label">Weapons Amount</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={numWeapons}>
                    </div>
                    <div class="flex flex-row gap-3 font-mono">
                        <div class="setting-label">Description Word Count</div>
                        <input type="text" id="first_name" class="input placeholder-gray-500 text-center w-[35%] focus:outline-none" placeholder="#" bind:value={descriptionLength}>
                    </div>
                </div>
            </div>
            <div class="min-h-px w-full bg-gray-800"></div>

            <button on:click={generate} class="generate-button">
                Generate
                <i class="bi bi-hammer generate-button-icon"></i>
            </button>
        </div>
    </div>
</main>

<style>

</style>
